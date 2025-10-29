// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.23;

import "@openzeppelin/contracts/access/Ownable.sol";

interface IAddressBook {
    struct Contact {
        uint id;
        string firstName;
        string lastName;
        uint[] phoneNumbers;
    }

    function addContact(
        string memory _firstName,
        string memory _lastName,
        uint[] memory _phoneNumbers
    ) external;

    function deleteContact(uint _id) external;

    function getContact(uint _id) external view returns (Contact memory);

    function getAllContacts() external view returns (Contact[] memory);
}

contract AddressBook is IAddressBook, Ownable {
    mapping(uint => Contact) private contacts;
    uint[] private contactIds;
    mapping(uint => uint) private idToIndex;
    uint private nextContactId;

    constructor() Ownable(msg.sender) {}

    function addContact(
        string memory _firstName,
        string memory _lastName,
        uint[] memory _phoneNumbers
    ) external override onlyOwner {
        uint currentId = nextContactId;
        contacts[currentId] = Contact({
            id: currentId,
            firstName: _firstName,
            lastName: _lastName,
            phoneNumbers: _phoneNumbers
        });
        
        idToIndex[currentId] = contactIds.length;
        contactIds.push(currentId);
        
        nextContactId++;
    }

    function deleteContact(uint _id) external override onlyOwner {
        require(idToIndex[_id] != 0 || contactIds[0] == _id, "Contact not found");

        uint index = idToIndex[_id];
        uint lastId = contactIds[contactIds.length - 1];
        
        contactIds[index] = lastId;
        idToIndex[lastId] = index;

        contactIds.pop();
        delete contacts[_id];
        delete idToIndex[_id];
    }

    function getContact(uint _id) external view override returns (Contact memory) {
        return contacts[_id];
    }

    function getAllContacts() external view override returns (Contact[] memory) {
        Contact[] memory allContacts = new Contact[](contactIds.length);
        for (uint i = 0; i < contactIds.length; i++) {
            allContacts[i] = contacts[contactIds[i]];
        }
        return allContacts;
    }
}


interface ISubmission {
    function deploy() external returns (address);
}

contract Submission is ISubmission {
    function deploy() external override returns (address) {
        AddressBook newAddressBook = new AddressBook();
        return address(newAddressBook);
    }
}# BL4.2
