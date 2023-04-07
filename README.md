This project is from [lw3](https://learnweb3.io/courses/c1d7081b-63a9-4c6e-b35c-9fcbbad418b2/lessons/7411199b-6463-4ffa-803d-80afa30585ec)

The associated smart contract is deployed on the Sepolia testnet at: [0x9A1eA39E30025fC237beF60666968844f8E31968](https://sepolia.etherscan.io/address/0x9A1eA39E30025fC237beF60666968844f8E31968)

Here are the smart contracts:

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

interface IWhitelist {
  // Getter function automatically generated from the publig mapping in the original contract.
  function whitelistedAddresses(address) external view returns (bool);
}
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "./IWhitelist.sol";

contract CryptoDevs is ERC721Enumerable, Ownable {
  // If set, the token will be a concatenation of 'baseURI' and 'tokenId'
  string _baseTokenURI;

  // price of one Crypto Dev NFT
  uint256 public _price = 0.01 ether;

  // Used to paude the contract if needed for emergency cases.
  bool public _paused;

  // the maximum number of cryptodevs
  uint256 maxTokenIds = 20;

  // total number of minted
  uint256 public tokenIds;

  // Whitelist contract instance
  IWhitelist whitelist;

  // bool to keep track of wether presale started or not
  bool public presaleStarted;

  // timestamp for when presale would end
  uint256 public presaleEnded;

  modifier onlyWhenNotPaused {
    require(!_paused, "Contract is paused!");
    _;
  }

  // ERC 721's contructor takes two string arguments, name and symbol of the collection.
  constructor (string memory baseURI, address whitelistContract) ERC721("Crypto Devs", "CD") {
    _baseTokenURI = baseURI;
    whitelist = IWhitelist(whitelistContract);
  }

  // onlyOwner modifier comes from the Ownable
  function startPresale() public onlyOwner {
    presaleStarted = true;
    presaleEnded = block.timestamp + 5 minutes;
  }

  function presaleMint() public payable onlyWhenNotPaused {
    require(presaleStarted && block.timestamp < presaleEnded, "Presale is not running");
    require(whitelist.whitelistedAddresses(msg.sender), "you are not whitelisted");
    require(tokenIds < maxTokenIds, "Exceeded maximum Crypto Devs supply");
    require(msg.value >= _price, "Ether sent is not correct");
    tokenIds += 1;
    //_safeMint is a safer version of the _mint function as it ensures that
    // if the address being minted to is a contract, then it knows how to deal with ERC721 tokens
    // If the address being minted to is not a contract, it works the same way as _mint
    _safeMint(msg.sender, tokenIds);
  }

  function mint() public payable onlyWhenNotPaused {
    require(presaleStarted && block.timestamp >=  presaleEnded, "Presale has not ended yet");
    require(tokenIds < maxTokenIds, "Exceeded maximum Crypto Devs supply");
    require(presaleStarted && block.timestamp < presaleEnded, "Presale is not running");

    tokenIds += 1;
    _safeMint(msg.sender, tokenIds);
  }

  function _baseURI() internal view virtual override returns (string memory) {
    return _baseTokenURI;
  }

  function setPaused(bool val) public onlyOwner {
    _paused = val;
  }

  function withdraw() public onlyOwner {
    address _owner = owner();
    uint256 amount = address(this).balance;
    (bool sent, ) = _owner.call{value: amount}("");
    require(sent, "Failed to send Ether");
  }

    // Function to receive Ether. msg.data must be empty
    receive() external payable {}

    // Fallback function is called when msg.data is not empty
    fallback() external payable {}
}
```
