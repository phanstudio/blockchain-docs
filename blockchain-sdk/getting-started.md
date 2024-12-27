---
icon: square-poll-vertical
---

# Getting Started

## Getting Started with Blockchain SDK

<figure><img src="../.gitbook/assets/Screenshot 2024-12-26 232559.png" alt=""><figcaption><p>Basic usage of the blockchan sdk</p></figcaption></figure>

### Introduction

The Blockchain SDK is a comprehensive toolkit for integrating blockchain functionality into Godot applications, particularly focused on Web3 and Ethereum-compatible networks. It provides a robust set of features including:

* Wallet management and connectivity
* Smart contract interaction
* ABI conversion utilities
* BigNumber handling for crypto calculations
* Chain switching capabilities
* Contract deployment and management

The SDK is designed to be modular and easy to integrate, with built-in support for multiple networks including Ethereum, Polygon, and Sei (mainnet, testnet, and devnet).

### Installation

1. Create a new or open an existing Godot project
2. Create an `addons` directory in your project root if it doesn't exist
3. Copy the blockchain\_sdk folder into the `addons` directory
4. Enable the plugin in Godot:
   * Go to Project → Project Settings → Plugins
   * Find "Blockchain SDK" in the list
   * Check the "Enable" checkbox

Required HTML dependencies for web builds:

```html
<script src="https://cdn.jsdelivr.net/gh/ethereum/web3.js/dist/web3.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.umd.min.js"></script>
```

### Basic Setup

1.  **Initialize Web3Global** The SDK automatically initializes the Web3Global autoload singleton which provides access to key managers:

    ```gdscript
    # Access wallet manager
    var wallet = Web3Global.wallet_manager

    # Access contract manager
    var contracts = Web3Global.contract_manager
    ```
2.  **Configure Wallet Support** The SDK supports multiple chains out of the box:

    ```gdscript
    # Available networks
    "ethereum": "0x1"    # Ethereum Mainnet
    "polygon": "0x89"    # Polygon Mainnet
    "Sei": "0x531"      # Sei Mainnet
    "Sei-devnet": "0xAE3F3"
    "Sei-testnet": "0x530"
    ```
3.  **Set up Contract Handling** The SDK includes an ABI converter for generating GDScript contract interfaces:

    ```gdscript
    # Generate contract interface from ABI JSON
    var converter = AbiConverter.new()
    converter.convert_from_file("path/to/contract.json")
    ```

### Quick Start Guide

Here's a basic example of connecting a wallet and interacting with a smart contract:

1.  **Connect Wallet**

    ```gdscript
    extends Node

    func _ready():
        # Connect wallet signals
        Web3Global.wallet_manager.wallet_connected.connect(_on_wallet_connected)
        Web3Global.wallet_manager.wallet_disconnected.connect(_on_wallet_disconnected)
        
    func connect_wallet():
        Web3Global.wallet_manager.connect_wallet()
        
    func _on_wallet_connected(address: String):
        print("Wallet connected: ", address)
        
    func _on_wallet_disconnected():
        print("Wallet disconnected")
    ```
2.  **Load and Use Contract**

    ```gdscript
    # Assuming you have generated a contract class from ABI
    var my_contract = MyContractClass.new()

    # Query contract (view function)
    func get_data():
        var result = await my_contract.someViewFunction()
        print("Contract data: ", result)

    # Execute contract (state-changing function)
    func execute_transaction():
        await my_contract.someWriteFunction({
            "value": "1000000000000000000"  # 1 ETH in wei
        })
    ```
3.  **Handle BigNumbers**

    ```gdscript
    # Create BigNumber instances for precise calculations
    var amount = BigNum.new("1000000000000000000")  # 1 ETH
    var fee = BigNum.new("50000000000000000")      # 0.05 ETH

    # Perform calculations
    amount.sub(fee)  # Subtract fee from amount
    ```
4.  **Switch Networks**

    ```gdscript
    func switch_to_testnet():
        await Web3Global.wallet_manager.switch_network("Sei-testnet")
    ```

This Quick Start Guide covers the basic functionality. For more advanced features and detailed API documentation, please refer to the specific sections in the documentation.

Note: Remember that this SDK is designed for web builds and requires appropriate HTML dependencies to be included in your web export template.
