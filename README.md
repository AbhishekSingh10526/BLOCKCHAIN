# BLOCKCHAIN
BLOCKCHAIN PROJECT FOR PARKING LOT .
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ParkingPaymentSystem {
    address public owner;
    uint256 public parkingRate; // Rate per minute in wei

    mapping(address => uint256) public parkingStartTime;
    mapping(address => uint256) public parkingEndTime;
    mapping(address => uint256) public parkingAmountPaid;

    event Parked(address indexed user, uint256 startTime);
    event Unparked(address indexed user, uint256 endTime, uint256 amountPaid);

    constructor(uint256 _parkingRate) {
        owner = msg.sender;
        parkingRate = _parkingRate;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }

    function park() external payable {
        require(parkingStartTime[msg.sender] == 0, "Already parked");
        parkingStartTime[msg.sender] = block.timestamp;
        emit Parked(msg.sender, block.timestamp);
    }

    function unpark() external payable{
        require(parkingStartTime[msg.sender] > 0, "Not parked");
        require(parkingEndTime[msg.sender] == 0, "Already unparked");
        parkingEndTime[msg.sender] = block.timestamp;

        uint256 parkingDuration = parkingEndTime[msg.sender] - parkingStartTime[msg.sender];
        uint256 amountToPay = parkingDuration * parkingRate;

        if (msg.value > amountToPay) {
            // Refund overpayment
            payable(msg.sender).transfer(msg.value - amountToPay);
        } else {
            // Collect payment
            parkingAmountPaid[msg.sender] = amountToPay;
        }

        emit Unparked(msg.sender, parkingEndTime[msg.sender], amountToPay);
    }

    function withdrawFunds() external onlyOwner {
        payable(owner).transfer(address(this).balance);
    }
}
