---
eip: <to be assigned>
title: Smart Non Fungible Token (SmartNFT)
description: A standar interface of smart non-fungible tokens representing smart assets (such as secure IoT devices) that can generate their own blockchain accounts and obey to users.
author: Javier Arcenegui <arcenegui@imse-cnm.csic.es>, Rosario Arjona <arjona@imse-cnm.csic.es>, Roberto Román <roman@imse-cnm.csic.es> and Iluminada Baturone <lumi@imse-cnm.csic.es>
discussions-to: https://ethereum-magicians.org/t/new-proposal-of-smart-non-fungible-token/7677
status: draft
type: standards track
category: erc
created: 2021-12-03
requires: 165, 721
---

## Abstract
The novelty introduced in this EIP is the proposal of smart Non-Fungible Tokens (NFTs), named as SmartNFTs, to represent smart assets such as IoT devices, which are physical smart assets. Smart assets can have a blockchain account (BCA) address to participate actively in the blockchain transactions, they are also identified as the utility of a user, they can establish secure communication channels with owners and users, and they operate dynamically with several operating modes associated with their token states. A SmartNFT is physically bound to a smart asset, for example an IoT device, because the device is the only one able to generate its BCA address from its private key. The physical asset is the only one in possesion of its private key. This can be ensured, for example, if the IoT device does not store the private key but uses a Physical Unclonable Function (PUF) that allows recovering its private key. SmartNFTs extend ERC-721 non-fungible tokens (which only allow representing assets by a unique identifier, as a possession of an owner). 
 
## Motivation
There are nowadays many smart assets, like IoT devices, that can interact actively with other blockchain participants, that is, they can have a BCA. Hence, we propose the SmartNFTs, which have the attribute "asset" to represent the BCA address associated with the physical smart asset. Besides, an smart asset is not only a possession of an owner but also an active agent that obeys to a user to carry out certain tasks in an application. Hence, the SmartNFTs have the NFT attribute "user", which is the BCA address of the user of the smart asset. In order to represent the secure dynamic activity of the smart asset, six more attributes are proposed: "timestamp", "timeout", "state", "hashK_OA", "hashK_UA", and "dataEngage".  The attribute "timestamp" registers in the blockchain whenever the smart asset checks it is bound to its token. The attribute "timeout" is the maximum delay time established for the asset to prove again the bounding. Token "state" is included to trace the operating mode of the smart asset while "hashK_OA", "hashK_UA", and "dataEngage" are public data added to trace if secure communication channels are established between the smart asset and its owner and user. SmartNFTS extend ERC-721 NFTs, which are designed to represent passive assets, which only have an owner that owns the token, can transfer ownership, and can approve others to act in its name. 

## Specification
This kind of non-fungible token is designed to represent a secure smart asset with the capacity of generating its own BCA and, hence, interacting with the blockchain. Besides, this SmartNFT can assign the token to a user and allows the establishment of secure communication channels with the owner and user. The following attributes are proposed for the token:

| Type | Name of variable | Defined by ERC-721 | Optional |
|--|--|--|--|
| uint256 | tokenId | Yes | No |
| address | owner | Yes | No |
| address | approved | Yes | Yes |
| address | approvedForAll | Yes | Yes |
| address | asset | No | Yes |
| address | user | No | Yes |
| enum | state | No | No |
| uint256 | hashK_OD | No | Yes |
| uint256 | HashK_UD | No | Yes |
| uint256 | dataEngagement | No | Yes |
| uint256 | timeStamp | No | No |
| uint256 | timeout | No | No |

The atributes do not need to be given in an specific order. The designer of the smart contract defines the order. Like in the ERC-721, the role of operator (approvedForAll) is an attribute more related to the owner than to the token.
 
Asset and user are optional attributes but at least one of them should be used in a SmartNFT. In the case of using only the attribute user, two states define if the token is assigned or not to a user. Figure 1 shows the corresponding states in a flow chart. When a token is created, transferred or unassigned, the token state is set to “notAssigned”. If the token is assigned to a valid user, the state is set to "userAssigned".
 
![Figure 1 : Flow chart of the token states with user defined (and asset undefined)](/Images/Figure1.png)
  
In the case of defining the asset attribute but not the user attribute, two states define if the token is waiting for authentication with the owner or authentication has finished successfully. Figure 2 shows the corresponding states in a flow chart. When a token is created or transferred to a new owner, then the token change its state to "waitingForOwner". In this state, the token is waiting for authentication by the owner. Once the asset is authenticated, its associated token changes its state to "engagedWithOwner".

![Figure 2 : Flow chart of the token states with asset defined (and user undefined)](/Images/Figure2.jpg)
 
Finally, if both the asset and user attributes are defined. The states define if the asset has been authenticated or not by the owner or the user (waitingForOwner, engagedWithOwner, waitingForUser and engagedWithUser respectively). The flow chart in Figure 3 shows all the possible state changes. The states related to the owner are the same as in Figure 2. The difference is that from the state “EngagedWithOwner”, the token can be assigned to a user. When a user is assigned, from the states "EngagedWithOwner", "waitingForUser" or "engagedWithUser", the token change its state to "waitingForUser". Once the asset is authenticated by the user, the state of its associated token is set to "engagedWithUser", and the user is able to use the token or asset.

 ![Figure 3 : Flow chart of the token states with user and asset defined](/Images/Figure3.jpg)
 
In order to complete a token transaction, a new owner must carry out a mutual authentication process off-chain with the asset and on-chain with the token, by using their blockchain accounts. Similarly, a new user must carry out a mutual authentication process. SmartNFTs define how the authentication processes start and finish. These autentication processes allow deriving fresh session cryptographic keys for secure communication between assets and owners, and between assets and users. Therefore, the trustworthiness of the assets can be traced even if new owners and users manage them. These processes are described below.
 
```solidity
pragma solidity ^0.8.0;

 /// @title SmartNFT: Extension of ERC-721 Non-Fungible Token Standard. 
 interface SmartNFT is ERC721{
    /// @dev This emits when the NFT is assigned as utility of a new user.
    ///  This event emits when the user of the token changes.
    ///  (`_addressUser` == 0) when no user is assigned.
    event UserAssigned(uint256 indexed tokenID, address indexed _addressUser);
    
    /// @dev This emits when user and asset finish mutual authentication process successfully.
    ///  This event emits when both the user and the asset prove they share a secure communication channel.
    event UserEngaged(uint256 indexed tokenID);
    
    /// @dev This emits when owner and asset finish mutual authentication process successfully.
    ///  This event emits when both the owner and the asset prove they share a secure communication channel.
    event OwnerEngaged(uint256 indexed tokenID);
    
    /// @dev This emits when it is checked that the timeout has expired.
    ///  This event emits when the timestamp of the SmartNFT is not updated in timeout.
    event TimeoutAlarm(uint256 indexed tokenID);

    /// @notice This function defines how the NFT is assigned as utility of a new user (if user is defined).
    /// @dev Only the owner of the SmartNFT can assign a user. If asset is defined, then the state of the token must be
    /// "engagedWithOwner","waitingForUser" or "engagedWithUser" and this function changes the state of the token defined by "_tokenID" to
    /// "waitingForUser". If asset is not defined, the state is set to "userAssigned". In both cases, this function sets the parameter 
    /// "addressUser" to "_addressUser". 
    /// @param _tokenId is the tokenID of the SmartNFT bound to the asset.
    /// @param _addressUser is the address of the new user.
    function setUser(uint256 _tokenId, address _addressUser) external; 

    /// @notice This function defines the initialization of the mutual authentication process between the owner and the asset.
    /// @dev Only the owner of the token can start this authentication process if asset is defined and the state of the token is "waitingForOwner".
    /// The function does not change the state of the token and saves "_dataEngagement" 
    /// and "_hashK_O" in the parameters of the token.
    /// @param _tokenId is the tokenID of the SmartNFT bound to the asset.
    /// @param _dataEngagement is the public data proposed by the owner for the agreement of the shared key.
    /// @param _hashK_O is the hash of the secret proposed by the owner to share with the asset.
    function startOwnerEngagement(uint256 _tokenId, uint256 _dataEngagement, uint256 _hashK_O) external;
 
    /// @notice This function completes the mutual authentication process between the owner and the asset.
    /// @dev Only the asset bound to the token can finish this authentication process provided that the state of the token is
    /// "waitingForOwner" and dataEngagement is different from 0. This function compares hashK_O saved in
    /// the token with hashK_A. If they are equal then the state of the token changes to "engagedWithOwner", dataEngagement is set to 0,
    /// and the event "OwnerEngaged" is emitted.
    /// @param _hashK_A is the hash of the secret generated by the asset to share with the owner.
    function ownerEngagement(uint256 _hashK_A) external; 
 
    /// @notice This function defines the initialization of the mutual authentication process between the user and the asset.
    /// @dev Only the user of the token can start this authentication process if asset and user are defined and
    /// the state of the token is "waitingForUser". The function does not change the state of the token and saves "_dataEngagement" 
    /// and "_hashK_U" in the parameters of the token.
    /// @param _tokenId is the tokenID of the SmartNFT bound to the asset.
    /// @param _dataEngagement is the public data proposed by the user for the agreement of the shared key.
    /// @param _hashK_U is the hash of the secret proposed by the user to share with the asset.
    function startUserEngagement(uint256 _tokenId, uint256 _dataEngagement, uint256 _hashK_U) external;
    
    /// @notice This function completes the mutual authentication process between the user and the asset.
    /// @dev Only the asset bound to the token can finish this authentication process provided that the state of the token is
    /// "waitingForUser" and dataEngagement if different from 0. This function compares hashK_U saved in
    /// the token with hashK_A. If they are equal then the state of the token changes to "engagedWithUser", dataEngagement is set to 0,
    /// and the event "UserEngaged" is emitted.
    /// @param _hashK_A is the hash of the secret generated by the asset to share with the user.
    function userEngagement(uint256 _hashK_A) external; 
 
    /// @notice This function checks if the timeout has expired.
    /// @dev Everybody can call this function to check if the timeout has expired. The event "TimeoutAlarm" is emitted
    /// if the timeout has expired.
    /// @param _tokenId is the tokenID of the SmartNFT bound to the asset.
    /// @return true if timeout has expired and false in other case.
    function checkTimeout(uint256 _tokenId) external returns (bool);
    
    /// @notice This function sets the value of timeout.
    /// @dev Only the owner of the token can set this value provided that the state of the token is "engagedWithOwner",
    /// "waitingForUser" or "engagedWithUser".
    /// @param _tokenId is the tokenID of the SmartNFT bound to the asset.
    /// @param _timeout is the value to assign to timeout.
    function setTimeout(uint256 _tokenId, uint256 _timeout) external; 
    
    /// @notice This function updates the timestamp, thus avoiding the timeout alarm.
    /// @dev Only the asset bound to the token can update its own timestamp.
    function updateTimestamp() external; 
    
    /// @notice This function lets obtain the tokenID from an address. 
    /// @dev Everybody can call this function. The code executed only reads from the blockchain.
    /// @param _addressSA is the address to obtain the tokenID from it.
    /// @return The tokenID of the token bound to the asset that generates _addressSA.
    function tokenFromBCA(address _addressSA) external view returns (uint256);
    
    /// @notice This function lets know the owner of the token from the address of the asset bound to the token.
    /// @dev Everybody can call this function. The code executed only reads from the blockchain.
    /// @param _addressSA is the address to obtain the owner from it.
    /// @return the owner of the token bound to the asset that generates _addressSA.
    function ownerOfFromBCA(address _addressSA) external view returns (address);
    
    /// @notice This function lets know the user of the token from its tokenID.
    /// @dev Everybody can call this function. The code executed only reads from the blockchain.
    /// @param _tokenId is the tokenID of the SmartNFT bound to the asset.
    /// @return The user of the token from its _tokenId.
    function userOf(uint256 _tokenId) external view returns (address);
    
    /// @notice This function lets know the user of the token from the address of the asset bound to the token.
    /// @dev Everybody can call this function. The code executed only reads from the blockchain.
    /// @param _addressSA is the address to obtain the user from it.
    /// @return The user of the token bound to the asset that generates _addressSA.
    function userOfFromBCA(address _addressSA) external view returns (address);
    
    /// @notice This function lets know how many tokens are assigned to a user.
    /// @dev Everybody can call this function. The code executed only reads from the blockchain.
    /// @param _addressUser is the address of the user.
    /// @return the number of tokens assigned to a user.
    function userBalanceOf(address _addressUser) external view returns (uint256);
    
    /// @notice This function lets know how many tokens of an particular owner are assigned to a user.
    /// @dev Everybody can call this function. The code executed only reads from the blockchain.
    /// @param _addressUser is the address of the user.
    /// @param _addressOwner is the address of the owner.
    /// @return the number of tokens assigned to a user from an owner.
    function userBalanceOfAnOwner(address _addressUser, address _addressOwner) external view returns (uint256);
}
```
This interface is an extension of the ERC-721, is compatible with the standard, and needs the ERC-721 interface. ERC-721 interface is available in https://eips.ethereum.org/EIPS/eip-721. Then, metadata and enumeration extensions are compatible and included in this draft.
 
## Rationale
This SmartNFT was developed because the ERC-721 standard does not define the "users" of an asset, only the "owners", and does not assign a blockchain account (BCA) address, "asset", to the physical asset. Smart assets (for example IoT devices) are increasing nowdays and they can be managed in a secure way if they are bound to SmartNFTs. SmartNFTs allow implementing a secure way to share a secret key between the owner and the asset and between the user and the asset, confirmed with the consensus of the blockchain. In this way, assets, owners, and users can ensure that they are exchanging information only with trusted parts.

**Secure Asset Bound to a SmartNFT**
 
Current non-fungible tokens are associated with passive assets, either virtual or physical things, but they do not include any standardized mechanism to bind the non-fungible token to the asset. Binding assets to NFTs is interesting because the "asset" can know anytime its "owner" and "user". The "asset" is an active part in any transfer of ownership and use. In addition, the "asset" is smart, for example to revoke orders from a non-authorized user, or to be inoperative until the authentication with the user or the owner is carried out.
 
**User Management Mechanism**
 
SmartNFTs allow implementing a new and useful user management mechanism. In the last few years many projects concerning assets sharing (for example vehicles) have been created and developed. We incorporate the blockchain account of the user of the token as another attribute in order to distinguish between users, who employ the asset for an specific application, and owners, who assign the token to users. Both users and owners can be traced by the blockchain.
 
**Secure Key Exchange Mechanism**
 
The engagements of the asset with an owner and a user are carried out after a mutual authentication protocol (for example based on elliptic curve Diffie-Hellman key exchange protocol). This protocol can be employed for a key agreement between the asset and its owner, in the one side, and the asset and its user, in the other side. 
 
When the SmartNFT is created or when the owmnership is transfered, the operating mode of the asset defined by the token state is "Waiting for owner". Assuming that the asset is an electronic physical asset, it saves in its memory the owner BCA address. The owner generates a pair of keys using the elliptic curve secp256k1 and the primitive element P used on this curve: a secret key (SK<sub>OD</sub>) and a Public Key (PK<sub>OD</sub>), so that PK<sub>OD</sub> = SK<sub>OD</sub>*P. To generate the shared key between owner and asset, (K<sub>O</sub>), the public key of the asset, (PK<sub>DEV</sub>), is employed as follows:
 
K<sub>O</sub>=PK<sub>DEV</sub>*SK<sub>OD</sub>

Using the function startOwnerEngagement, the owner saves PK<sub>OD</sub> and the hash of K<sub>O</sub> in the SmartNFT. The owner sends PK<sub>OD</sub> signed to the asset. The asset checks the signature to verify the identity of the owner and calculates:
 
K<sub>OD</sub> = SK<sub>DEV</sub>*PK<sub>OD</sub>:

If everything is correctly done, K<sub>O</sub> and K<sub>OD</sub> are the same since:
 
K<sub>O</sub>=PK<sub>DEV</sub>*SK<sub>OD</sub>=(SK<sub>DEV</sub>*P)*SK<sub>OD</sub>= SK<sub>DEV</sub>*(SK<sub>OD</sub>*P)=SK<sub>DEV</sub>*PK<sub>OD</sub>

Using the function ownerEngagement, the assset sends the K<sub>OD</sub> obtained and if it is the same as K<sub>O</sub>, then the state of the token changes to "Engaged with owner" and the event OwnerEngaged is sent. Once the asset receives the event, it changes its operation mode to "Engaged with owner". This process is shown in Figure 4. From this moment, the asset can be managed by the owner. 
 
 ![Figure 4: Steps in a successful owner and asset muthual authentication process](/Images/Figure4.jpg)
 
If the asset consults the blockchain and the state of its SmartNFT is "Waiting for user", the asset (assuming it is an electronic physical asset) saves in its memory the user BCA address. Then, a mutual authentication process is carried out with the user, as already done with the owner. If the user verifies the asset, the user sends the transaction associated with the function startUserEngagement. As in starOwnerEngagement, this function saves the public key generated by the user, PK<sub>UD</sub>, and the hash of K<sub>U</sub>=PK<sub>DEV</sub>*SK<sub>UD</sub>.
The user sends PK<sub>UD</sub> signed to the asset. The latter checks the signature and optionally checks PK<sub>UD</sub> on the attribute dataEngagement of its token. If the user is verified, the asset calculates:

K<sub>UD</sub> = SK<sub>DEV</sub>*PK<sub>UD</sub>:

If everything is correctly done, K<sub>U</sub> and K<sub>UD</sub> are the same since:
 
K<sub>U</sub>=PK<sub>DEV</sub>*SK<sub>UD</sub>=(SK<sub>DEV</sub>*P)*SK<sub>UD</sub>= SK<sub>DEV</sub>*(SK<sub>UD</sub>*P)=SK<sub>DEV</sub>*PK<sub>UD</sub>

Using the function userEngagement, the asset sends the K<sub>UD</sub> obtained and if it is the same as K<sub>U</sub>, then the state of the token changes to "Engaged with user" and the event UserEngaged is sent. Once the asset receives the event, it changes its operation mode to "Engaged with user". This process is shown in Figure 5. From this moment, the asset can be managed by the user. 
 
 ![Figure 5: Steps in a successful user and asset muthual authentication process](/Images/Fig5_rev.png)

Since the establishment of a shared secret is very important for a secure communication, we propose the inclusion of the attributes hashK_OD, hashK_UD, and dataEngage in the SmartNFT. The first two attributes define, respectively, the hash of the secret shared between the asset and its owner and between the asset and its user. Assets, owners, and users should check they are using the correct shared secrets. The attribute dataEngagement defines the public data needed for the agreement. If the mutual authentication fails, dataEngagement allows detecting which parts failed.
  
## Backwards Compatibility
This implementation is an extension of the ERC-721 standard, then it is not only compatible with the standard, but also is a way to improve some of the actual NFT tokens based on the ERC-721 standard.

## Test Cases
The test case presented on the paper shown below is addressed in <b>0x7eB5A03E7ED70ABf70fee48965D0411d37F335aC</b> and the code is available in <b>https://github.com/Hardblock-IMSE/Smart-Non-Fungible-Token</b>

## Reference Implementation
A first version was presented in a paper of the Special Issue <b>Security, Trust and Privacy in New Computing Environments)</b> of <b>Sensors</b> journal of <b>mdpi</b> editorial. The paper, entitled <b><k>Secure Combination of IoT and Blockchain by Physically Binding IoT Devices to Smart Non-Fungible Tokens Using PUFs</k></b>, was written by the same authors of this draft. The doi of this paper is: <b>https://doi.org/10.3390/s21093119</b>

## Security Considerations
In this draft, a generic system has been proposed for the creation of non-fungible tokens able to represent smart assets. A generic point of view based on the improvements of the current ERC-721 standard is provided, such as the implementation of the user management mechanism, which does not affect the token's ownership.
The main objective is to bind a smart physical asset with a non-fungible token. The physical asset should have the ability to generate a blockchain account from itself in a totally random way so that only the asset is able to know the secret from which the blockchain account is generated. In this way, identity theft is avoided and the asset can be proven to be completely genuine. In order to ensure this, it is recommended that only the manufacturer of the asset has the ability to create its associated token, since it is intended to be backward compatible. In the case of an IoT device, the device firmware will be unable to share and modify the secret. It is recommended that assets reconstruct their secrets from non-sensitive information such as the helper data associated with Physical Unclonable Functions (PUFs). Although a secure key exchange protocol has been proposed, the token is open to coexist with other types of key exchange.  

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
