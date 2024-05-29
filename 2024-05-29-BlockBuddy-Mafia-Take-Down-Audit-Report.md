

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
- [High Issues](#high-issues)
  - [H-1: Arbitrary `from` passed to `transferFrom` (or `safeTransferFrom`)](#h-1-arbitrary-from-passed-to-transferfrom-or-safetransferfrom)
  - [H-2 `Laundrette::configureDependencies` the weapon dependency index should be marked 1 intead of 0](#h-2-laundretteconfiguredependencies-the-weapon-dependency-index-should-be-marked-1-intead-of-0)
  - [H-3 `Laundrette::retrieveAdmin` function is missing proper natspec about the functioning and it is not clear what the function is intended to do.](#h-3-laundretteretrieveadmin-function-is-missing-proper-natspec-about-the-functioning-and-it-is-not-clear-what-the-function-is-intended-to-do)
  - [H-4 `MoneyVault::withdrawUSDC` function has no check for the return value of the transfer function being used](#h-4-moneyvaultwithdrawusdc-function-has-no-check-for-the-return-value-of-the-transfer-function-being-used)
  - [H-5 `MoneyShelf::withdrawUSDC` function has no check for the return value of the transfer function being used](#h-5-moneyshelfwithdrawusdc-function-has-no-check-for-the-return-value-of-the-transfer-function-being-used)
- [Medium](#medium)
  - [M-1 `Laundrette::quitGang` member can just quit the gang , without approval of godfather or gang members](#m-1-laundrettequitgang-member-can-just-quit-the-gang--without-approval-of-godfather-or-gang-members)
- [Low Issues](#low-issues)
  - [L-1: Centralization Risk for trusted owners](#l-1-centralization-risk-for-trusted-owners)
  - [L-2: Unsafe ERC20 Operations should not be used](#l-2-unsafe-erc20-operations-should-not-be-used)
  - [L-3: javascript pragma should be specific, not wide](#l-3-javascript-pragma-should-be-specific-not-wide)
  - [L-4: Missing checks for `address(0)` when assigning values to address state variables](#l-4-missing-checks-for-address0-when-assigning-values-to-address-state-variables)
  - [L-5: `public` functions not used internally could be marked `external`](#l-5-public-functions-not-used-internally-could-be-marked-external)
  - [L-6: Modifiers invoked only once can be shoe-horned into the function](#l-6-modifiers-invoked-only-once-can-be-shoe-horned-into-the-function)
  - [L-7 In `CrimeMoney::kernel` \& `MoneyVault::usdc and MoneyVault::crimeMoney`  \& `MoneyShelf::usdc & MoneyShelf::crimeMoney` should be marked immutable , as it is defined only once in the contract](#l-7-in-crimemoneykernel--moneyvaultusdc-and-moneyvaultcrimemoney---moneyshelfusdc--moneyshelfcrimemoney-should-be-marked-immutable--as-it-is-defined-only-once-in-the-contract)
    - [CrimeMoney.sol](#crimemoneysol)
    - [MoneyVault,sol](#moneyvaultsol)
    - [MoneyShelf.sol](#moneyshelfsol)
  - [L-8 `MoneyVault::depositUSDC` there is no any method of depositing funds in this contract , but there is deposit function which does nothing but just reverts](#l-8-moneyvaultdepositusdc-there-is-no-any-method-of-depositing-funds-in-this-contract--but-there-is-deposit-function-which-does-nothing-but-just-reverts)

# Protocol Summary

The protocol is of a working of a mafia gang and how they function. There are different contracts for different purposes such as guns and money. There is a GodFather among the gang who has control obver the majority of the decisions in the protocol and he assigns task and works to the gang members. Members can join the gang and are assigned work and roles by the godfather.

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
| High      | 5                      |
| Medium    | 1                      |
| Low       | 8                      |
| Info      | 0                      |
| Total     | 14                     |


# Findings :


# High Issues

## H-1: Arbitrary `from` passed to `transferFrom` (or `safeTransferFrom`)

Passing an arbitrary `from` address to `transferFrom` (or `safeTransferFrom`) can lead to loss of funds, because anyone can transfer tokens from the `from` address if an approval is made.  

- Found in src/modules/MoneyShelf.sol [Line: 31](src/modules/MoneyShelf.sol#L31)

	```javascript
	        usdc.transferFrom(account, address(this), amount);
	```




## H-2 `Laundrette::configureDependencies` the weapon dependency index should be marked 1 intead of 0

**Description** 
```
function configureDependencies() external override onlyKernel returns (Keycode[] memory dependencies) {
        dependencies = new Keycode[](2);

        dependencies[0] = toKeycode("MONEY");
        moneyShelf = MoneyShelf(getModuleAddress(toKeycode("MONEY")));
        
-->>       dependencies[0] = toKeycode("WEAPN");
        weaponShelf = WeaponShelf(getModuleAddress(toKeycode("WEAPN")));
    }
```
**Impact** The weapon and money have the same dependency so they will not be segregated, thus the gang have no seperation for weapons and money.

**Recommended Mitigation** Replace these lines
```diff
function configureDependencies() external override onlyKernel returns (Keycode[] memory dependencies) {
        dependencies = new Keycode[](2);

        dependencies[0] = toKeycode("MONEY");
        moneyShelf = MoneyShelf(getModuleAddress(toKeycode("MONEY")));
        
+        dependencies[1] = toKeyCode("WEAPN");
-        dependencies[0] = toKeycode("WEAPN");
        weaponShelf = WeaponShelf(getModuleAddress(toKeycode("WEAPN")));
    }
```

## H-3 `Laundrette::retrieveAdmin` function is missing proper natspec about the functioning and it is not clear what the function is intended to do.

**Description** The `Laundrette::retrieveAdmin`  , as the name suggests it should only retrieve who the Admin is but in its description it is changing the admin.

```javascript
function retrieveAdmin() external {
        kernel.executeAction(Actions.ChangeAdmin, kernel.executor());
    }
```
**Impact** As the name of the function and functioning suggest different things , wrong calling of the function maybe done which may cause the different action to be made instead of the desired action.

**Recommended Mitigation** 
1. A proper natspec should be provided about the function , what is it doing and what the different components are for.

2. Name the function appropriately.

3. The function should be made to only be called by the GodFather for now to ensure security and avoid any mishappening.


## H-4 `MoneyVault::withdrawUSDC` function has no check for the return value of the transfer function being used

**Description** 
```javascript
 function withdrawUSDC(address account, address to, uint256 amount) external {
        require(to == kernel.executor(), "MoneyVault: only GodFather can receive USDC");
        withdraw(account, amount);
        crimeMoney.burn(account, amount);
-->>        usdc.transfer(to, amount);  //no check for return value 
    }
```
**Impact** The user can call the withdrawUDSC infinite number of times and may cause harm to the protocol by using some exploits as the return value of the transfer function is not checked.

**Recommended Mitigation** Add following lines in the code 
```diff
function withdrawUSDC(address account, address to, uint256 amount) external {
        require(to == kernel.executor(), "MoneyVault: only GodFather can receive USDC");
        withdraw(account, amount);
        crimeMoney.burn(account, amount);
+		require(usdc.transfer(to,amount) != false , "Transfer Failed");
_       usdc.transfer(to, amount);   
    }
```

## H-5 `MoneyShelf::withdrawUSDC` function has no check for the return value of the transfer function being used

**Description** 
```javascript
  function withdrawUSDC(address account, address to, uint256 amount) external {
        withdraw(account, amount);
        crimeMoney.burn(account, amount);
-->>        usdc.transfer(to, amount);
    }
```
**Impact** The user can call the withdrawUDSC infinite number of times and may cause harm to the protocol by using some exploits as the return value of the transfer function is not checked.

**Recommended Mitigation** Add following lines in the code 
```diff
function withdrawUSDC(address account, address to, uint256 amount) external {
        withdraw(account, amount);
        crimeMoney.burn(account, amount);
+        require(usdc.transfer(to,amount) != false , "Transfer failed!!");
-        usdc.transfer(to, amount);
    }
```


# Medium

## M-1 `Laundrette::quitGang` member can just quit the gang , without approval of godfather or gang members

**Description** A gang member doesn't needs anyone's permission or approval to leave the gang.

```javascript
function quitTheGang(address account) external onlyRole("gangmember") {
        kernel.revokeRole(Role.wrap("gangmember"), account);
    }
```

**Impact** The gang member can do some mischief and just leave the gang without anyone knowing on his own will , which will cause a lot of problem for the gang.

**Recommended Mitigation** The godFather must approve the request for leaving the gang of any member , after then only he will be able to leave the gang.




# Low Issues

## L-1: Centralization Risk for trusted owners

Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

- Found in src/policies/Laundrette.sol [Line: 74](src/policies/Laundrette.sol#L74)

	```javascript
	        onlyRole("gangmember")
	```

- Found in src/policies/Laundrette.sol [Line: 80](src/policies/Laundrette.sol#L80)

	```javascript
	    function takeGuns(address account, uint256 amount) external onlyRole("gangmember") isAuthorizedOrRevert(account) {
	```

- Found in src/policies/Laundrette.sol [Line: 84](src/policies/Laundrette.sol#L84)

	```javascript
	    function addToTheGang(address account) external onlyRole("gangmember") isGodFather {
	```

- Found in src/policies/Laundrette.sol [Line: 88](src/policies/Laundrette.sol#L88)

	```javascript
	    function quitTheGang(address account) external onlyRole("gangmember") {
	```



## L-2: Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

- Found in src/modules/MoneyShelf.sol [Line: 31](src/modules/MoneyShelf.sol#L31)

	```javascript
	        usdc.transferFrom(account, address(this), amount);
	```

- Found in src/modules/MoneyShelf.sol [Line: 39](src/modules/MoneyShelf.sol#L39)

	```javascript
	        usdc.transfer(to, amount);
	```

- Found in src/modules/MoneyVault.sol [Line: 36](src/modules/MoneyVault.sol#L36)

	```javascript
	        usdc.transfer(to, amount);
	```



## L-3: javascript pragma should be specific, not wide

Consider using a specific version of javascript in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.0;`, use `pragma solidity 0.8.0;`

- Found in src/CrimeMoney.sol [Line: 2](src/CrimeMoney.sol#L2)

	```javascript
	pragma javascript ^0.8.24;
	```

- Found in src/Kernel.sol [Line: 2](src/Kernel.sol#L2)

	```javascript
	pragma javascript ^0.8.15;
	```

- Found in src/utils/KernelUtils.sol [Line: 2](src/utils/KernelUtils.sol#L2)

	```javascript
	pragma javascript ^0.8.15;
	```



## L-4: Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

- Found in src/CrimeMoney.sol [Line: 12](src/CrimeMoney.sol#L12)

	```javascript
	        kernel = _kernel;
	```

- Found in src/Kernel.sol [Line: 65](src/Kernel.sol#L65)

	```javascript
	        kernel = kernel_;
	```

- Found in src/Kernel.sol [Line: 74](src/Kernel.sol#L74)

	```javascript
	        kernel = newKernel_;
	```

- Found in src/Kernel.sol [Line: 209](src/Kernel.sol#L209)

	```javascript
	            executor = target_;
	```

- Found in src/Kernel.sol [Line: 211](src/Kernel.sol#L211)

	```javascript
	            admin = target_;
	```

- Found in src/Kernel.sol [Line: 226](src/Kernel.sol#L226)

	```javascript
	        getModuleForKeycode[keycode] = newModule_;
	```

- Found in src/Kernel.sol [Line: 347](src/Kernel.sol#L347)

	```javascript
	            modulePermissions[request.keycode][policy_][request.funcSelector] = grant_;
	```

- Found in src/modules/MoneyShelf.sol [Line: 18](src/modules/MoneyShelf.sol#L18)

	```javascript
	        usdc = _usdc;
	```

- Found in src/modules/MoneyShelf.sol [Line: 19](src/modules/MoneyShelf.sol#L19)

	```javascript
	        crimeMoney = _crimeMoney;
	```

- Found in src/modules/MoneyVault.sol [Line: 20](src/modules/MoneyVault.sol#L20)

	```javascript
	        usdc = _usdc;
	```

- Found in src/modules/MoneyVault.sol [Line: 21](src/modules/MoneyVault.sol#L21)

	```javascript
	        crimeMoney = _crimeMoney;
	```

- Found in src/modules/Shelf.sol [Line: 13](src/modules/Shelf.sol#L13)

	```javascript
	        bank[account] += amount;
	```

- Found in src/modules/Shelf.sol [Line: 17](src/modules/Shelf.sol#L17)

	```javascript
	        bank[account] -= amount;
	```



## L-5: `public` functions not used internally could be marked `external`

Instead of marking a function as `public`, consider marking it as `external` if it is not used internally.

- Found in src/CrimeMoney.sol [Line: 20](src/CrimeMoney.sol#L20)

	```javascript
	    function mint(address to, uint256 amount) public onlyMoneyShelf {
	```

- Found in src/CrimeMoney.sol [Line: 24](src/CrimeMoney.sol#L24)

	```javascript
	    function burn(address from, uint256 amount) public onlyMoneyShelf {
	```

- Found in src/Kernel.sol [Line: 382](src/Kernel.sol#L382)

	```javascript
	    function grantRole(Role role_, address addr_) public onlyAdmin {
	```

- Found in src/Kernel.sol [Line: 393](src/Kernel.sol#L393)

	```javascript
	    function revokeRole(Role role_, address addr_) public onlyAdmin {
	```

- Found in src/modules/MoneyShelf.sol [Line: 22](src/modules/MoneyShelf.sol#L22)

	```javascript
	    function KEYCODE() public pure override returns (Keycode) {
	```

- Found in src/modules/MoneyVault.sol [Line: 24](src/modules/MoneyVault.sol#L24)

	```javascript
	    function KEYCODE() public pure override returns (Keycode) {
	```

- Found in src/modules/WeaponShelf.sol [Line: 10](src/modules/WeaponShelf.sol#L10)

	```javascript
	    function KEYCODE() public pure override returns (Keycode) {
	```

## L-6: Modifiers invoked only once can be shoe-horned into the function



- Found in src/Kernel.sol [Line: 177](src/Kernel.sol#L177)

	```javascript
	    modifier onlyExecutor() {
	```

## L-7 In `CrimeMoney::kernel` & `MoneyVault::usdc and MoneyVault::crimeMoney`  & `MoneyShelf::usdc & MoneyShelf::crimeMoney` should be marked immutable , as it is defined only once in the contract


### CrimeMoney.sol
```javascript
contract CrimeMoney is ERC20 {
   
-->>    Kernel public kernel;

    constructor(Kernel _kernel) ERC20("CrimeMoney", "CRIME") {
-->>        kernel = _kernel;
    }
}
```
### MoneyVault,sol
```javascript
contract MoneyVault is Shelf {

    
-->>    IERC20 private usdc;
-->>    CrimeMoney private crimeMoney;

    constructor(Kernel kernel_, IERC20 _usdc, CrimeMoney _crimeMoney) Shelf(kernel_) {
        usdc = _usdc;
        crimeMoney = _crimeMoney;
    }
}
```

### MoneyShelf.sol
```javascript
contract MoneyShelf is Shelf {

    IERC20 private usdc;
    CrimeMoney private crimeMoney;

    constructor(Kernel kernel_, IERC20 _usdc, CrimeMoney _crimeMoney) Shelf(kernel_) {
        
-->>        usdc = _usdc;
-->>        crimeMoney = _crimeMoney;
    }
}
```

## L-8 `MoneyVault::depositUSDC` there is no any method of depositing funds in this contract , but there is deposit function which does nothing but just reverts

**Description** 

```javascript
 function depositUSDC(address, address, uint256) external pure {
        revert("MoneyVault: depositUSDC is disabled");
    }
```
**Impact** Waste of Gas

**Recommended Mitigation** We should get rid of the deposit function as mentioned in the doc that MoneyVault is used in an emergency situation and no funds can be deposited in it.
 

