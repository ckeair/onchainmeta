// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

library Base64 {
    string internal constant TABLE_ENCODE = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
    bytes  internal constant TABLE_DECODE = hex"0000000000000000000000000000000000000000000000000000000000000000"
                                            hex"00000000000000000000003e0000003f3435363738393a3b3c3d000000000000"
                                            hex"00000102030405060708090a0b0c0d0e0f101112131415161718190000000000"
                                            hex"001a1b1c1d1e1f202122232425262728292a2b2c2d2e2f303132330000000000";

    function encode(bytes memory data) internal pure returns (string memory) {
        if (data.length == 0) return '';

        // load the table into memory
        string memory table = TABLE_ENCODE;

        // multiply by 4/3 rounded up
        uint256 encodedLen = 4 * ((data.length + 2) / 3);

        // add some extra buffer at the end required for the writing
        string memory result = new string(encodedLen + 32);

        assembly {
            // set the actual output length
            mstore(result, encodedLen)

            // prepare the lookup table
            let tablePtr := add(table, 1)

            // input ptra
            let dataPtr := data
            let endPtr := add(dataPtr, mload(data))

            // result ptr, jump over length
            let resultPtr := add(result, 32)

            // run over the input, 3 bytes at a time
            for {} lt(dataPtr, endPtr) {}
            {
                // read 3 bytes
                dataPtr := add(dataPtr, 3)
                let input := mload(dataPtr)

                // write 4 characters
                mstore8(resultPtr, mload(add(tablePtr, and(shr(18, input), 0x3F))))
                resultPtr := add(resultPtr, 1)
                mstore8(resultPtr, mload(add(tablePtr, and(shr(12, input), 0x3F))))
                resultPtr := add(resultPtr, 1)
                mstore8(resultPtr, mload(add(tablePtr, and(shr( 6, input), 0x3F))))
                resultPtr := add(resultPtr, 1)
                mstore8(resultPtr, mload(add(tablePtr, and(        input,  0x3F))))
                resultPtr := add(resultPtr, 1)
            }

            // padding with '='
            switch mod(mload(data), 3)
            case 1 { mstore(sub(resultPtr, 2), shl(240, 0x3d3d)) }
            case 2 { mstore(sub(resultPtr, 1), shl(248, 0x3d)) }
        }

        return result;
    }

    function decode(string memory _data) internal pure returns (bytes memory) {
        bytes memory data = bytes(_data);

        if (data.length == 0) return new bytes(0);
        require(data.length % 4 == 0, "invalid base64 decoder input");

        // load the table into memory
        bytes memory table = TABLE_DECODE;

        // every 4 characters represent 3 bytes
        uint256 decodedLen = (data.length / 4) * 3;

        // add some extra buffer at the end required for the writing
        bytes memory result = new bytes(decodedLen + 32);

        assembly {
            // padding with '='
            let lastBytes := mload(add(data, mload(data)))
            if eq(and(lastBytes, 0xFF), 0x3d) {
                decodedLen := sub(decodedLen, 1)
                if eq(and(lastBytes, 0xFFFF), 0x3d3d) {
                    decodedLen := sub(decodedLen, 1)
                }
            }

            // set the actual output length
            mstore(result, decodedLen)

            // prepare the lookup table
            let tablePtr := add(table, 1)

            // input ptr
            let dataPtr := data
            let endPtr := add(dataPtr, mload(data))

            // result ptr, jump over length
            let resultPtr := add(result, 32)

            // run over the input, 4 characters at a time
            for {} lt(dataPtr, endPtr) {}
            {
               // read 4 characters
               dataPtr := add(dataPtr, 4)
               let input := mload(dataPtr)

               // write 3 bytes
               let output := add(
                   add(
                       shl(18, and(mload(add(tablePtr, and(shr(24, input), 0xFF))), 0xFF)),
                       shl(12, and(mload(add(tablePtr, and(shr(16, input), 0xFF))), 0xFF))),
                   add(
                       shl( 6, and(mload(add(tablePtr, and(shr( 8, input), 0xFF))), 0xFF)),
                               and(mload(add(tablePtr, and(        input , 0xFF))), 0xFF)
                    )
                )
                mstore(resultPtr, shl(232, output))
                resultPtr := add(resultPtr, 3)
            }
        }

        return result;
    }
}

contract SwordNft is ERC721, ERC721Enumerable, Ownable {
    mapping(string => bool) private takenNames;
    mapping(uint256 => Attr) public attributes;

    struct Attr {
        string creator;
        string material;
        string testname;
        string subject;
    }

    constructor() ERC721("Sword", "SWORD") {}

    function uint2str(uint _i) internal pure returns (string memory _uintAsString) {
        if (_i == 0) {
            return "0";
        }
        uint j = _i;
        uint len;
        while (j != 0) {
            len++;
            j /= 10;
        }
        bytes memory bstr = new bytes(len);
        uint k = len;
        while (_i != 0) {
            k = k-1;
            uint8 temp = (48 + uint8(_i - _i / 10 * 10));
            bytes1 b1 = bytes1(temp);
            bstr[k] = b1;
            _i /= 10;
        }
        return string(bstr);
    }

    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._beforeTokenTransfer(from, to, tokenId);
    }

    function _burn(uint256 tokenId) internal override(ERC721) {
        super._burn(tokenId);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }

    function mint(
        address to, 
        uint256 tokenId, 
        string memory _creator, 
        string memory _material, 
        string memory _testname,
        string memory _subject)
    public onlyOwner {
        _safeMint(to, tokenId);
        attributes[tokenId] = Attr(_creator, _material, _testname, _subject);
    }

    function getSvg(uint tokenId) private view returns (string memory) {
        string memory svg;
        svg = "https://ntttest1.s3.us-west-1.amazonaws.com/ntttest.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjENb%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLXdlc3QtMSJIMEYCIQC3IFwRvJPMU1B8p0DOO0sq78%2BLAJFQOCDcjn69h29dWAIhAJrY8y6tgnxeIJ6h3HZ7E3qsc7cQtxoIDnEZgemrNL99Ku0CCN%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQAhoMNDg0MDE2NTU4Nzg3IgxK4JpwqVjIgjqSKJsqwQLb%2B40mVduZeQUg9Lq29ccCvnqyM1pZ2dllpHXK0PHj%2FL%2B4beTtcdwjc3E6UB1Wl3isExKg5%2BrKH5Ahv9qnyqviCX4SfNzp9apBKuvjnZodSzmVM%2FMDTWTP8WbxokTwX6C2L66bJ1njo4uXemDyT9qmdIR4DUdu64cGKZ%2BtO1OiLwW8Iz2IcYqHqnm38F6OLSg1ouLeJs7KSs7CqeDHA8coIE1O1gjVYUKAuLKk2mRmnOdrWQ%2FcT72lvgDPbUQw3f9f7ZoC%2F6ygyGuj56bwGv3PCwHRnaqj3iZIFd2YecLcOS1twISSJxpy1XCzp%2BRyYAwkN94pU2yR5xOXNSDrCQS8msamA9I1dxXO5nB4oaXswF4Ak%2FwbIrDUBirZ43G3l%2Bp8FPuov7TzNgbx66jXmz6XgU%2BvpynqdZFqymSNLlw441Ywo%2B2zlQY6sgLUfaZSHNKWTu2ewVNAouktq6Nf2WM%2FTXdGQQrYu8TunK4L7N4u5lF65kpF4%2FXsPlP7F2qRRvIZrP%2Fmfzm5ncC3ZqF6%2BAVp2h7qXfuUR74KeEZAW%2Bbkx4IQCuH9CcgeTDzFRbaIl59A7kpqkrWV67U3sYV%2FQw0mZ1Axu1wb6rnQBybKAvrLhPRhRYtff2vAfneI5BZuxkBYOvoi4admGSOqTmytbXgIRfQSsjFXOq6mfNzI7ZVeR1PB34nwSrmuAmxSyrZFgZIc%2BwsfBXbd6zRpy0mJBBabiWRCaYW%2BndKsdQWXa62kq08VoK9UoffFNLVPa%2FQI8iuHmW4XQ9Npivb68mafkRt3LwJcPjm0s0zqJQhqYcxWlXwzPvEMN7jP3pvVUUlMfRi4nntoMMYPbe1Cgoc%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220617T215119Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIAXBMNCSLBZHSGU74K%2F20220617%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=b53da65af6d679fefdc26c3d0e13141ed66ff8e4b366f2650c9f4ea47b4a3555";
      return svg;
   }    

    function tokenURI(uint256 tokenId) override(ERC721) public view returns (string memory) {
        string memory json = Base64.encode(
            bytes(string(
                abi.encodePacked(
                    '{"name": "', attributes[tokenId].creator, '",',
                    '"image_data": "', getSvg(tokenId), '",',
                    '"attributes": [{"trait_type":  "testname", "value": "', (attributes[tokenId].testname), '"},',
                    '{"trait_type": "Material", "value": "', attributes[tokenId].creator, '"},',
                    '{"trait_type": "Material", "value": "', attributes[tokenId].material, '"},',
                    '{"trait_type": "Material", "value": "', attributes[tokenId].testname, '"},',
                    '{"trait_type": "Material", "value": "', attributes[tokenId].subject, '"},',
                    ']}'
                )
            ))
        );
        return string(abi.encodePacked('data:application/json;base64,', json));
    }    
}
