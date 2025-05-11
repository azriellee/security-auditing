# Audit-1: Codehawks First Flights Rock Paper Scissors

Repo: https://github.com/azriellee/2025-04-rock-paper-scissors.git

## Findings Summary:

1 High, 2 Medium, 5 Info/Gas

[H-1]RockPaperScissors::joinGameWithEth Allows Users to Join Token Games Without Paying
[M-1] Funds/Token Lockup Risk Due to Unresponsive Opponent After
Move Commit
[M-2] Improper Validation in RockPaperScissors::timeoutReveal Allows Premature Game Forfeits

## Results Summary:

2 High, 1 Medium, 5 Info/Gas

(1 Medium Severity Finding was upgraded to High)

[H-1]RockPaperScissors::joinGameWithEth Allows Users to Join Token Games Without Paying
[H-2] Improper Validation in RockPaperScissors::timeoutReveal Allows Premature Game Forfeits
[M-1] Funds/Token Lockup Risk Due to Unresponsive Opponent After Move Commit

### Findings Missed

Total Findings: 5 High, 3 Medium

[H1] PlayerB Address Can Be Overwritten in joinGameWithEth and joinGameWithToken Functions
[H3] Denial of Service via ETH Transfer Revert
[H5] RevealMove before Another Player CommitMove Makes Hacker Win All Multi-Turn Games
