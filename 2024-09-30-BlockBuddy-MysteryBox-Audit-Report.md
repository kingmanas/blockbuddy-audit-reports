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
	- [HIGH](#high)
		- [\[H-1\] Reentrancy in "MysteryBox::claimAllRewards" \& "MysterBox::claimSingleReward" as it does not follow CEI](#h-1-reentrancy-in-mysteryboxclaimallrewards--mysterboxclaimsinglereward-as-it-does-not-follow-cei)
	- [\[H-2\] Anyone can chnage the owner in the "MysteryBox::chnageOwner" function , not the owner](#h-2-anyone-can-chnage-the-owner-in-the-mysteryboxchnageowner-function--not-the-owner)
	- [LOW](#low)
		- [\[L-1\] In the "TestMysterBox::setUp" function there is no setUp to send the ether to the MysteryBox contract thats why the test will not run](#l-1-in-the-testmysterboxsetup-function-there-is-no-setup-to-send-the-ether-to-the-mysterybox-contract-thats-why-the-test-will-not-run)

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
| High      | 2                      |
| Medium    | 1                      |
| Low       | 3                      |
| Info      | 0                      |
| Total     | 6                      |

# Findings :

## HIGH

### [H-1] Reentrancy in "MysteryBox::claimAllRewards" & "MysterBox::claimSingleReward" as it does not follow CEI

**Description** The external call should be done at the last , rather than before the effect.

```javascript

 function claimAllRewards() public { 
        uint256 totalValue = 0;
        for (uint256 i = 0; i < rewardsOwned[msg.sender].length; i++) {
            totalValue += rewardsOwned[msg.sender][i].value;
        }
        require(totalValue > 0, "No rewards to claim");

        (bool success,) = payable(msg.sender).call{value: totalValue}("");
        require(success, "Transfer failed");

        delete rewardsOwned[msg.sender]; // ==>> Effect after Interaction
    }

    function claimSingleReward(uint256 _index) public { 
        require(_index <= rewardsOwned[msg.sender].length, "Invalid index");
        uint256 value = rewardsOwned[msg.sender][_index].value;
        require(value > 0, "No reward to claim");

        (bool success,) = payable(msg.sender).call{value: value}("");
        require(success, "Transfer failed");

        delete rewardsOwned[msg.sender][_index]; // ==>> Effect after Interaction
    }
```
**Impact** It can lead to user entering the function again and again draining the funds and claiming everything.

**Recommended Mitigation** Do the following :

```diff
function claimAllRewards() public { 
        uint256 totalValue = 0;
        for (uint256 i = 0; i < rewardsOwned[msg.sender].length; i++) {
            totalValue += rewardsOwned[msg.sender][i].value;
        }
        require(totalValue > 0, "No rewards to claim");

-        (bool success,) = payable(msg.sender).call{value: totalValue}("");
-        require(success, "Transfer failed");

-        delete rewardsOwned[msg.sender]; 

+		 delete rewardsOwned[msg.sender]; 

+	     (bool success,) = payable(msg.sender).call{value: totalValue}("");
+        require(success, "Transfer failed");
    }

    function claimSingleReward(uint256 _index) public { 
        require(_index <= rewardsOwned[msg.sender].length, "Invalid index");
        uint256 value = rewardsOwned[msg.sender][_index].value;
        require(value > 0, "No reward to claim");

-        (bool success,) = payable(msg.sender).call{value: value}("");
-        require(success, "Transfer failed");

-        delete rewardsOwned[msg.sender][_index]; 

+		 delete rewardsOwned[msg.sender][_index]; 

+		 (bool success,) = payable(msg.sender).call{value: value}("");
+        require(success, "Transfer failed");

	}
```

## [H-2] Anyone can chnage the owner in the "MysteryBox::chnageOwner" function , not the owner

**Description** No check for who can change the owner
```javascript
function changeOwner(address _newOwner) public { //=> No check for who can change the owner
        owner = _newOwner;
    }
```

**Impact** Anyone can change the owner and misuse it for their benefit

**Proof of Code** The following test confirms it

<details>
<summary>PoC</summary>

```javascript
 function testChangeOwner() public {
        mysteryBox.changeOwner(user1);
        assertEq(mysteryBox.owner(), user1);
    }
```
</details>

**Recommended Mitigation** Add some check for who can call the function , one possible approach is mentioned below
```diff
function changeOwner(address _newOwner) public { 
+       require(msg.sender == owner);
        owner = _newOwner;
    }

```

## LOW

### [L-1] In the "TestMysterBox::setUp" function there is no setUp to send the ether to the MysteryBox contract thats why the test will not run

**Description** The MysterBox contract should be initialized with a minimum value of 0.1 ether or above 

```javascript

function setUp() public {
        owner = makeAddr("owner");
        user1 = address(0x1);
        user2 = address(0x2);

        vm.prank(owner);                    // => no funds in the owners address
        
        mysteryBox = new MysteryBox();      // => contract initialized without any ether
        console.log("Reward Pool Length:", mysteryBox.getRewardPool().length);
    }

```

**Impact** The MysteryBox contract will not initialize and the test will not run

We are encountering the following error:

→ new <unknown>@0x88F59F8826af5e695B13cA934d6c7999875A9EeA
    │   └─ ← [Revert] 100 bytes of code
    └─ ← [Revert] revert: Incorrect ETH sent

**Recommended Mitigation**

```diff
 owner = makeAddr("owner");
        user1 = address(0x1);
        user2 = address(0x2);

        vm.prank(owner);                    
+       vm.deal(owner , 5 ether); 

-        mysteryBox = new MysteryBox();      
+		 mysteryBox = new MysteryBox{value: 0.5 ether}();
        console.log("Reward Pool Length:", mysteryBox.getRewardPool().length);

```
