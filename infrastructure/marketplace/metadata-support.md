# Metadata Support

## Metadata Structure

Your collection's contract must return a valid URI for the `tokenURI()` (ERC721) or `uri()` (ERC1155) calls. Examples of valid token URIs include:

* A URL to an IPFS file (e.g., 'ipfs://Qm.......')
* A URL to an Arweave file (e.g., 'ar://....')
* An 'http://' or 'https://' URL.
* A base64 encoded JSON object (e.g., 'data:application/json;base64...')
* A UTF-8 encoded JSON object (e.g., 'data:application/json;utf8,....')

## Supported Media Formats

* Static images (.png, .jpg, .gif, .svg, .webp)
* Animated images (.gif, .webp)
* Videos (.mp4)

## Attributes

For the initial launch, Trover will exclusively support basic attributes presented in a grid format. More advanced features such as number ranges and dynamic or visual representations of attributes will be incorporated at a later date.

When including attributes for your NFTs on Trevor, ensure that string attributes are represented as strings (enclosed in quotes), and numeric properties are expressed as either floats or integers. This practice ensures that Trevor can appropriately display and interpret the attributes.
