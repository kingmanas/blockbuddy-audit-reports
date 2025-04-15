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
| Medium    | 1                      |
| Low       | 1                      |
| Info      | 0                      |
| Total     | 6                      |

# Findings :

### [L-1]  We are using transient nonRentrant which can be risky

**Description** The `InheritanceManager::nonReentrant` modifier is using transient storage which can be risky

```javascript
modifier nonReentrant() {
        assembly {
            if tload(1) { revert(0, 0) }
-->            tstore(0, 1)
        }
        _;
        assembly {
-->            tstore(0, 0)
        }
    }
```

**Impact** It can be risky(although gas effeceint) but as transient storage is cleared only after the end of transaction, not at the end of the call!!!

**Recommended mitigation** Use Reentrancy Guard inatead of transient storage.

### [H-1] No Zero Address & Self Address check in `InheritanceManager::sendEth()` function 

**Description** In `InheritanceManager::sendEth()` there are no checks for zero address and self address.

```javascript
function sendETH(uint256 _amount, address _to) external nonReentrant onlyOwner {
       // --->> NO CHECKS FOR ADDRESS(0) AND SELF ADDRESS 
        (bool success,) = _to.call{value: _amount}("");
        require(success, "Transfer Failed");
        _setDeadline();
    }
```

**Impact** 1. For `address(0)` the funds would be lost forever(which are sent to it)
2. For self address there will wastage of gas money

**Proof of Code** In `InheritanceManagertest.t.sol` add the below test functions

```javascript
function test_sendEtherFromOwnertoZeroAddress() public {
        vm.deal(address(im), 10e18);
        vm.startPrank(owner);
        im.sendETH(1e18, address(0));
        assertEq(address(im).balance, 10e18);
        vm.stopPrank();
    }

function test_sendEtherFromOwnertoSelfAddress() public {
        vm.deal(address(im), 10e18);
        vm.startPrank(owner);
        im.sendETH(1e18, address(im));
        assertEq(address(im).balance, 10e18);
        vm.stopPrank();
    }
```

**Recommended Mitigation** Add the following in your `InheritanceManager::sendEth()` function

```diff
function sendETH(uint256 _amount, address _to) external nonReentrant onlyOwner {
+       require(_to != address(0), "Cannot send to zero address");
+       require(_to != address(this), "Cannot send to self"); 
        (bool success,) = _to.call{value: _amount}("");
        require(success, "Transfer Failed");
        _setDeadline();
    }
```

### [M-1] No Zero Address Check in `InheritanceManager::contractInteractions()` Function

**Description:**  
The `InheritanceManager::contractInteractions` function allows the `owner` to make arbitrary low-level calls to **any `_target` address**, including the zero address (`address(0)`)

```javascript
function contractInteractions(address _target, bytes calldata _payload, uint256 _value, bool _storeTarget)
    external
    nonReentrant
    onlyOwner
{
        /// --->> NO CHECKS FOR ZERO TARGET ADDRESS
    (bool success, bytes memory data) = _target.call{value: _value}(_payload);
    require(success, "interaction failed");
    if (_storeTarget) {
        interactions[_target] = data;
    }
}
```
**Impact** 1. Sending ETH to address(0) will result in permanent loss of funds.

2. Low-level calls to the zero address are guaranteed to fail, wasting gas.

**Proof of Code** Add this function in `InheritanceManageTest.t.sol` 

```javascript

function testContractInteractionPassesOnZeroAddress() public {
        vm.deal(address(im), 10e18);
        vm.prank(owner);
        address target = address(0); 
        bytes memory payload = abi.encodeWithSignature("nonExistentFunction()");
        uint256 value = 5e18;
        bool storeTarget = true;
        im.contractInteractions(target, payload, value, storeTarget);
        console.log(address(im).balance);
        console.log(address(target).balance);
    }
```

**Recommended Mitigation** Add the following zero address check in `InheritranceManager::contractInteractions()`

```diff

function contractInteractions(address _target, bytes calldata _payload, uint256 _value, bool _storeTarget)
        external
        nonReentrant
        onlyOwner
    {
+       require(_target != address(0) , "Revert Zero Address");
        (bool success, bytes memory data) = _target.call{value: _value}(_payload);
        require(success, "interaction failed");
        if (_storeTarget) {
            interactions[_target] = data;
        }
    }
```

### [H-2] InheritanceManager Cannot Receive ETH Directly — No `receive()` or `fallback()` Function Defined

**Description:**  
The `InheritanceManager` contract **does not implement a `receive()` or `fallback()` function**, meaning it **cannot directly receive ETH via `transfer()`, `send()`, or `selfdestruct`** calls. This is a critical issue for a contract meant to **hold and manage ETH balances for inheritance purposes**.

**Impact:**

1. **Loss of Funds:** ETH sent to the contract using `transfer()` or `send()` will be **reverted**, preventing funding.
2. **Blocked UX/Functionality:** Common interfaces like Gnosis Safe, UI wallets, or protocols sending ETH may fail.
3. **Breaks Inheritance Logic:** If ETH cannot be received, the core purpose of managing inheritance is undermined.

**Recommended Mitigation:**

No `receive()` or `fallback()` function is present in the contract:

```solidity
// Missing this:
receive() external payable {}

fallback() external payable {}
```

### [H-3] Missing Fund Distribution to Caller in `buyOutEstateNFT()` Function

**Description:**  
In the `InheritanceManager::buyOutEstateNFT()` function, if the caller (`msg.sender`) is one of the beneficiaries, the function **returns early without executing the rest of the code**, particularly the **fund distribution logic**.

```solidity
for (uint256 i = 0; i < beneficiaries.length; i++) {
    if (msg.sender == beneficiaries[i]) {
        return; // --->> RETURNS EARLY, SKIPPING FUND TRANSFERS
    } else {
        IERC20(assetToPay).safeTransfer(beneficiaries[i], finalAmount / divisor);
    }
    nft.burnEstate(_nftID);
}
```

**Impact:**

1. Loss of User Funds: The caller transfers tokens, but no NFT is burned or distributed.

2. Logic Violation: The entire purpose of buying out the NFT and redistributing the funds is broken.

**Recommended Mitigation:**

```diff
for (uint256 i = 0; i < beneficiaries.length; i++) {
-    if (msg.sender == beneficiaries[i]) {
+    if (msg.sender != benficiaries[i]){   
         return; 
    } else {
        IERC20(assetToPay).safeTransfer(beneficiaries[i], finalAmount / divisor);
    }
    nft.burnEstate(_nftID);
}
}
```

### [H-4] Loss of funds if `owner` removes a beneficiary

**Description:**

If the `owner` removes a `beneficiary` from the `InheritanceManager::beneficiaries[]` array by calling `InheritanceManager::removeBeneficiary()`, a susequent call to `InheritanceManager::inherit()` and `InheritanceManager::withdrawInheritedFunds()` after the 90 day lock, results in a share of the funds being burned.

**Recommended Mitigation:**
Instead of `delete()` keyword use `beneficiaries.pop()`








 
