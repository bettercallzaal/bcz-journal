---
topic: inspiration
type: guide
status: research-complete
last-validated: 2026-05-13
parent-doc: 647
tier: STANDARD
---

# 647a — Platform Governance Internals (Blue + CRE blue-review)

## TL;DR

Mental-Wealth-Academy's BlueKillStreak governance contract implements a dual-layer voting model where Blue AI agent (holding 40% of governance tokens) gates all proposals with a confidence level (0-4), each level maps to a veto-proof vote weight (0%, 10%, 20%, 30%, 40%), and community voting is snapshot-protected to prevent token-transfer attacks. CRE blue-review workflow triggers on ProposalCreated events, calls Eliza Cloud AI to score proposals across 6 dimensions (clarity, impact, feasibility, budget, ingenuity, chaos), computes Blue's level from total score, and writes a DON-signed review on-chain via onReport() interface — the entire AI review is decentralized and cryptographically verifiable. Governance requires 50% token approval threshold and auto-executes when threshold is crossed.

## Key Findings

- BlueKillStreak.sol contains 1 main contract (468 LOC), 4 enums/structs, 8 custom errors, 8 events, 10 state variables (immutable governance token, USDC token, Blue address, total supply, approval threshold, proposal count, min/max voting periods, Keystone forwarder). Base contract: Ownable (OpenZeppelin) + ReentrancyGuard (contracts/src/BlueKillStreak.sol:1-12).

- Blue's veto is level-0 only (Blue votes rejected) — levels 1-4 approve with increasing token weights per formula at BlueKillStreak.sol:326: `blueVoteWeight = (totalGovernanceTokens * _level * 10) / 100`, yielding level 1 = 10%, level 2 = 20%, level 3 = 30%, level 4 = 40% of total supply.

- Approval threshold hardcoded to 50% at BlueKillStreak.sol:171: `approvalThreshold = (_totalSupply * 50) / 100`.

- Snapshot anti-manipulation: voting power captured at proposal creation (BlueKillStreak.sol:246) and stored in proposal.snapshotBlock, then voters pull past votes at BlueKillStreak.sol:355 via `IVotes(address(governanceToken)).getPastVotes(msg.sender, proposal.snapshotBlock)`. Test coverage (BlueKillStreakTest.t.sol:439-467) demonstrates voter1 transferring tokens post-snapshot does not grant voter2 inflated vote weight.

- Dual onReport() receivers: actionType 1 (BlueKillStreak.sol:387-395) auto-executes proposal if forVotes >= threshold; actionType 2 (BlueKillStreak.sol:396-407) calls _blueReviewInternal() to apply Blue's AI-derived confidence level. Both gated by onlyForwarder modifier (BlueKillStreak.sol:215-217).

- CRE blue-review workflow triggers on ProposalCreated event (cre-workflows/blue-review/main.ts:92-100, event signature "ProposalCreated(uint256,address,address,uint256,string,uint256)"), reads proposal from contract, sends to Eliza Cloud API, parses 6-dim response, computes level (cre-workflows/blue-review/main.ts:142), builds actionType 2 report payload, submits DON-signed report via writeReport() to onReport() receiver.

- 6-dim AI scoring (cre-workflows/blue-review/main.ts:60-90): clarity, impact, feasibility, budget, ingenuity, chaos (each 0-10). Total score out of 60. If total < 25, level = 0 (reject); otherwise level = ceil((total / 60) * 4), capped at 4 (cre-workflows/blue-review/main.ts:142).

- Eliza Cloud API endpoint: elizacloud.ai, POST /api/v1/chat with Bearer token auth (cre-workflows/blue-review/config.production.json:3). Workflow caches responses for 60s (cre-workflows/blue-review/main.ts:165).

- Live contract on Base: 0x2cbb90a761ba64014b811be342b8ef01b471992d (cre-workflows/blue-review/config.production.json:2).

- Test suite: 31 governance tests across 5 categories (proposal creation 3 tests, Blue review 4 tests, community voting 8 tests, snapshot 1 test, view functions 3 tests, admin functions 4 tests, CRE integration 5 tests, gas optimization 2 tests, security fixes 6 tests). Total LOC in test file: 843 lines.

## BlueKillStreak.sol Surface

| Function | Access | Purpose | On-chain Effect |
|----------|--------|---------|-----------------|
| createProposal | External | Create a new funding proposal with voting period | Increments proposalCount, sets proposal to Pending, captures snapshotBlock |
| blueReview | External onlyBlue | Blue reviews proposal with confidence level (0-4) | Sets blueLevel, updates status to Active or Rejected, computes vote weight |
| vote | External nonReentrant | Community member casts vote on Active proposal | Records vote, updates forVotes/againstVotes, may trigger _executeProposal if threshold crossed |
| executeProposal | External nonReentrant | Manually execute proposal if threshold reached | Calls _executeProposal internally |
| _executeProposal | Internal | Internal execution logic | Sets executed=true, status=Executed, transfers USDC to recipient |
| onReport | External onlyForwarder nonReentrant | Receive DON-signed CRE reports (actionType 1 or 2) | Dispatches to auto-execute or _blueReviewInternal based on actionType |
| _blueReviewInternal | Internal | Apply Blue review from CRE (bypasses onlyBlue) | Identical to blueReview; used by onReport with actionType 2 |
| setKeystoneForwarder | External onlyOwner | Set Chainlink CRE forwarder address | Updates keystoneForwarder state |
| cancelProposal | External onlyAdmin | Cancel a non-executed proposal | Sets status to Cancelled |
| setAdmin | External onlyOwner | Add/remove admin address | Updates isAdmin mapping |
| setBlueAgent | External onlyOwner | Update Blue agent address | Updates blueAgent state |
| emergencyWithdraw | External onlyOwner | Withdraw USDC from contract | Transfers USDC to owner |
| getProposal | External view | Fetch full proposal struct | Returns Proposal memory |
| getVote | External view | Fetch individual vote details | Returns Vote memory |
| hasReachedThreshold | External view | Check if proposal forVotes >= approvalThreshold | Returns bool |
| getVotingProgress | External view | Fetch vote counts and percentage | Returns (forVotes, againstVotes, percentageFor) |
| getVotingPower | External view | Get delegated votes for address | Calls IVotes.getVotes (current block) |

## Voting + Veto Math

### Level-to-Weight Mapping
```
Blue's confidence level determines her vote weight:
- Level 0: Reject proposal outright (status = Rejected, weight = 0, veto is absolute)
- Level 1: 10% approval  = (totalSupply * 1 * 10) / 100 = 0.1 * totalSupply
- Level 2: 20% approval  = (totalSupply * 2 * 10) / 100 = 0.2 * totalSupply
- Level 3: 30% approval  = (totalSupply * 3 * 10) / 100 = 0.3 * totalSupply
- Level 4: 40% approval  = (totalSupply * 4 * 10) / 100 = 0.4 * totalSupply
```

### Approval Logic
```
Approval threshold = 50% of total supply (immutable at construction)

Proposal passes if:
  forVotes >= approvalThreshold
  where forVotes = Blue's vote weight + sum(community votes cast in favor)

Auto-execution: When a community vote pushes forVotes >= threshold,
  _executeProposal() is invoked immediately (nonReentrant).

Snapshot protection:
  Each proposal stores createdAt = block.timestamp, snapshotBlock = block.number
  Voters pull voting power at proposal.snapshotBlock (via getPastVotes)
  Prevents vote-transfer attacks (token transfers post-snapshot do not inflate voting power)
```

### Example Scenarios
```
Scenario A: Blue Level 4 (40%) + Voter1 (10%) = 50% -> Auto-execute
Scenario B: Blue Level 1 (10%) + 4 voters @ 10% each = 50% -> Auto-execute
Scenario C: Blue Level 0 -> Immediate rejection, no community vote possible
```

## CRE blue-review Workflow

### Sequence Diagram
```
1. User calls createProposal(recipient, usdcAmount, title, description, votingPeriod)
   -> ProposalCreated event emitted with (proposalId, proposer, recipient, usdcAmount, title, deadline)

2. CRE blue-review listens for ProposalCreated on 0x2cbb...992d (Base mainnet)
   -> Event matched, proposalId extracted from log.topics[1]

3. CRE calls BlueKillStreak.getProposal(proposalId)
   -> Returns full proposal struct: title, description, usdcAmount, status (must be Pending)

4. CRE sends POST /api/v1/chat to elizacloud.ai with:
   {
     "messages": [
       { "role": "system", "parts": [{ "type": "text", "text": REVIEW_SYSTEM_PROMPT }] },
       { "role": "user", "parts": [{ "type": "text", "text": userPrompt }] }
     ],
     "id": "gpt-4o"
   }
   Authorization: Bearer ${ELIZA_API_KEY}
   Cache: store=true, maxAge=60s

   userPrompt constructs:
   **Title:** {proposal.title}
   **Proposal:** {proposal.description}
   **Requested Amount:** {usdcAmount / 1e6} USDC

5. Eliza AI returns JSON (possibly SSE-streamed):
   {
     "decision": "approved" or "rejected",
     "scores": {
       "clarity": 0-10,
       "impact": 0-10,
       "feasibility": 0-10,
       "budget": 0-10,
       "ingenuity": 0-10,
       "chaos": 0-10
     },
     "tokenAllocation": 1-40 or null,
     "reasoning": "..."
   }

6. CRE computes blueLevel:
   total = clarity + impact + feasibility + budget + ingenuity + chaos
   if total < 25:
     level = 0
   else:
     level = min(ceil((total / 60) * 4), 4)

7. CRE builds actionType 2 report payload:
   innerPayload = abi.encode(uint256(proposalId), uint256(level))
   reportPayload = abi.encode(uint8(2), innerPayload)

8. CRE runtime generates DON-signed report via prepareReportRequest(reportPayload)
   -> Signs with oracle quorum, proof-of-work consensus

9. CRE calls BlueKillStreak.onReport("", signedReport) via writeReport
   -> onReport dispatches on actionType == 2

10. onReport calls _blueReviewInternal(proposalId, level)
    -> Sets proposal.blueLevel = level
    -> If level 0: status = Rejected
    -> If level 1-4: status = Active, computes blueVoteWeight, records Blue's vote

11. Proposal now Active; community can vote
```

### Key Integration Points
- **Event Trigger**: ProposalCreated (cre-workflows/shared/abi.ts:64) emitted at createProposal():249
- **Contract Read**: getProposal ABI (cre-workflows/shared/abi.ts:14-44) decodes 15-tuple (id, proposer, recipient, usdcAmount, title, description, createdAt, votingDeadline, status, forVotes, againstVotes, blueLevel, blueApproved, executed, snapshotBlock)
- **API Call Shape**: POST to elizaApiUrl/api/v1/chat, Authorization header with ELIZA_API_KEY secret, request body contains system prompt + user proposal text, caches for 60 seconds
- **Scoring Consensus**: consensusIdenticalAggregation aggregates AI responses across DON nodes
- **Report Submission**: writeReport(receiver=contractAddress, report=signedReport) invokes onReport on-chain

## Test Coverage (31 governance tests)

| Test Name | What Asserted | Category |
|-----------|---------------|----------|
| test_CreateProposal | Proposal created with correct ID, proposer, recipient, amount, status=Pending, snapshotBlock > 0 | Proposal Creation |
| test_RevertWhen_CreateProposalZeroRecipient | Revert InvalidProposal when recipient=address(0) | Proposal Creation |
| test_RevertWhen_CreateProposalZeroAmount | Revert InvalidAmount when usdcAmount=0 | Proposal Creation |
| test_BlueReviewLevel0_Kill | Blue Level 0 sets status=Rejected, blueApproved=false, forVotes=0 | Blue Review |
| test_BlueReviewLevel1_10Percent | Blue Level 1 sets blueLevel=1, forVotes=(totalSupply*10)/100, status=Active | Blue Review |
| test_BlueReviewLevel4_40Percent | Blue Level 4 sets forVotes=(totalSupply*40)/100 | Blue Review |
| test_RevertWhen_NonBlueCannotReview | Revert Unauthorized when non-Blue calls blueReview | Blue Review |
| test_CommunityVoting | Voter1 votes, forVotes updated with vote weight | Community Voting |
| test_50PercentThresholdAutoExecutes | Blue (10%) + 4 voters (40% total) = 50% triggers auto-execute, USDC transferred | Community Voting |
| test_BlueLevel4NeedsOnly10PercentMore | Blue (40%) + 1 voter (10%) = 50% auto-executes | Community Voting |
| test_RevertWhen_DoubleVoting | Revert AlreadyVoted when same voter votes twice | Community Voting |
| test_RevertWhen_VoteAfterDeadline | Revert VotingEnded when vote cast after votingDeadline | Community Voting |
| test_VotingAgainst | againstVotes incremented when voter votes false | Community Voting |
| test_SnapshotPreventsVoteTransferAttack | voter1 transfers tokens to voter2 post-snapshot; both still vote at snapshot weight, voter2 does NOT double-count | Snapshot |
| test_GetVotingProgress | getVotingProgress returns forVotes, againstVotes, percentageFor | View Functions |
| test_HasReachedThreshold | hasReachedThreshold returns false before threshold, true after | View Functions |
| test_GetVotingPower | getVotingPower returns IVotes.getVotes (delegated votes) | View Functions |
| test_CancelProposal | Admin cancels proposal, status=Cancelled | Admin Functions |
| test_SetAdmin | Owner adds/removes admin, isAdmin mapping updated | Admin Functions |
| test_SetBlueAgent | Owner updates blueAgent address | Admin Functions |
| test_EmergencyWithdraw | Owner withdraws USDC, balance increased | Admin Functions |
| test_SetKeystoneForwarder | Owner sets keystoneForwarder address | CRE Integration |
| test_RevertWhen_NonOwnerSetsForwarder | Revert OwnableUnauthorizedAccount when non-owner calls setKeystoneForwarder | CRE Integration |
| test_OnReportAutoExecute | CRE forwarder calls onReport(actionType=1), proposal auto-executes if threshold met | CRE Integration |
| test_OnReportBlueReview | CRE forwarder calls onReport(actionType=2, proposalId, level=3), proposal becomes Active with blueLevel=3 | CRE Integration |
| test_OnReportBlueReviewLevel0Kill | CRE forwarder calls onReport(actionType=2, level=0), proposal=Rejected | CRE Integration |
| test_OnReportExecuteViaForwarder | End-to-end: CRE reviews (level 3) + community votes to threshold + auto-executes | CRE Integration |
| test_RevertWhen_NonForwarderCallsOnReport | Revert Unauthorized when non-forwarder calls onReport | CRE Integration |
| test_RevertWhen_OnReportInvalidActionType | Revert InvalidProposal when actionType not in [1, 2] | CRE Integration |
| test_RevertWhen_VotingPeriodTooShort | Revert "Voting period out of bounds" when votingPeriod < 1 hour | Security Fixes |
| test_RevertWhen_VotingPeriodTooLong | Revert "Voting period out of bounds" when votingPeriod > 30 days | Security Fixes |
| test_VotingPeriodMinBound | Proposal created with votingPeriod = 1 hour (MIN_VOTING_PERIOD) | Security Fixes |
| test_VotingPeriodMaxBound | Proposal created with votingPeriod = 30 days (MAX_VOTING_PERIOD) | Security Fixes |
| test_EmitKeystoneForwarderUpdated | setKeystoneForwarder emits KeystoneForwarderUpdated(oldForwarder, newForwarder) | Security Fixes |
| test_EmitBlueAgentUpdated | setBlueAgent emits BlueAgentUpdated(oldAgent, newAgent) | Security Fixes |
| test_EmitAdminUpdated | setAdmin emits AdminUpdated(admin, status) | Security Fixes |
| test_EmitEmergencyWithdraw | emergencyWithdraw emits EmergencyWithdraw(to, amount) | Security Fixes |
| test_GasCreateProposal | createProposal uses < 250k gas | Gas Optimization |
| test_GasVoting | vote uses < 200k gas | Gas Optimization |

## Build-on-top Hooks (for ZAO)

| ZAO Surface | Reusable Pattern | Integration Effort | Notes |
|------------|------------------|-------------------|-------|
| ZOE governance scoring | Blue's 6-dim AI scoring (clarity, impact, feasibility, budget, ingenuity, chaos) | S | Map ZOE dimensions to Eliza prompt; reuse parseBlueResponse + computeLevel logic |
| Proposal approval % tiers | Blue's level→weight formula (level N = N*10%) | S | Adapt to ZOE's reputation/skill tiers; snapshot mechanism blocks sybil votes |
| Token-weighted voting | Dual-layer governance (Blue veto + community vote, 50% threshold) | M | ZAO could enforce min community quorum before Blue votes; test vote transfer attacks |
| CRE event-trigger workflow | ProposalCreated → Eliza API → DON-signed report → on-chain | M | Architecture proven; reuse trigger pattern for WaveWarZ or SongJam treasury proposals; adapt abi.ts for new events |
| Snapshot anti-manipulation | getPastVotes at proposal.snapshotBlock | S | Copy BlueKillStreak.vote() vote logic; OpenZeppelin IVotes standard; prevents token-swap attacks post-proposal |
| USDC transfer execution | Standard ERC20.transfer() in _executeProposal | S | Replace USDC with SANG or ZOL; test nonReentrant guard against price-oracle contracts |
| Chainlink DON integration | writeReport + onReport dispatch (actionType 1 vs 2) | M | Dual actionType architecture clean; extend for auto-rebalance (actionType 3) or treasury hedges |
| Test harness (31 tests) | Forge + ERC20Permit + ERC20Votes mocks, vm.prank/vm.warp/vm.roll | M | Covers proposal lifecycle, threshold math, snapshot protection, CRE integration; extend for multi-sig recovery |

## Sources

- [BlueKillStreak.sol (GitHub blob)](https://github.com/Mental-Wealth-Academy/platform/blob/main/contracts/src/BlueKillStreak.sol)
- [BlueKillStreak.t.sol (GitHub blob)](https://github.com/Mental-Wealth-Academy/platform/blob/main/contracts/test/BlueKillStreak.t.sol)
- [cre-workflows/blue-review/main.ts (GitHub blob)](https://github.com/Mental-Wealth-Academy/platform/blob/main/cre-workflows/blue-review/main.ts)
- [cre-workflows/blue-review/workflow.yaml (GitHub blob)](https://github.com/Mental-Wealth-Academy/platform/blob/main/cre-workflows/blue-review/workflow.yaml)
- [cre-workflows/shared/abi.ts (GitHub blob)](https://github.com/Mental-Wealth-Academy/platform/blob/main/cre-workflows/shared/abi.ts)
- [BlueKillStreak on BaseScan (live contract)](https://basescan.org/address/0x2cbb90a761ba64014b811be342b8ef01b471992d)
