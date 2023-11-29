# First Flight #2: Puppy Raffle
<br />

## Goals

- [x] Check that duplicated addresses can only participate once.
<br>

- [x] Check all variables are well used and unchanged variables remain immutable.
<br>

- [x] Check the refund logic. Special attention to where the funds come from, conditions of refund, and that it is not reentrant.
<br>

- [x] Check that fee address can only be set by the owner.
<br>

- [x] Check that there are enough funds for any refunds, prices and fees. Also that there is no case where the owner cant withdraw fees.
<br>

## Notes

- Call the enterRaffle function with the following parameters:
  - address[] participants: A list of addresses that enter. You can use this to enter yourself multiple times, or yourself and a group of your friends.
<br>

- Duplicate addresses are not allowed
<br>

- Users are allowed to get a refund of their ticket & value if they call the refund function
<br>

- Every X seconds, the raffle will be able to draw a winner and be minted a random puppy
<br>

- The owner of the protocol will set a feeAddress to take a cut of the value, and the rest of the funds will be sent to the winner of the puppy.
<br>

## Variables Tree

```solidity
    uint256 public immutable entranceFee;

    address[] public players;
    uint256 public raffleDuration;
    uint256 public raffleStartTime;
    address public previousWinner;

    // We do some storage packing to save gas
    address public feeAddress;
    uint64 public totalFees = 0;

    // mappings to keep track of token traits
    mapping(uint256 => uint256) public tokenIdToRarity;
    mapping(uint256 => string) public rarityToUri;
    mapping(uint256 => string) public rarityToName;

    // Metadata
    string private commonImageUri = "ipfs://QmSsYRx3LpDAb1GZQm7zZ1AuHZjfbPkD6J7s9r41xu1mf8";
    uint256 public constant COMMON_RARITY = 70;
    string private constant COMMON = "common";

    string private rareImageUri = "ipfs://QmUPjADFGEKmfohdTaNcWhp7VGk26h5jXDA7v3VtTnTLcW";
    uint256 public constant RARE_RARITY = 25;
    string private constant RARE = "rare";

    string private legendaryImageUri = "ipfs://QmYx6GsYAKnNzZ9A6NvEKV9nf1VaDzJrqDR23Y8YSkebLU";
    uint256 public constant LEGENDARY_RARITY = 5;
    string private constant LEGENDARY = "legendary";
```

## Roles

- Owner: Deployer of the protocol, has the power to change the wallet address to which fees are sent through the changeFeeAddress function.
<br>

- Player: Participant of the raffle, has the power to enter the raffle with the enterRaffle function and refund value through refund function.
<br>

## Findings

- PuppyRaffle::raffleDuration can be set to immutable.
<br>

- PuppyRaffle::tokenIdToRarity ; PuppyRaffle::rarityToUri ; PuppyRaffle::rarityToName can be packed into a single mapping by creating a rarity struct.
<br>

- PuppyRaffle::totalFees can have an overflow. We should consider using uint256 instead of uint64.
<br>

- PuppyRaffle::refund is a reentrant function.
<br>

- PuppyRaffle::getActivePlayerIndex , player in position 0 can be confused as an inactive player.
<br>

- PuppyRaffle::selectWinner , as anyonce can call this function and its winner and rarity are determined by onchain data, anyone could call this function when they know that they will be the winner or that a certain rarity will be given. Making this only callable by the owner would still be a bad idea since an attacker can read the mempool and know right before the owner calls the function wether they will be the winner. If they are not, they can refund their ticket and make a risk-free raffle.
<br>

- PuppyRaffle::tokenURI can be uploaded to IPFS to save gas.
<br>

- PuppyRaffle::selectWinner will never be used. We can remove it to save deployment gas.
<br>

- PuppyRaffle::withdrawFees can easily be blocked by another user by sending an insignificant amount of ether to the contract. Since the balance of the contract will never match the totalFees, the owner will never be able to withdrawFees.
<br>

## Post Audit Notes

- Always question where the funds come from when sending to an address.
<br>

- Always question how much funds are left when making a move and check if there can be a case where there are not enough funds.
<br>

- Always question where the funds go when sending to an address. Avoid address(0) and handle cases where the desired address can't receive funds.