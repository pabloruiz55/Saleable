# Saleable
A protocol for Saleable Smart Contracts

## Simple Summary
The Saleable Contract protocol allows contracts to be sold in exchange of ether.

## Abstract
The following discusses a protocol for smart contracts that can be sold by their owners. 
User A offers User B the ownership of his contract. User B accepts the offer by sending the arranged sum of ether. By sending the specified sum of ether, User B agrees to buy the contract and becomes the new owner. 

## Motivation
Given that there is no concept of "whoever deploys a contract owns it" in Ethereum, but it's very common for contracts to be assigned an owner upon initialization, there should be mechanisms in place to allow ownership to change hands. This ERC proposes the implementation of a secure transfer of ownership made in exchange of ether.
Such protocol could allow, among other things, for a "smart contract marketplace" or be used by contractors so they can be certain that the code they deploy on behalf of someone else will be paid for when they relinquish ownership.

## Specification
## Saleable 
**NOTE:** The contract inheriting the Saleable base contract must enforce the usage of the onlyOwner modifier. Owning a contract means having `owner` set to your account address. 

### Methods

#### owner
Returns the current owner of the contract. The owner is the EOA or other contract that can execute key functions of the contract.

``` js
function owner() view returns (address owner);
```

#### selling
Returns whether or not the contract is being sold. A contract currently being sold might block certain functions until the transaction is completed or canceled by applying the `ifNotLocked` modifier.

``` js
function selling() view returns (bool selling);
```

#### sellingTo
Returns the address set by the seller, if they specified a particular target buyer when putting the contract for sale.

OPTIONAL - When putting the contract for sale, this variable can be set to `0x0`. In this case, anyone might be able to buy the contract in a first-come, first-serve basis.

``` js
function sellingTo() view returns (address sellingTo);
```

#### askingPrice
Returns the price at which the contract has been offered for sale.

``` js
function askingPrice() view returns (uint askingPrice);
```

#### initiateSale
Puts the contract for sale. Only allowed to the owner of the contract, they set an asking price for the contract and (optionally) set a target buyer.
It MUST throw if the contract is already being sold. 
It MUST throw if the target buyer is itself or the owner.

NOTE: The asking price can be 0, in the case the owner wants to give the contract away for free or "donate" it.

NOTE: `address _to` is optional. It can be `0x0`, in which case, any other account will be able to buy it. 

``` js
function initiateSale(uint _price, address _to) onlyOwner public
```

#### cancelSale
Cancels the sale of the contract. Resets all the sale parameters. Only available to the owner of the contract.

``` js
function cancelSale() onlyOwner public
```

#### completeSale
Completes the sale by having a buyer transfer the ether being asked by the seller and changing ownership of the contract. MUST fire `Transfer` event.
Called by the `sellingTo` account if specified or anyone other than the contract itself/owner if `sellingTo` was not specified.
Value sent MUST match the askingPrice.

``` js
function completeSale() public payable
```

### Events

#### Transfer
MUST be fired when a sale is completed. Keeps a history of who the previous and new owners are, as well as the sale date and sale price. It also MUST be fired when the contract is created (with _from being 0x0 and salePrice being 0).

``` js
event Transfer(uint _saleDate, address _from, address _to, uint _salePrice);
```

### Modifiers

#### onlyOwner
MUST be implemented in functions that should only be executed by the owner of the contract. 

``` js
modifier onlyOwner
```

#### ifNotLocked
MUST be implemented in functions that should only be executed while the contract is not for sale.
Implementing this modifier would give the buyers the certainty that key contract values are not being modified while they analyze the purchase. 

``` js
modifier ifNotLocked
```

## Rationale
The rationale behind this is allowing owners of a particular contract to sell them with the certainty they will receive the asking price at the same time the contract changes hands.
This protocol could also allow for a contract marketplace to be created, where developers can deploy specific contracts and put them for sale. 

- Allowing donations: The contract can be sold for 0 ether if the owner wishes so. This can be useful if the owner wants to gift/donate/give away the contract to someone in particular or whoever gets it first.

- Allowing contracts with a balance to be put for sale: I'm putting this up for discussion. Right now whether or not the contract has some balance in it is not important, but maybe initiateSale() SHOULD throw if it has some balance to prevent selling a contract that held money inadvertently. 
