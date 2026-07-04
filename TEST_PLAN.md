# Test Plan

11 tests in `test/AIJudgeCommitReveal.ts`, all passing on local Hardhat network.

| # | Case | Expected |
|---|------|----------|
| 1 | Create bounty with submission + reveal deadlines | Deadlines stored correctly |
| 2 | Reveal deadline before submission deadline | Reverts |
| 3 | Commit during submission phase | Commitment stored, answer hidden |
| 4 | Read submission before reveal | Plaintext answer is empty |
| 5 | Second commitment from same address | Reverts |
| 6 | Reveal before submission deadline passes | Reverts |
| 7 | Reveal with wrong answer/salt (hash mismatch) | Reverts |
| 8 | Reveal from address that never committed (Mallory) | Reverts |
| 9 | Valid reveal matching commitment | Accepted, answer now visible |
| 10 | Reveal twice | Reverts |
| 11 | Winner index that never revealed | Would revert in `finalizeWinner` |

## Not covered locally

`judgeAll()` calls `LLM_INFERENCE_PRECOMPILE`, which only exists on Ritual Chain.
Not testable on a local Hardhat network — verified by code review and would be
exercised on Ritual testnet directly.
