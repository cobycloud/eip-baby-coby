---
title: A Factory and Registration based specification for ERC721 Ownership of Smart Contracts
description: <Description is one full (short) sentence>
author: <a comma separated list of the author's or authors' name + GitHub username (in parenthesis), or name and email (in angle brackets).  Example, FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: <Standards Track, Meta, or Informational>
category: <Core, Networking, Interface, or ERC> # Only required for Standards Track. Otherwise, remove this field.
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires: <EIP number(s)> # Only required when you reference an EIP in the `Specification` section. Otherwise, remove this field.
---

<!--
  READ EIP-1 (https://eips.ethereum.org/EIPS/eip-1) BEFORE USING THIS TEMPLATE!

  This is the suggested template for new EIPs. After you have filled in the requisite fields, please delete these comments.

  Note that an EIP number will be assigned by an editor. When opening a pull request to submit your EIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`.

  The title should be 44 characters or less. It should not repeat the EIP number in title, irrespective of the category.

  TODO: Remove this comment before submitting
-->

## Abstract

<!--
  The Abstract is a multi-sentence (short paragraph) technical summary. This should be a very terse and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this specification does.

  TODO: Remove this comment before submitting
-->

This EIP's primary purpose is to enable the safe transferability of 

## Motivation
There are several motivations for this framework:
- Enable the safe transferablity and sale of contracts on open markets.
- Reduce the deployment costs of look-alike contracts
- Enable a system for contract registration or contract portfolio mgmt

## Specification
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Overview
A Factory shall be constructed with two parameters:
 - ERC721 compliant Owner's contract address
 - an implementation contract address.

Upon deployment of the factory, the Owner's contract shall be deployed and the first owners' token shall be minted as Token ID# 0.

Owner's Token ID #0 shall own the Owner's contract.

The Owner of the Owner's contract may choose to mint more Owner's tokens on the Owner's contract. 

Owner's tokens can be spent at the Factory. Upon spending an Owner's token, a clone of the implementation shall be deployed and assigned to the spent Owner's token. 

The bearer of the Owner's token shall be the owner expressed by the implementation contract via the owner() interface which shall override the ERC167 owner interface typically seen at the time of this publication.

Owner's Token ID #1 shall own the first implementation deployment, Owner's Token ID #2 shall own the second implementation deployment, and so on.

### Interfaces

#### Public Interfaces
```solidity
function ownersAddr() public view returns (address) 
function collectionRegistry(uint256 tokenId_) public view returns (address) 
function CLONE_IMPLEMENTATION() public view returns (address) 
function OWNERS_IMPLEMENTATION() public view returns (address) 
```

#### Internal Interfaces
```solidity
function _createOwnersContract(string memory name_, string memory symbol_, string memory tokenURI_, uint256 maxSupply_, uint256 maxMint_) internal returns (address)
function _clone(address implementation_) public returns (address instance)
```
## Rationale

### Problem #1 Lack of support for transferablity or sale of smart contracts on open markets.

Current reliance on ERC173 does not afford contracts to be transferred on open markets such as Blur or Opensea. To enable contracts to immediately take advantage of current infrastructures, this improvement seeks to recommend a solution in which smart contract ownership can be bought and sold.  

### Problem #2 Look-alike deployments

##### The ethereum ecosystem is filled with development and deployments that are nearly identical, however as a community, individuals do not have a system for taking advantage of a factory to reduce gas costs. By utilizing an ERC1167 factory in the implementation of cloning templates which adhere to our specification, we see several benefits such as:
- Reduce gas costs across like-minded community deployments
- Consolidated security audits (audit the integrity of a clone)
- Common parameterization flags and interfaces (clones share same state potentials)


### Problem #3 Contract Portfolios

#### 


<!--
  The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

TBD

## Backwards Compatibility

<!--

  This section is optional.

  All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

No backward compatibility issues found.

## Test Cases

<!--
  This section is optional for non-Core EIPs.

  The Test Cases section should include expected input/output pairs, but may include a succinct set of executable tests. It should not include project build files. No new requirements may be be introduced here (meaning an implementation following only the Specification section should pass all tests here.)
  If the test suite is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`. External links will not be allowed

  TODO: Remove this comment before submitting
-->

## Reference Implementation

##### The minimum implementation must ultimately deploy three contracts:
- an ERC721 and **eip9999*** compliant contract to act as the Owner's tokens' contract
- a **EIP9999** compliant contract to act as the implementation contract
- a factory contract to as a registry



#### ERC721aOwnersBase.sol
```solidity
// SPDX-License-Identifier: MIT
// Original Creator: Baby Coby
// Once a Coby, Always a Coby.

pragma solidity ^0.8.0;


import '@openzeppelin/contracts/utils/Context.sol';
import '@openzeppelin/contracts/access/Ownable.sol';
import "../../token/ERC721aGatedBaseV2/ERC721aGatedBaseV2.sol";

/**
 * @dev
 */

contract GenericFactory is Context, Ownable {
    event Deployed(address addr);
    
    // Data for Collection, currently this only reserves 20 bytes.
    struct CollectionData {
        address contractAddress;
    }
    
    address private _ownersImplementation;
    address private _cloneImplementation;
    mapping (uint256 => CollectionData) private _collectionRegistry;

    /**
    * @dev
    */
    constructor(address ownersImplementation_, address cloneImplementation_, string memory name_, string memory symbol_, string memory tokenURI_, uint256 maxSupply_, uint256 maxMint_) {
        _ownersImplementation = ownersImplementation_;
        _cloneImplementation = cloneImplementation_;
        _createOwnersContract(name_, symbol_, tokenURI_, maxSupply_, maxMint_);
    }

    /**
    * @dev View of Owners Token
    */
    function OWNERS_ADDR() public view returns (address) {
        return _collectionRegistry[0].contractAddress;
    }

    /**
    * @dev View of contract addresses registered
    */
    function collectionRegistry(uint256 tokenId_) public view returns (address) {
        return _collectionRegistry[tokenId_].contractAddress;
    }

    /**
    * @dev Address of base contract
    */
    function cloneImplementation() public view returns (address) {
        return _cloneImplementation;
    }

    /**
    * @dev Address of base contract
    */
    function ownersImplementation() public view returns (address) {
        return _ownersImplementation;
    }

    /**
    * @dev Creates Owners token, which are required to be spent to assign ownership to a factory-enabled contract.
    */
    function _createOwnersContract(string memory name_, string memory symbol_, string memory tokenURI_, uint256 maxSupply_, uint256 maxMint_) internal returns (address) {
        address newCollection;
        newCollection = _clone(_ownersImplementation);
        ERC721aGatedBaseV2(newCollection).initialize(0, name_, symbol_, tokenURI_, maxSupply_, maxMint_);
        emit Deployed(newCollection);
        _collectionRegistry[0].contractAddress = newCollection;
        
        return _collectionRegistry[0].contractAddress;
    }


    
    fallback() external payable {
        address OWNERS_ADDR_ = OWNERS_ADDR();
        uint256 tokenId_;
        
        assembly {
            calldatacopy(0x0, 4, 36)
            tokenId_ := mload(0x0)
        }

        require(ERC721aGatedBaseV2(OWNERS_ADDR_).ownerOf(tokenId_) == msg.sender, "Sender is not authorized to spend Owners token.");
        require(_collectionRegistry[tokenId_].contractAddress == address(0), "Owners token is already spent.");

        address newCollection;
        newCollection = _clone(_cloneImplementation);
        _collectionRegistry[tokenId_].contractAddress = newCollection;
        emit Deployed(newCollection);

        // Execute external function from _cloneImplementation using delegatecall and return any value
        assembly {
            
            // copy function selector and any arguemnts
            calldatacopy(0, 0, calldatasize())
            // execute function call using the newCollection address
            let result := call(gas(), newCollection, callvalue(), 0, calldatasize(), 0, 0)

            // get any return value
            returndatacopy(0, 0, returndatasize())
            // return any return value or error back to the caller
            switch result
                case  0 {
                     revert(0, returndatasize())
                }
                default {
                    return(0, returndatasize())
                }

        }
        
        

       
    }

    
    /**
    * @dev Clones the implementation
    */
    function _clone(address implementation_) public returns (address instance) {
        // solhint-disable-next-line no-inline-assembly
        assembly {
            let ptr := mload(0x40)
            mstore(ptr, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
            mstore(add(ptr, 0x14), shl(0x60, implementation_))
            mstore(add(ptr, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
            instance := create(0, ptr, 0x37)
        }
        require(instance != address(0), "ERC1167: create failed");
    }
       

}

```

<!--
  This section is optional.

  The Reference Implementation section should include a minimal implementation that assists in understanding or implementing this specification. It should not include project build files. The reference implementation is not a replacement for the Specification section, and the proposal should still be understandable without it.
  If the reference implementation is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`. External links will not be allowed.

  TODO: Remove this comment before submitting
-->

## Security Considerations

<!--
  All EIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. EIP submissions missing the "Security Considerations" section will be rejected. An EIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

Needs discussion.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
