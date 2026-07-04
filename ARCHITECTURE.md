# Architecture Note: Commit-Reveal vs. Ritual-Native Encrypted Submissions

## Commit-Reveal (implemented)

- **Where plaintext exists:** nowhere on-chain until the reveal transaction. It exists only in
  the participant's own hands (and their local machine/wallet) before that.
- **What's stored on-chain:** a `bytes32` hash during commit; the plaintext `string` answer
  once revealed (public from that point on, before judging finishes).
- **How the LLM sees it:** the owner reads all revealed answers off-chain, builds one batch
  prompt, and sends it via `judgeAll`.
- **Limitation:** between `revealAnswer` and `judgeAll`, answers ARE public on-chain. A
  participant who reveals early can still be copied by someone who hasn't revealed yet, as
  long as it's before their own reveal tx lands. This is a real gap in a public mempool — the
  required track accepts this as a known trade-off of a fully generic EVM solution.

## Ritual-Native Encrypted Submissions (advanced track, design only)

- **Where plaintext exists:** only inside the TEE executor at judging time. It is decrypted
  in-memory inside the enclave, never written to public chain state, and never visible to the
  owner or other participants.
- **What's stored on-chain:** an encrypted blob (or a reference/hash to one stored off-chain,
  e.g. via a storage precompile), keyed per participant. No commit/reveal step is needed
  because the ciphertext itself can safely sit in public storage.
- **How the LLM receives submissions:** at `judgeAll` time, a TEE-backed precompile call
  decrypts all submissions inside the enclave and forwards them together to the LLM in one
  batch request — same "judge all at once" requirement as the required track, but the
  plaintext never touches any public state or mempool, before or after commit.
- **How the final reveal happens:** after judging, the system can publish a bundle of the
  original answers (e.g. to IPFS) and store only `revealedAnswersHash` on-chain, so anyone
  can later verify the published answers weren't tampered with.
- **How the contract verifies the bundle:** compare `keccak256(offchainBundle)` against
  `revealedAnswersHash` stored at judging time.

## Trade-off Summary

| | Commit-Reveal | Ritual-Native (TEE) |
|---|---|---|
| Works on any EVM chain | Yes | No (Ritual-specific) |
| Answers ever fully hidden pre-judging | Partially (gap after reveal) | Yes, until judging completes |
| Extra step for participants | Yes (must return to reveal) | No (submit once) |
| Trust assumption | Cryptographic hash only | Hash + TEE attestation |
| Implementation complexity | Low | Higher (precompile + off-chain bundle) |

Commit-reveal is the right default for "works everywhere, simple to reason about." The
Ritual-native approach removes the post-reveal exposure window entirely and is a better fit
once you're committed to building specifically on Ritual.
