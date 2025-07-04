# Audit-4: Codehawks Snowman Merkle Airdrop

Repo: https://github.com/azriellee/2025-06-snowman-merkle-airdrop.git

## Audit Experience:

This was a really good experience, teaching me a lot on NFTs and airdrops. In particular, I learnt about how merkle trees are used in airdrops to efficiently verify users and the amount of NFTs they should be able to claim. In terms of auditing, I mainly found business logic flaws as usual. I was not able to find any exploitable issues with the merkle tree generation or the signature use, though it is worth noting that replay attacks were possible just that there would be very minute impacts.

## Findings Summary:

4 High, 0 Medium, 1 Low, 6 Info/Gas

[H-1] `Snowman::mintSnowman()` Does Not Control Caller Allowing Anyone To Mint Arbitrary Number of NFTs To Themselves
[H-2] `Snow::earnSnow()` Function Uses Shared `s_earnTimer` Allowing Only One Address To Earn Snow Per Week
[H-3] `Snow::buySnow()` Updates Shared `s_earnTimer` Variable Causing Denial Of Service Within `Snow::earnSnow()`
[H-4] `SnowmanAirdrop::claimSnowman()` Uses Current Token Balance For Merkle Proof Making It Susceptible to DoS
[L-1] `Snow::buySnow()` Expects Exact Amount Of Ether, Resulting In Unexpected Reverts / WETH Transfer

## Results Summary:

Rank: 34
Results: 1 High, 1 Medium, 1 Low

[H-1] `Snowman::mintSnowman()` Does Not Control Caller Allowing Anyone To Mint Arbitrary Number of NFTs To Themselves
[L-1] `Snow::buySnow()` Updates Shared `s_earnTimer` Variable Causing Denial Of Service Within `Snow::earnSnow()`
[M-1] `SnowmanAirdrop::claimSnowman()` Uses Current Token Balance For Merkle Proof Making It Susceptible to DoS

[Invalid] `Snow::earnSnow()` Function Uses Shared `s_earnTimer` Allowing Only One Address To Earn Snow Per Week (Expected Behaviour)
[Invalid] `Snow::buySnow()` Expects Exact Amount Of Ether, Resulting In Unexpected Reverts / WETH Transfer (Non-acceptable Severity)

### Severe Findings Missed:

[H] Inconsistent `MESSAGE_TYPEHASH` with standard EIP-712 declaration on contract `SnowmanAirdrop`

Things to note: I should be extra careful with message hashes in future contracts, though this should also be easily spotted with better testing as it renders a function useless.
