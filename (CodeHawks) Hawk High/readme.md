# Audit-2: Codehawks First Flights Hawk High

Repo: https://github.com/azriellee/2025-05-hawk-high.git

## Audit Experience:

This codebase was a really interesting test I guess? All the high vulnerabilities I found were about breaking the invariants, and I think the code base was designed with too many business logic flaws. Some business errors were so bad that they spanned across multiple findings and it was quite hard to suggest a mitigation.

I think it was good practice for me to read and understand the code base and intended functionalities, and test against them. However I would have appreciated more findings related to vulnerable smart contract practices. There could potentially be a vulnerability related to the logic contract not disabling initializers, however I could not find a way to exploit this.

## Findings Summary:

7 High, 0 Medium, 1 Low, 6 Info/Gas

[H-1] `LevelOne:graduateAndUpgrade` Does Not Perform Actual Upgrade Or Initialization
[H-2] `LevelOne:graduateAndUpgrade` Can Be Called Before `sessionEnd` Reached
[H-3] `LevelOne:graduateAndUpgrade` Does Not Filter Students Below Cutoff
[H-4] Payment Disrbursement When `LevelOne:graduateAndUpgrade` Called Is Incorrect
[H-5] `LevelOne:graduateAndUpgrade` Lacks Review Count Check
[H-6] `LevelOne:giveReview` Does Not Increment `reviewCount`
[H-7] Mismatch In Storage Layout Of `LevelOne` And `LevelTwo`
[L-1] `LevelOne:initialize` Not Protected

## Results Summary:

Rank: 8
Results: 4 High, 1 Medium, 2 Low

[H] `LevelOne:graduateAndUpgrade` Does Not Perform Actual Upgrade Or Initialization
[H] `LevelOne:graduateAndUpgrade` Does Not Filter Students Below Cutoff
[H] Payment Disrbursement When `LevelOne:graduateAndUpgrade` Called Is Incorrect
[H] `LevelOne:graduateAndUpgrade` Lacks Review Count Check
[M] Mismatch In Storage Layout Of `LevelOne` And `LevelTwo`
[L] `LevelOne:giveReview` Does Not Increment `reviewCount`
[L] `LevelOne:graduateAndUpgrade` Can Be Called Before `sessionEnd` Reached

### Severe Findings Missed:

[H] No Mechanism to handle remaining bursary funds locked within contract
[M] Unprotected Initialization
