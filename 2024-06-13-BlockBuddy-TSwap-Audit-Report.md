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
    - [\[H-1\] `deadline` in `TSWapPool::deposit` is not used \& can be made either 0 or infinite to break the functioning of the protocol](#h-1-deadline-in-tswappooldeposit-is-not-used--can-be-made-either-0-or-infinite-to-break-the-functioning-of-the-protocol)
    - [\[H-2\] There is a `null` function in `TSwapPool` which has no name \& also is specified a return but doesn't return anything](#h-2-there-is-a-null-function-in-tswappool-which-has-no-name--also-is-specified-a-return-but-doesnt-return-anything)
  - [MEDIUM](#medium)
    - [\[M-1\] Don't use strict equalities in `TSwapPool::revertIfZero` modifier](#m-1-dont-use-strict-equalities-in-tswappoolrevertifzero-modifier)
  - [LOWS](#lows)
    - [\[L-1\] In `Poolfactory::error PoolFactory__PoolDoesNotExist(address tokenAddress)` is not used anywhere](#l-1-in-poolfactoryerror-poolfactory__pooldoesnotexistaddress-tokenaddress-is-not-used-anywhere)
    - [\[L-2\] `block.timestamp` can be manipulated by external means to break it](#l-2-blocktimestamp-can-be-manipulated-by-external-means-to-break-it)
    - [\[L-3\] In `TSwapPool::_swap` function there is a use of literal \& no event return after token transfer](#l-3-in-tswappool_swap-function-there-is-a-use-of-literal--no-event-return-after-token-transfer)

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

### [H-1] `deadline` in `TSWapPool::deposit` is not used & can be made either 0 or infinite to break the functioning of the protocol

**Description** The `deadline` input which is meant to tell the time till the functioning has to be completed is not used in `TSWapPool::deposit`

**Impact** Someone can make the deadline `0` or infinite according to their need and will break the protocol.

**Proof of Code**

<details>
<summary> PoC </summary>

```javascript
function testDeposit() public {
        vm.startPrank(liquidityProvider);
        weth.approve(address(pool), 100e18);
        poolToken.approve(address(pool), 100e18);
-->>        pool.deposit(100e18, 100e18, 100e18, 0);

        assertEq(pool.balanceOf(liquidityProvider), 100e18);
        assertEq(weth.balanceOf(liquidityProvider), 100e18);
        assertEq(poolToken.balanceOf(liquidityProvider), 100e18);

        assertEq(weth.balanceOf(address(pool)), 100e18);
        assertEq(poolToken.balanceOf(address(pool)), 100e18);
    }
```

</details>

**Recommended Mitigation**

1. Consider removing the `deadline` inout from the `TSWapPool::deposit` function
2. If still you are using it have some checks for deadline passing.

### [H-2] There is a `null` function in `TSwapPool` which has no name & also is specified a return but doesn't return anything

**Description**

```javascript
-->> function  (
        IERC20 inputToken,
        uint256 inputAmount,
        IERC20 outputToken,
        uint256 minOutputAmount,
        uint64 deadline
    )
        public
        revertIfZero(inputAmount)
        revertIfDeadlinePassed(deadline)
 -->>       returns (uint256 output)
    {
        uint256 inputReserves = inputToken.balanceOf(address(this));
        uint256 outputReserves = outputToken.balanceOf(address(this));

        uint256 outputAmount = getOutputAmountBasedOnInput(
            inputAmount,
            inputReserves,
            outputReserves
        );

        if (outputAmount < minOutputAmount) {
            revert TSwapPool__OutputTooLow(outputAmount, minOutputAmount);
        }

        _swap(inputToken, inputAmount, outputToken, outputAmount);
        // no returns..!!
    }
```

**Impact**

1. The function has no name , so how to call it and distinguish it & what does it do..?
2. Also a return is specified and it doesn't return anything , so maybe a waste of gas and computatiion

**Recommended Mitigation**

1. Name the function properly
2. If it doen't return anything consider removing the return statement.

## MEDIUM

### [M-1] Don't use strict equalities in `TSwapPool::revertIfZero` modifier

**Description**

```javascript
modifier revertIfZero(uint256 amount) {
-->>        if (amount == 0) {
            revert TSwapPool__MustBeMoreThanZero();
        }
        _;
    }
```

**Recommended Mitigation** USe `<=` equality instead

```javascript
modifier revertIfZero(uint256 amount) {
-->>        if (amount <= 0) {
            revert TSwapPool__MustBeMoreThanZero();
        }
        _;
    }
```

## LOWS

### [L-1] In `Poolfactory::error PoolFactory__PoolDoesNotExist(address tokenAddress)` is not used anywhere

**Description** It is declared but not used in the contract

```javascript
contract PoolFactory {
    error PoolFactory__PoolAlreadyExists(address tokenAddress);
-->>    error PoolFactory__PoolDoesNotExist(address tokenAddress);
```

**Impact** Waste of gas and one extra line of code

**Recommended Mitigation**

1. Consider removing it

### [L-2] `block.timestamp` can be manipulated by external means to break it

**Description**

```javascript
modifier revertIfDeadlinePassed(uint64 deadline) {
-->>        if (deadline < uint64(block.timestamp)) {
            revert TSwapPool__DeadlineHasPassed(deadline);
        }
        _;
    }
```

**Recommended Mitigation**

1. Consider also using `block.number` to be more secure.

### [L-3] In `TSwapPool::_swap` function there is a use of literal & no event return after token transfer

**Description**

```javascript
function _swap(
        IERC20 inputToken,
        uint256 inputAmount,
        IERC20 outputToken,
        uint256 outputAmount
    ) private {
        if (
            _isUnknown(inputToken) ||
            _isUnknown(outputToken) ||
            inputToken == outputToken
        ) {
            revert TSwapPool__InvalidToken();
        }

        swap_count++;
        if (swap_count >= SWAP_COUNT_MAX) {
            swap_count = 0;
-->>            outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000); // don't use literals
        }
        emit Swap(
            msg.sender,
            inputToken,
            inputAmount,
            outputToken,
            outputAmount
        );
        inputToken.safeTransferFrom(msg.sender, address(this), inputAmount);
        outputToken.safeTransfer(msg.sender, outputAmount);
-->>        // should emit a event that tokens are transferred
    }
```

**Recommended Mitigation**

1. Use constants in place of literals
2. Emit an event ater tokens are transferred
