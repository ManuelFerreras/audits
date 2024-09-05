# First Flight #23: MyCut

## Goals

- [] Check that only allowed claimants can claim during a 90 days period.

<br>

- [] Admin can take a cut only once the 90 days period ends and only once.

<br>

- [] The remainder is distributed only amongst those who claimed during the 90 days period and equally.

<br>

## Notes

`notes`

<br>

## Variables Tree

```solidity
    address[] public contests;
    mapping(address => uint256) public contestToTotalRewards;

    address[] private i_players;
    uint256[] private i_rewards;
    address[] private claimants;
    uint256 private immutable i_totalRewards;
    uint256 private immutable i_deployedAt;
    IERC20 private immutable i_token;
    mapping(address => uint256) private playersToRewards;
    uint256 private remainingRewards;
    uint256 private constant managerCutPercent = 10;
```

## Roles

- Owner/Admin (Trusted) - Is able to create new Pots, close old Pots when the claim period has elapsed and fund Pots

<br>

- User/Player - Can claim their cut of a Pot

<br>

## Findings

- No events are being emitted for critical actions in both contracts. Some that can be added:

  - ContestManager::ContestCreated
  - ContestManager::ContestFunded
  - ContestManager::ContestClosed

  - Pot::CutClaimed
  - Pot::PotClosed

<br>

# Potential Loss of Funds During a Pot Close Event

## Summary

In the `closePot` function, when a pot is closed, there can be a difference between the sum of the claimant cuts and the real funds to be claimed.

## Vulnerability Details

In the `closePot` function, when a pot is closed, since the `claimantCut` is calculated using the `i_players.length` variable, it can lead to a difference between the length of `i_players` and the length of `claimants`, resulting in a fund loss.

Let's say that for a pot of 50 tokens for `Alice` and 100 tokens for `John` the only claimant is `Alice`, once the pot is closed, from the 100 tokens left in the pot:

- 10 tokens will go for the manager cut.

- 90 tokens will be distributed to the claimants with the formula:
  `uint256 claimantCut = (remainingRewards - managerCut) / i_players.length;`

So in this example, we have 100 tokens as remaining rewards, 10 tokens as manager cut, and 2 players in the pot. The claimant cut will be 45 tokens, but since there is only one claimant, the remaining 45 tokens will be lost.

## Impact

Loss of funds if there are players that never claimed their cut.

## Tools Used

Manual Revision

## Recommendations

Instead of using `i_players.length` use `claimants.length` to calculate the real claimant cut.

## Post Audit Notes
