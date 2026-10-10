# MoonlitWhispers
MoonlitWhispers
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Strings.sol";

/// @title MoonlitWhispers
/// @notice 月明かりの下で交わされる秘密の囁きを永遠に記録するコントラクト
contract MoonlitWhispers is ERC721, Ownable {
    using Strings for uint256;

    uint256 public constant MAX_WHISPER_LENGTH = 280;
    uint256 public constant WHISPER_PRICE = 0.00008 ether;

    uint256 public totalWhispers;

    struct Whisper {
        address author;
        string content;
        uint256 energy;        // 共鳴によって蓄積されるエネルギー
        uint256 timestamp;
        bool isStarlight;      // エネルギーが一定値を超えると星になる
    }

    mapping(uint256 => Whisper) public whispers;
    mapping(uint256 => mapping(address => bool)) public hasResonated;

    event WhisperLeft(uint256 indexed tokenId, address author, string content);
    event Resonated(uint256 indexed tokenId, address resonator, uint256 newEnergy);
    event BecameStarlight(uint256 indexed tokenId);

    constructor() ERC721("MoonlitWhispers", "WHISPER") Ownable(msg.sender) {}

    /// @notice 新しい密語を残してNFTとしてミントする
    function leaveWhisper(string calldata _content) external payable {
        require(bytes(_content).length > 0 && bytes(_content).length <= MAX_WHISPER_LENGTH, "Invalid length");
        require(msg.value >= WHISPER_PRICE, "Insufficient ETH");

        totalWhispers++;
        uint256 tokenId = totalWhispers;

        whispers[tokenId] = Whisper({
            author: msg.sender,
            content: _content,
            energy: 1,
            timestamp: block.timestamp,
            isStarlight: false
        });

        _safeMint(msg.sender, tokenId);

        emit WhisperLeft(tokenId, msg.sender, _content);
    }

    /// @notice 他人の密語に共鳴する（エネルギーを増やす）
    function resonate(uint256 _tokenId) external {
        require(_ownerOf(_tokenId) != address(0), "Whisper does not exist");
        require(!hasResonated[_tokenId][msg.sender], "Already resonated");
        require(msg.sender != whispers[_tokenId].author, "Cannot resonate own whisper");

        hasResonated[_tokenId][msg.sender] = true;
        whispers[_tokenId].energy += 1;

        // エネルギーが一定を超えると星になる
        if (whispers[_tokenId].energy >= 12 && !whispers[_tokenId].isStarlight) {
            whispers[_tokenId].isStarlight = true;
            emit BecameStarlight(_tokenId);
        }

        emit Resonated(_tokenId, msg.sender, whispers[_tokenId].energy);
    }

    /// @notice 指定した密語の情報を取得する
    function getWhisper(uint256 _tokenId) external view returns (Whisper memory) {
        require(_ownerOf(_tokenId) != address(0), "Whisper does not exist");
        return whispers[_tokenId];
    }

    /// @notice 最新の密語を取得する
    function getLatestWhispers(uint256 _limit) external view returns (Whisper[] memory) {
        uint256 count = totalWhispers < _limit ? totalWhispers : _limit;
        Whisper[] memory result = new Whisper[](count);

        for (uint256 i = 0; i < count; i++) {
            result[i] = whispers[totalWhispers - i];
        }
        return result;
    }

    /// @notice メタデータを生成する
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        require(_ownerOf(tokenId) != address(0), "Token does not exist");

        Whisper memory w = whispers[tokenId];
        string memory starStatus = w.isStarlight ? "Starlight" : "Whisper";

        return string(abi.encodePacked(
            "data:application/json;utf8,",
            '{"name":"Moonlit Whisper #', tokenId.toString(),
            '","description":"', w.content,
            '","attributes":[',
            '{"trait_type":"Energy","value":', w.energy.toString(), '},',
            '{"trait_type":"Status","value":"', starStatus, '"}',
            ']}'
        ));
    }

    /// @notice コントラクト内のETHを引き出す
    function withdraw() external onlyOwner {
        (bool success, ) = payable(owner()).call{value: address(this).balance}("");
        require(success, "Withdraw failed");
    }
}
contract address	0x659D708b00ca29ADCdcbDEc386ee7E6e3d8A2b6c
