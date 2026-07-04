# Reflection

What should be public, what should stay hidden, and what should be decided by AI versus by
a human in a bounty system?

The bounty's existence, reward, rubric, and deadlines should always be public — participants
need to trust the rules before committing effort. The answers themselves should stay hidden
during the submission window and only become visible once judging is locked in, otherwise the
system just rewards whoever submits last. What should stay hidden the longest is anything an
answer might leak about a participant's identity or approach before the contest is decided —
that's the whole point of commit-reveal. AI should handle the repetitive, consistent part:
scoring every submission against the same rubric in one batch pass, which no bored human judge
can do fairly at scale. But AI should never have the final say on payment — a human owner
should review the AI's ranking and reasoning before finalizing, because the AI can be wrong,
gamed by prompt injection in an answer, or simply miss context a human would catch. In short:
rules public, answers hidden until it's fair to reveal them, AI proposes, human disposes.
