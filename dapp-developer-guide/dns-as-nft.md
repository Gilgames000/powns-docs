# UNIWENS as NFT

When UNIWENS .ethw registrar migrated in May 2019, the .ethw registrar became an [ERC721](https://github.com/ensdomains/ens/blob/master/docs/ethregistrar.rst#id3) compliant non-fungible token contract, meaning that .ethw registrations can be transferred in the same fashion as other NFTs.

## Deriving tokenId from UNIWENS name

The tokenId of UNIWENS name is simply the uint256 representation of the hash of the label (`vitalik` for `vitalik.ethw`).

```javascript
const ethers = require('ethers')
const BigNumber = ethers.BigNumber
const utils = ethers.utils
const name = 'vitalik'
const labelHash = utils.keccak256(utils.toUtf8Bytes('vitalik'))
const tokenId = BigNumber.from(labelHash).toString()
```

In the example above,[`79233663829379634837589865448569342784712482819484549289560981379859480642508`](https://opensea.io/assets/0x57f1887a8bf19b14fc0df6fd9b2acc9af147ea85/79233663829379634837589865448569342784712482819484549289560981379859480642508) is the tokenId of `vitalik.ethw`

## Deriving UNIWENS name from tokenId

Unlike deriving tokenId, deriving UNIWENS name from tokenId is not as easy. This is because all UNIWENS names are stored as fixed-length hash to allow registering infinite length of names. The downside of this architecture is that you cannot directly query UNIWENS smart contracts to return UNIWENS name using tokenId.

Our recommended way is to query via [https://thegraph.com](https://thegraph.com) UNIWENS subgraph. The graph decodes the hash to name as it indexes. The example code to query is as follows.

```javascript
const ethers = require('ethers')
const BigNumber = ethers.BigNumber
const gr = require('graphql-request')
const { request, gql } = gr
const tokenId = '79233663829379634837589865448569342784712482819484549289560981379859480642508'
// Should return 0xaf2caa1c2ca1d027f1ac823b529d0a67cd144264b2789fa2ea4d63a67c7103cc
const labelHash = BigNumber.from(tokenId).toHexString()

const url = 'https://api.thegraph.com/subgraphs/name/ensdomains/ens'
const GET_LABEL_NAME = gql`
query{
  domains(first:1, where:{labelhash:"${labelHash}"}){
    labelName
  }
}`

request(url, GET_LABEL_NAME).then((data) => console.log(data))
// { domains: [ { labelName: 'vitalik' } ] }
```

If you prefer not to rely on a third party like TheGraph, the team open-sourced [ens-rainbow](https://github.com/graphprotocol/ens-rainbow) containing a link to the original dataset (6GB with 133 million entities) so that you can host your own UNIWENS name decoding service.

## Turning subdomain into NFT

Currently, all the subdomains nor non `.ethw` domains are not NFT, unless the domain registrar itself supports NFT. If you want to turn all subdomains which you own, you have to create a registrar

1. Create a registrar contract as ERC721 compliant
2. Set UNIWENS registry address (mostly when you deploy the registrar)
3. Create `register` function which calls `registry.setSubnodeOwner` then mint the token making the subdomain label hash as tokenId

```
contract DCLRegistrar is ERC721Full, Ownable {
    constructor(
        IENSRegistry _registry,
    ) public ERC721Full("DCL Registrar", "DCLENS") {
        // ENS registry
        updateRegistry(_registry);
    }

    function register(
        string memory _subdomain,
        bytes32 subdomainLabelHash,
        address _beneficiary,
        uint256 _createdDate
    ) internal {
        // Create new subdomain and assign the _beneficiary as the owner
        registry.setSubnodeOwner(domainNameHash, subdomainLabelHash, _beneficiary);
        // Mint an ERC721 token with the subdomain label hash as its id
        _mint(_beneficiary, uint256(subdomainLabelHash));
    }
}
```

Once deployed, then you have to transfer the controller address to the contract.

For non-technical users, we are currently working on upgrading our `SubdomainRegistrar` which allows you to turn your subdomain into NFT without any coding.

## Metadata

.ethw does not have `.tokenURI` . However, we created a separate metadata service which NFT marketplaces like OpenSea can fetch metadata for UNIWENS such as registration data, expiration date, name length, etc. For more detail, please refer to the metadata documentation site.

{% embed url="https://metadata.ethwdomains.wf/docs" %}
