---
eip: 4500 
title:  NFT Memories Standard
description: <Description is one full (short) sentence>
author: Ken Woodruff (@kennethhoytwoodruff), David Menard (@davidmd222), Rocky Melchor (@rocky-m)
discussions-to: https://github.com/RealItems/EIPs/issues/1
status: Draft
type: Standards Track
category (*only required for Standards Track): ERC
created: 2021-10-27
requires: 721 
---

## Simple Summary

A standardized framework to describe and add 'memories' to an NFT in order to enable universal support across all NFT marketplaces and ecosystem participants. 

## Abstract

This standard allows contracts, such as NFTs that support [ERC-721](./eip-721.md) and [ERC-1155](./eip-1155.md) interfaces, to add 'memories' to an NFT.  This is intended for NFT dApps and marketplaces that want to support memories on top of existing NFT standards. Memories are stored as events by calling a method on the smart contract containing the NFT.  This ERC should be considered a minimal, gas-efficient building block for extending the [ERC-721](.eip-721.md) and [ERC-1155](.eip-1155.md) standards.

## Motivation
There are many dApps and marketplaces that enable creators to create, view, auction and sell NFTs.  While these NFTs are tradeable and sellable in perpetuity, their content is often immutable and static.  While these properties are desireable and even critical for the utility of NFTs, there are specific types of NFTs which need more than the orginal static content in order to be useful.  For example, if an NFT is a Phygital (a physical object with a digital twin represented by an NFT) that NFT needs a way to represent its 'life events' on the blockchain. Because Phygitals (examples are automobiles, art, musical instruments) have stories to tell there is a need for a standard to tell those stories, which we call memories.  The value of memories can be utilitarian in nature - such as adding maintenance and repair records to an automobile - or less quantifiable - such as adding concert footage to a violin.  While there are proprietary, siloed methods for adding memories to NFTs there is not yet a universal standard.

While this standard focuses on NFTs and compatibility with the ERC-721 and ERC-1155 standards, EIP-4500 does not require compatibility with ERC-721 and ERC-1155 standards. Any other contract could integrate with EIP-4500 to describe the memories for any other data structure. ERC-4500 is, therefore, a universal memory standard for many asset types.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
RFC 2119.

**ERC-721 and ERC-1155 compliant contracts MAY implement this ERC to provide a standard method of specifying memory information.**

dApps that support this standard **MUST** implement some method of storing the memory metadata. Standards for specifying this metadata are including in this EIP.

Implementers of this standard **MUST** have all of the following functions:

```solidity

pragma solidity ^0.6.0;
import "./IERC165.sol";

///
/// @dev Interface for the NFT Memories Standard
///
interface IERC4500 is IERC165 {
    /// ERC165 bytes to add to interface array - set in parent contract
    /// implementing this standard
    ///
    /// bytes4(keccak256("addMemory(uint256,string)")) == 0x2a55205a
    /// bytes4 private constant _INTERFACE_ID_ERC4500 = 0x2a55205a;
    /// _registerInterface(_INTERFACE_ID_ERC4500);

    /// @notice Called with the tokenID and the URI of the memory's metadata
    /// @param _tokenId - the NFT asset that the memory is added to 
    /// @param metadataURI - the URI that points to the memory metadata
    function addMemory(uint256 _tokenId, string calldata _metadataURI) external {
        require(_exists(_tokenId), "ERC721: operator query for nonexistent token");
        require(_isApprovedOrOwner(_msgSender(), _itemId), "CONTRACT::addMemory: caller is not owner nor approved");
        emit LogMemoryEvent(_itemId, _metadataURI);
    }

    event LogMemoryEvent(uint256 indexed itemId, string metadataURI );
}

interface IERC165 {
    /// @notice Query if a contract implements an interface
    /// @param interfaceID The interface identifier, as specified in ERC-165
    /// @dev Interface identification is specified in ERC-165.
    /// @return `true` if the contract implements `interfaceID` and
    ///  `interfaceID` is not 0xffffffff, `false` otherwise
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}
```

Implementers of this standard **MUST** implement the metadata using the following JSON schema (note that only tokenId and externalUrl are required):

```yaml
type: object
properties:
  tokenId:
    type: number
  externalUrl:
    type: string
  files:
    type: array
    items:
      type: object
      properties:
        type:
          type: string
        uri:
          type: string
  notes:
    type: string
  geoInfo:
    type: object
    properties:
      latitude:
        type: number
      longitude:
        type: number
  country_name:
    type: string
  region_name:
    type: string
  time_zone:
    type: string
  zip_code:
    type: string
  region_code:
    type: string
  ip:
    type: string
  userAgent:
    type: string
  userId:
    type: string
required: [
    "tokenId", "externalUrl"
  ]
```

Here is an example of this JSON schema.

```json
{
    "tokenId":410,
    "externalUrl":"https://polygonstaging.realitems.io/items/Oo4oGL4Y",
    "files":[ 
        { 
            "type": "image", 
            "uri": "ipfs://QmfJQyoTQBDPovYdrJgReFXarrzvP6G67MDxBESizredM" 
        },
        {
            "type": "video",
            "uri": "https://ipfs.io/ipfs/QmXYyFCE3fHaGi4QxqqamPj5JV3sqtgwByqsAsHYgib77"
        },
        {
            "type": "sound",
            "uri": "https://ipfs.io/ipfs/QmXYyFCE3fHaGi4QxqqamPj5JV3sqtgwByqsACsHYib77"
        }
    ],
    "notes":"This is a memory",
    "geoInfo": {
        "latitude":37.976,
        "longitude":-122.3359
    },
    "country_name":"United States",
    "region_name":"California",
    "time_zone":"America/Los_Angeles",
    "zip_code":"94806",
    "region_code":"CA",
    "ip":"43.199.84.252",
    "userAgent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.54 Safari/537.36",
    "userId":"ken@realitems.org"
}
```