# AI Bounty Judge — Commit-Reveal Edition

Fixes the workshop version's flaw: answers were public immediately, so later
participants could copy earlier ideas. Now answers stay hidden until judging.

## Lifecycle

1. **Create** — owner calls `createBounty(title, rubric, submissionDeadline, revealDeadline)` with a reward.
2. **Commit** (before `submissionDeadline`) — each participant calls `submitCommitment(bountyId, commitment)` where:
   ```solidity
   commitment = keccak256(abi.encodePacked(answer, salt, msg.sender, bountyId));
   ```
   Only the hash is stored. No one — not even the owner — can read the answer yet.
3. **Reveal** (between `submissionDeadline` and `revealDeadline`) — each participant calls
   `revealAnswer(bountyId, answer, salt)`. The contract recomputes the hash and checks it
   matches. Wrong hash, wrong sender, or wrong bountyId → reverts.
4. **Judge** (after `revealDeadline`) — owner calls `judgeAll(bountyId, llmInput)`. `llmInput`
   is built off-chain from **only the revealed answers** and sent as one batch request to the
   Ritual LLM inference precompile (`0x0802`) — not one call per submission.
5. **Finalize** — owner calls `finalizeWinner(bountyId, winnerIndex)`. Reverts if that index
   never revealed. Pays the reward to the winner.

## Why `msg.sender` and `bountyId` are in the hash

Without them, someone could see another participant's commitment on-chain, and once *any*
answer is revealed anywhere, copy that exact `(answer, salt)` pair and "reveal" it as their
own. Binding the hash to the sender's address and the specific bounty makes a copied
commitment fail for anyone but the original submitter.

## What's NOT testable locally

`judgeAll` calls a Ritual-only precompile (`LLM_INFERENCE_PRECOMPILE`) that doesn't exist on a
local Hardhat network. Tests cover the full commit-reveal state machine instead — see
`test/AIJudgeCommitReveal.ts`. `judgeAll`/`finalizeWinner` happy-path needs the Ritual testnet.

## Files

- `contracts/AIJudgeCommitReveal.sol` — the updated contract
- `contracts/utils/PrecompileConsumer.sol` — unchanged, from the workshop starter
- `test/AIJudgeCommitReveal.ts` — commit-reveal test suite
- `ARCHITECTURE.md` — commit-reveal vs. Ritual-native encrypted submissions
