use if to replace require to save some gas : 
org : 
    modifier onlyOwner(uint256 tokenId) {
        require(
            ERC721(this).ownerOf(tokenId) == _msgSender(),
            "OwnableApprovable: caller is not the owner"
        );
        _;
    }

modify:
    modifier onlyOwner(uint256 tokenId) {
        if(ERC721(this).ownerOf(tokenId) == _msgSender()){
            _;
        } else {
            revert("OwnableApprovable: caller is not the owner");
        }
    }