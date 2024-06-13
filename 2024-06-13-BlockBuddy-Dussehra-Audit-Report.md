<h1 align="center">BlockBuddy Audit Report</h1>

Prepared by: [BlockBuddy](https://github.com/kingmanas)
Lead Auditors:

- BlockBuddy

# Table of Contents

- [Table of Contents](#table-of-contents)
- [Protocol Summary](#protocol-summary)
- [Disclaimer](#disclaimer)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
  - [Scope](#scope)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
- [Findings :](#findings-)
  - [High](#high)
    - [H-1: In `RamNFT.sol` an arbitrary address is used , instead of using `ChoosingRam` address in `setChoosingRamContract` function](#h-1-in-ramnftsol-an-arbitrary-address-is-used--instead-of-using-choosingram-address-in-setchoosingramcontract-function)
    - [H-2: Anyone can mint the RamNFT in `mintRamNFT` function , as it has no check for who can mint the RamNFT](#h-2-anyone-can-mint-the-ramnft-in-mintramnft-function--as-it-has-no-check-for-who-can-mint-the-ramnft)
    - [H-3: In `RamNFT::mintRamNFT` there is a possibility of Reentrancy as effects are done after an external call](#h-3-in-ramnftmintramnft-there-is-a-possibility-of-reentrancy-as-effects-are-done-after-an-external-call)
    - [H-4: In `Dussehra::killRavana` , it sends the eth to an arbitrary address(user)](#h-4-in-dussehrakillravana--it-sends-the-eth-to-an-arbitrary-addressuser)
  - [Medium](#medium)
    - [M-1: Weak PRNG in `ChoosingRam.sol`](#m-1-weak-prng-in-choosingramsol)
    - [M-2: Reentrancy in `Dussehra.sol::withdraw` function , state update after an external call](#m-2-reentrancy-in-dussehrasolwithdraw-function--state-update-after-an-external-call)
  - [Lows](#lows)
    - [L-1: Missing checks for `address(0)` when assigning values to address state variables](#l-1-missing-checks-for-address0-when-assigning-values-to-address-state-variables)
    - [L-2: `public` functions not used internally could be marked `external`](#l-2-public-functions-not-used-internally-could-be-marked-external)
    - [L-3: Define and use `constant` variables instead of using literals](#l-3-define-and-use-constant-variables-instead-of-using-literals)
    - [L-4: Event is missing `indexed` fields](#l-4-event-is-missing-indexed-fields)
    - [L-5: PUSH0 is not supported by all chains](#l-5-push0-is-not-supported-by-all-chains)
    - [L-6: Modifiers invoked only once can be shoe-horned into the function](#l-6-modifiers-invoked-only-once-can-be-shoe-horned-into-the-function)
    - [L-7: State variables defined only once should be marked `immutable`](#l-7-state-variables-defined-only-once-should-be-marked-immutable)
    - [L-8 In `RamNFT::updateCharacteristics` we can change the `tokenId` also](#l-8-in-ramnftupdatecharacteristics-we-can-change-the-tokenid-also)
    - [L-9: Potential Reentrancy in `ChoosingRam::increaseValuesOfParticipants` , as it does not follows CEI](#l-9-potential-reentrancy-in-choosingramincreasevaluesofparticipants--as-it-does-not-follows-cei)
  - [Informational](#informational)
    - [I-1 : `Dussehra.sol , ChoosingRam.sol` uses timestamp for comparison which can be manipulated by the miners](#i-1--dussehrasol--choosingramsol-uses-timestamp-for-comparison-which-can-be-manipulated-by-the-miners)
    - [I-2: Name the variables accordingly](#i-2-name-the-variables-accordingly)
    - [I-3: Unused state variables](#i-3-unused-state-variables)
    - [I-3: In `ChoosingRam::increaseValuesOfParticipants` function booleans are compared with booloean constants](#i-3-in-choosingramincreasevaluesofparticipants-function-booleans-are-compared-with-booloean-constants)
    - [I-4: In `ChoosingRam::increaseValuesOfParticipants` , the function has become very complex and has a complexity score of more than 11](#i-4-in-choosingramincreasevaluesofparticipants--the-function-has-become-very-complex-and-has-a-complexity-score-of-more-than-11)

# Protocol Summary

The protocol is based on the hindufestival of Dussehra where Lord Rama killed Ravana as a symbolism of victory of good over evil. The protocol presents it as a game where we can choose our ram , mint that NFT of our Ram and eventually kill Ravana. The protocol demontrates it in a gamify way with the help of smart contracts.

# Disclaimer

The BlockBuddy team makes all effort to find as many vulnerabilities in the code in the given time period, but holds no responsibilities for the findings provided in this document. A security audit by the team is not an endorsement of the underlying business or product. The audit was time-boxed and the review of the code was solely on the security aspects of the Solidity implementation of the contracts.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

# Audit Details

## Scope

## Roles

# Executive Summary

## Issues found

| Severtity | Number of issues found |
| --------- | ---------------------- |
| High      | 4                      |
| Medium    | 2                      |
| Low       | 9                      |
| Info      | 4                      |
| Total     | 19                     |

# Findings :

## High

### H-1: In `RamNFT.sol` an arbitrary address is used , instead of using `ChoosingRam` address in `setChoosingRamContract` function

**Description**

```javascript
-->>  address public choosingRamContract;

-->>  function setChoosingRamContract(address _choosingRamContract) public onlyOrganiser {
        choosingRamContract = _choosingRamContract;
    }
```

**Impact** Any arbitrary address can become the ChoosingRam contract and thus can do malicious things in the contract for their benefit.

**Proof Of Code**

<details>
<summary>PoC</summary>

In `Dussehra.t.sol` add this test

```javascript
function testAnyAddressInsteadOfChoosingRamAddress() public {
        address attacker = makeAddr("attacker");
        vm.startPrank(organiser);
        ramNFT.setChoosingRamContract(attacker);
        vm.stopPrank();
    }
```

</details>

**Recommended Mitigation**

```diff
+ import {ChoosingRam} from "./ChoosingRam .sol";

- address public choosingRamContract;
+ ChoosingRam public choosingRamContract;

- function setChoosingRamContract(address _choosingRamContract) public onlyOrganiser {
        choosingRamContract = _choosingRamContract;
    }
+ function setChoosingRamContract(ChoosingRam _choosingRamContract) public onlyOrganiser {
        choosingRamContract = _choosingRamContract;
    }
```

### H-2: Anyone can mint the RamNFT in `mintRamNFT` function , as it has no check for who can mint the RamNFT

**Description**

As we can clearly see below that the mint function doesn't have any check for who can mint the RamNFT

```javascript
-->> function mintRamNFT(address to) public {
        uint256 newTokenId = tokenCounter++;
        _safeMint(to, newTokenId);

        Characteristics[newTokenId] = CharacteristicsOfRam({
            ram: to,
            isJitaKrodhah: false,
            isDhyutimaan: false,
            isVidvaan: false,
            isAatmavan: false,
            isSatyavaakyah: false
        });
    }
```

**Impact**
Anyone can mint the RamNFT , which may cause loss to the protocol

**Proof Of Code**

<details>
<summary> PoC </summary>

In `Dussehra.t.sol` add this test

```javacript
function mintRamNFT(address to) public {
        uint256 newTokenId = tokenCounter++;
        _safeMint(to, newTokenId);

        Characteristics[newTokenId] = CharacteristicsOfRam({
            ram: to,
            isJitaKrodhah: false,
            isDhyutimaan: false,
            isVidvaan: false,
            isAatmavan: false,
            isSatyavaakyah: false
        });
    }
```

</details>

**Recommended Mitigation**

1. Use the `onlyOragniser` modifier or the `onlyChoosingRamContract`
2. Have some checks for who can mint the `RamNFT`

### H-3: In `RamNFT::mintRamNFT` there is a possibility of Reentrancy as effects are done after an external call

**Description**

We can clearly see that effects are done after an external `safeMint` call

```javascript
 function mintRamNFT(address to) public {
        uint256 newTokenId = tokenCounter++;
-->        _safeMint(to, newTokenId);

-->        Characteristics[newTokenId] = CharacteristicsOfRam({
            ram: to,
            isJitaKrodhah: false,
            isDhyutimaan: false,
            isVidvaan: false,
            isAatmavan: false,
            isSatyavaakyah: false
        });
    }
```

**Impact**
Someone can mint the RamNFT again and agian without changing the characterstics

**Recommended Mitigation**

1. Use Reentrancy Guard
2. Make the function follow CEI to prevent it

```diff
 function mintRamNFT(address to) public {
        uint256 newTokenId = tokenCounter++;
+        Characteristics[newTokenId] = CharacteristicsOfRam({
+            ram: to,
+            isJitaKrodhah: false,
+            isDhyutimaan: false,
+            isVidvaan: false,
+            isAatmavan: false,
+            isSatyavaakyah: false
+        });
-        _safeMint(to, newTokenId);

-        Characteristics[newTokenId] = CharacteristicsOfRam({
-            ram: to,
-            isJitaKrodhah: false,
-            isDhyutimaan: false,
-            isVidvaan: false,
-            isAatmavan: false,
-            isSatyavaakyah: false
-        });
+         _safeMint(to, newTokenId);
    }
```

### H-4: In `Dussehra::killRavana` , it sends the eth to an arbitrary address(user)

**Description**

```javascript
function killRavana() public RamIsSelected {

        if (block.timestamp < 1728691069) {
            revert Dussehra__MahuratIsNotStart();
        }
        if (block.timestamp > 1728777669) {
            revert Dussehra__MahuratIsFinished();
        }
        IsRavanKilled = true;
        uint256 totalAmountByThePeople = WantToBeLikeRam.length * entranceFee;
        totalAmountGivenToRam = (totalAmountByThePeople * 50) / 100;
-->        (bool success, ) = organiser.call{value: totalAmountGivenToRam}("");
        require(success, "Failed to send money to organiser");
    }
```

**Impact**

Some exploiter can send the reward to himself rather than it going to the correct user

**Recommended Mitigation**

1. Make the address of the user whose reward it is rather than having any arbitrary one recieving it.

## Medium

### M-1: Weak PRNG in `ChoosingRam.sol`

**Description**

```javascript
--> uint256 random =
            uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, msg.sender))) % 2;

--> uint256 random = uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao))) % ramNFT.tokenCounter();
```

**Impact**

Anyone can manipulate the random number for their benefit

**Recommended Mitigation**

1. Use ChainLink VRF

### M-2: Reentrancy in `Dussehra.sol::withdraw` function , state update after an external call

**Description**

```javascript
function withdraw() public RamIsSelected OnlyRam RavanKilled {
        if (totalAmountGivenToRam == 0) {
            revert Dussehra__AlreadyClaimedAmount();
        }
        uint256 amount = totalAmountGivenToRam;
-->        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Failed to send money to Ram");
-->        totalAmountGivenToRam = 0;
    }
```

**Impact**

Someone can enter the function again & again and do some mischief before state change

**Recommended Mitigation**

1. Use Reentrancy Guard.
2. Make the function follow CEI.

```diff
function withdraw() public RamIsSelected OnlyRam RavanKilled {
        if (totalAmountGivenToRam == 0) {
            revert Dussehra__AlreadyClaimedAmount();
        }
        uint256 amount = totalAmountGivenToRam;
+       totalAmountGivenToRam = 0;
-       (bool success, ) = msg.sender.call{value: amount}("");
-        require(success, "Failed to send money to Ram");
-       totalAmountGivenToRam = 0;
+       (bool success, ) = msg.sender.call{value: amount}("");
+        require(success, "Failed to send money to Ram");
    }
```

## Lows

### L-1: Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

<details><summary>4 Found Instances</summary>

- Found in src/ChoosingRam.sol [Line: 32](src/ChoosingRam.sol#L32)

  ```solidity
          ramNFT = RamNFT(_ramNFT);
  ```

- Found in src/Dussehra.sol [Line: 50](src/Dussehra.sol#L50)

  ```solidity
          ramNFT = RamNFT(_ramNFT);
  ```

- Found in src/Dussehra.sol [Line: 51](src/Dussehra.sol#L51)

  ```solidity
          choosingRamContract = ChoosingRam(_choosingRamContract);
  ```

- Found in src/RamNFT.sol [Line: 48](src/RamNFT.sol#L48)

  ```solidity
          choosingRamContract = _choosingRamContract;
  ```

</details>

### L-2: `public` functions not used internally could be marked `external`

Instead of marking a function as `public`, consider marking it as `external` if it is not used internally.

<details><summary>10 Found Instances</summary>

- Found in src/ChoosingRam.sol [Line: 35](src/ChoosingRam.sol#L35)

  ```solidity
      function increaseValuesOfParticipants(uint256 tokenIdOfChallenger, uint256 tokenIdOfAnyPerticipent)
  ```

- Found in src/ChoosingRam.sol [Line: 85](src/ChoosingRam.sol#L85)

  ```solidity
      function selectRamIfNotSelected() public RamIsNotSelected OnlyOrganiser {
  ```

- Found in src/Dussehra.sol [Line: 54](src/Dussehra.sol#L54)

  ```solidity
      function enterPeopleWhoLikeRam() public payable {
  ```

- Found in src/Dussehra.sol [Line: 69](src/Dussehra.sol#L69)

  ```solidity
      function killRavana() public RamIsSelected {
  ```

- Found in src/Dussehra.sol [Line: 83](src/Dussehra.sol#L83)

  ```solidity
      function withdraw() public RamIsSelected OnlyRam RavanKilled {
  ```

- Found in src/RamNFT.sol [Line: 47](src/RamNFT.sol#L47)

  ```solidity
      function setChoosingRamContract(address _choosingRamContract) public onlyOrganiser {
  ```

- Found in src/RamNFT.sol [Line: 51](src/RamNFT.sol#L51)

  ```solidity
      function mintRamNFT(address to) public {
  ```

- Found in src/RamNFT.sol [Line: 65](src/RamNFT.sol#L65)

  ```solidity
      function updateCharacteristics(
  ```

- Found in src/RamNFT.sol [Line: 84](src/RamNFT.sol#L84)

  ```solidity
      function getCharacteristics(uint256 tokenId) public view returns (CharacteristicsOfRam memory) {
  ```

- Found in src/RamNFT.sol [Line: 88](src/RamNFT.sol#L88)

  ```solidity
      function getNextTokenId() public view returns (uint256) {
  ```

</details>

### L-3: Define and use `constant` variables instead of using literals

If the same constant literal value is used multiple times, create a constant state variable and reference it throughout the contract.

<details><summary>2 Found Instances</summary>

- Found in src/ChoosingRam.sol [Line: 49](src/ChoosingRam.sol#L49)

  ```solidity
          if (block.timestamp > 1728691200) {
  ```

- Found in src/ChoosingRam.sol [Line: 86](src/ChoosingRam.sol#L86)

  ```solidity
          if (block.timestamp < 1728691200) {
  ```

</details>

### L-4: Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

<details><summary>1 Found Instances</summary>

- Found in src/Dussehra.sol [Line: 30](src/Dussehra.sol#L30)

  ```solidity
      event PeopleWhoLikeRamIsEntered(address competitor);
  ```

</details>

### L-5: PUSH0 is not supported by all chains

Solc compiler version 0.8.20 switches the default target EVM version to Shanghai, which means that the generated bytecode will include PUSH0 opcodes. Be sure to select the appropriate EVM version in case you intend to deploy on a chain other than mainnet like L2 chains that may not support PUSH0, otherwise deployment of your contracts will fail.

<details><summary>4 Found Instances</summary>

- Found in src/ChoosingRam.sol [Line: 2](src/ChoosingRam.sol#L2)

  ```solidity
  pragma solidity 0.8.20;
  ```

- Found in src/Dussehra.sol [Line: 2](src/Dussehra.sol#L2)

  ```solidity
  pragma solidity 0.8.20;
  ```

- Found in src/RamNFT.sol [Line: 2](src/RamNFT.sol#L2)

  ```solidity
  pragma solidity 0.8.20;
  ```

- Found in src/mocks/mock.sol [Line: 2](src/mocks/mock.sol#L2)

  ```solidity
  pragma solidity 0.8.20;
  ```

</details>

### L-6: Modifiers invoked only once can be shoe-horned into the function

<details><summary>5 Found Instances</summary>

- Found in src/ChoosingRam.sol [Line: 25](src/ChoosingRam.sol#L25)

  ```solidity
      modifier OnlyOrganiser() {
  ```

- Found in src/Dussehra.sol [Line: 37](src/Dussehra.sol#L37)

  ```solidity
      modifier OnlyRam() {
  ```

- Found in src/Dussehra.sol [Line: 42](src/Dussehra.sol#L42)

  ```solidity
      modifier RavanKilled() {
  ```

- Found in src/RamNFT.sol [Line: 28](src/RamNFT.sol#L28)

  ```solidity
      modifier onlyOrganiser() {
  ```

- Found in src/RamNFT.sol [Line: 35](src/RamNFT.sol#L35)

  ```solidity
      modifier onlyChoosingRamContract() {
  ```

</details>

### L-7: State variables defined only once should be marked `immutable`

<details>
<summary>Some instances</summary>

In `Dussehra.sol`

```diff
-   uint256 public entranceFee;
-   address public organiser;
-   RamNFT public ramNFT;
-   ChoosingRam public choosingRamContract;

+   uint256 public constant  entranceFee;
+   address public constant organiser;
+   RamNFT public constant- ramNFT;
+   ChoosingRam public constant choosingRamContract;
```

In `RamNFT.sol`

```diff
- address public organiser;
+ address public constant organiser;
```

</details>

### L-8 In `RamNFT::updateCharacteristics` we can change the `tokenId` also

**Description**

```javascript
 function updateCharacteristics(
-->       uint256 tokenId,
       bool _isJitaKrodhah,
       bool _isDhyutimaan,
       bool _isVidvaan,
       bool _isAatmavan,
       bool _isSatyavaakyah
   ) public onlyChoosingRamContract {

       Characteristics[tokenId] = CharacteristicsOfRam({
-->            ram: Characteristics[tokenId].ram,
           isJitaKrodhah: _isJitaKrodhah,
           isDhyutimaan: _isDhyutimaan,
           isVidvaan: _isVidvaan,
           isAatmavan: _isAatmavan,
           isSatyavaakyah: _isSatyavaakyah
       });
   }
```

**Impact**

It may lead to someone changing their tokenId with others , which may lead to confusion nad ownership issues

**Recommended Mitigation**

1. TokenId should not be updated wuth characteristics
2. There should be a check that only the owner of that RamNFT is able to change it

### L-9: Potential Reentrancy in `ChoosingRam::increaseValuesOfParticipants` , as it does not follows CEI

**Description**

```javascript
 function increaseValuesOfParticipants(uint256 tokenIdOfChallenger, uint256 tokenIdOfAnyPerticipent)
        public
        RamIsNotSelected
    {
        if (tokenIdOfChallenger > ramNFT.tokenCounter()) {
            revert ChoosingRam__InvalidTokenIdOfChallenger();
        }
        if (tokenIdOfAnyPerticipent > ramNFT.tokenCounter()) {
            revert ChoosingRam__InvalidTokenIdOfPerticipent();
        }
        if (ramNFT.getCharacteristics(tokenIdOfChallenger).ram != msg.sender) {
            revert ChoosingRam__CallerIsNotChallenger();
        }


        if (block.timestamp > 1728691200) {
            revert ChoosingRam__TimeToBeLikeRamFinish();
        }
         // answered
        uint256 random =
            uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, msg.sender))) % 2;

        // answered
        if (random == 0) {
            if (ramNFT.getCharacteristics(tokenIdOfChallenger).isJitaKrodhah == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, false, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isDhyutimaan == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isVidvaan == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isAatmavan == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isSatyavaakyah == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, true);
-->                selectedRam = ramNFT.getCharacteristics(tokenIdOfChallenger).ram;
            }
        } else {
            if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isJitaKrodhah == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, false, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isDhyutimaan == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isVidvaan == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isAatmavan == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, true, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isSatyavaakyah == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, true, true);
-->                selectedRam = ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).ram;
            }
        }
    }
```

**Recommended Mitigation**

1. Use Reentrancy Guard
2. Make the function follow CEI

## Informational

### I-1 : `Dussehra.sol , ChoosingRam.sol` uses timestamp for comparison which can be manipulated by the miners

**Description**

```javascript
-->>       if (block.timestamp > 1728691200) {
            revert ChoosingRam__TimeToBeLikeRamFinish();
        }

	  function killRavana() public RamIsSelected {

-->>        if (block.timestamp < 1728691069) {
            revert Dussehra__MahuratIsNotStart();
        }
-->>        if (block.timestamp > 1728777669) {
            revert Dussehra__MahuratIsFinished();
        }
```

**Impact** Can be maipulated by the miner for his benefit..!!

**Recommended Mitigation**

1. Use constant varialbes intead of literals
2. Rather than using a hardcoded timestamp make it such that it should be chnaged according to the need but only by the owner.

### I-2: Name the variables accordingly

<details>
<summary> Some Instances </summary>

```diff
- mapping(uint256 tokenId => CharacteristicsOfRam) public Characteristics;
+ mapping(uint256 tokenId => CharacteristicsOfRam) public IdToCharacteristics;
```

</details>

### I-3: Unused state variables

<details>

<summary> Some Instances </summary>

In `Dussehra.sol`, SelectedRam is not used anywhere

```javascript
address public SelectedRam;
```

</details>

### I-3: In `ChoosingRam::increaseValuesOfParticipants` function booleans are compared with booloean constants

<details>
<summary> Some Instances </summary>

```javascript
if (random == 0) {
  if (ramNFT.getCharacteristics(tokenIdOfChallenger).isJitaKrodhah == false) {
    ramNFT.updateCharacteristics(
      tokenIdOfChallenger,
      true,
      false,
      false,
      false,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfChallenger).isDhyutimaan == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfChallenger,
      true,
      true,
      false,
      false,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfChallenger).isVidvaan == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfChallenger,
      true,
      true,
      true,
      false,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfChallenger).isAatmavan == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfChallenger,
      true,
      true,
      true,
      true,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfChallenger).isSatyavaakyah == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfChallenger,
      true,
      true,
      true,
      true,
      true
    );
    selectedRam = ramNFT.getCharacteristics(tokenIdOfChallenger).ram;
  }
} else {
  if (
    ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isJitaKrodhah == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfAnyPerticipent,
      true,
      false,
      false,
      false,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isDhyutimaan == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfAnyPerticipent,
      true,
      true,
      false,
      false,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isVidvaan == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfAnyPerticipent,
      true,
      true,
      true,
      false,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isAatmavan == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfAnyPerticipent,
      true,
      true,
      true,
      true,
      false
    );
  } else if (
    ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isSatyavaakyah == false
  ) {
    ramNFT.updateCharacteristics(
      tokenIdOfAnyPerticipent,
      true,
      true,
      true,
      true,
      true
    );
    selectedRam = ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).ram;
  }
}
```

</details>

**Recommended Mitigation**

Booleans can directly be checked for true or false rather than comparing them to a boolean constant

### I-4: In `ChoosingRam::increaseValuesOfParticipants` , the function has become very complex and has a complexity score of more than 11

**Description**

We can clearly see that then function is very complex due to a lot comparisons and state change

```javascript
function increaseValuesOfParticipants(uint256 tokenIdOfChallenger, uint256 tokenIdOfAnyPerticipent)
        public
        RamIsNotSelected
    {
        if (tokenIdOfChallenger > ramNFT.tokenCounter()) {
            revert ChoosingRam__InvalidTokenIdOfChallenger();
        }
        if (tokenIdOfAnyPerticipent > ramNFT.tokenCounter()) {
            revert ChoosingRam__InvalidTokenIdOfPerticipent();
        }
        if (ramNFT.getCharacteristics(tokenIdOfChallenger).ram != msg.sender) {
            revert ChoosingRam__CallerIsNotChallenger();
        }


        if (block.timestamp > 1728691200) {
            revert ChoosingRam__TimeToBeLikeRamFinish();
        }

        uint256 random =
            uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, msg.sender))) % 2;


        if (random == 0) {
-->            if (ramNFT.getCharacteristics(tokenIdOfChallenger).isJitaKrodhah == false){
-->                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, false, false, false, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isDhyutimaan == false){
-->                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, false, false, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isVidvaan == false){
-->                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, false, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isAatmavan == false){
-->                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isSatyavaakyah == false){
-->                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, true);
-->                selectedRam = ramNFT.getCharacteristics(tokenIdOfChallenger).ram;
            }
        } else {
-->            if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isJitaKrodhah == false){
-->                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, false, false, false, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isDhyutimaan == false){
-->                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, false, false, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isVidvaan == false){
-->                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, false, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isAatmavan == false){
-->                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, true, false);
-->            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isSatyavaakyah == false){
-->                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, true, true);
-->                selectedRam = ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).ram;
            }
        }
    }
```
