// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract Project is ERC721URIStorage, Ownable, ReentrancyGuard {
    uint256 private _tokenIds;
    uint256 private _itemsSold;
    
    uint256 listingPrice = 0.001 ether;
    
    struct MarketItem {
        uint256 tokenId;
        address payable seller;
        address payable owner;
        uint256 price;
        bool sold;
    }
    
    mapping(uint256 => MarketItem) private idToMarketItem;
    
    event MarketItemCreated(
        uint256 indexed tokenId,
        address seller,
        address owner,
        uint256 price,
        bool sold
    );
    
    event MarketItemSold(
        uint256 indexed tokenId,
        address seller,
        address buyer,
        uint256 price
    );
    
    constructor() ERC721("NFT Marketplace", "NFTM") Ownable(msg.sender) {}
    
    /**
     * @dev Creates a new NFT and lists it on the marketplace
     * @param _tokenURI The metadata URI for the NFT
     * @param _price The price in wei for the NFT
     */
    function createAndListNFT(
        string memory _tokenURI,
        uint256 _price
    ) public payable nonReentrant {
        require(_price > 0, "Price must be greater than 0");
        require(msg.value == listingPrice, "Must pay listing fee");
        
        _tokenIds++;
        uint256 newTokenId = _tokenIds;
        
        _mint(msg.sender, newTokenId);
        _setTokenURI(newTokenId, _tokenURI);
        
        _createMarketItem(newTokenId, _price);
        
        emit MarketItemCreated(
            newTokenId,
            msg.sender,
            address(0),
            _price,
            false
        );
    }
    
    /**
     * @dev Purchases an NFT from the marketplace
     * @param _tokenId The ID of the NFT to purchase
     */
    function purchaseNFT(uint256 _tokenId) public payable nonReentrant {
        uint256 price = idToMarketItem[_tokenId].price;
        address seller = idToMarketItem[_tokenId].seller;
        
        require(msg.value == price, "Please submit the asking price");
        require(idToMarketItem[_tokenId].sold == false, "Item already sold");
        require(seller != msg.sender, "Cannot buy your own NFT");
        
        idToMarketItem[_tokenId].owner = payable(msg.sender);
        idToMarketItem[_tokenId].sold = true;
        _itemsSold++;
        
        _transfer(seller, msg.sender, _tokenId);
        
        payable(seller).transfer(msg.value);
        payable(owner()).transfer(listingPrice);
        
        emit MarketItemSold(_tokenId, seller, msg.sender, price);
    }
    
    /**
     * @dev Resells an owned NFT on the marketplace
     * @param _tokenId The ID of the NFT to resell
     * @param _price The new price for the NFT
     */
    function resellNFT(uint256 _tokenId, uint256 _price) public payable nonReentrant {
        require(idToMarketItem[_tokenId].owner == msg.sender, "Only owner can resell");
        require(msg.value == listingPrice, "Must pay listing fee");
        require(_price > 0, "Price must be greater than 0");
        
        idToMarketItem[_tokenId].sold = false;
        idToMarketItem[_tokenId].price = _price;
        idToMarketItem[_tokenId].seller = payable(msg.sender);
        idToMarketItem[_tokenId].owner = payable(address(0));
        _itemsSold--;
        
        _transfer(msg.sender, address(this), _tokenId);
    }
    
    // Helper function to create market item
    function _createMarketItem(uint256 _tokenId, uint256 _price) private {
        idToMarketItem[_tokenId] = MarketItem(
            _tokenId,
            payable(msg.sender),
            payable(address(0)),
            _price,
            false
        );
        
        _transfer(msg.sender, address(this), _tokenId);
    }
    
    // View functions
    function fetchMarketItems() public view returns (MarketItem[] memory) {
        uint256 itemCount = _tokenIds;
        uint256 unsoldItemCount = _tokenIds - _itemsSold;
        uint256 currentIndex = 0;
        
        MarketItem[] memory items = new MarketItem[](unsoldItemCount);
        for (uint256 i = 0; i < itemCount; i++) {
            if (idToMarketItem[i + 1].owner == address(0)) {
                uint256 currentId = i + 1;
                MarketItem storage currentItem = idToMarketItem[currentId];
                items[currentIndex] = currentItem;
                currentIndex += 1;
            }
        }
        return items;
    }
    
    function fetchMyNFTs() public view returns (MarketItem[] memory) {
        uint256 totalItemCount = _tokenIds;
        uint256 itemCount = 0;
        uint256 currentIndex = 0;
        
        for (uint256 i = 0; i < totalItemCount; i++) {
            if (idToMarketItem[i + 1].owner == msg.sender) {
                itemCount += 1;
            }
        }
        
        MarketItem[] memory items = new MarketItem[](itemCount);
        for (uint256 i = 0; i < totalItemCount; i++) {
            if (idToMarketItem[i + 1].owner == msg.sender) {
                uint256 currentId = i + 1;
                MarketItem storage currentItem = idToMarketItem[currentId];
                items[currentIndex] = currentItem;
                currentIndex += 1;
            }
        }
        return items;
    }
    
    function getListingPrice() public view returns (uint256) {
        return listingPrice;
    }
    
    function updateListingPrice(uint256 _listingPrice) public onlyOwner {
        listingPrice = _listingPrice;
    }
}
