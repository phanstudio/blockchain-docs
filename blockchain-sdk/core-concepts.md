---
icon: code-commit
---

# Core Concepts

### Architecture Overview

The Blockchain SDK is built on a modular architecture with several key components working together:

#### Component Structure

```
blockchain_sdk/
├── autoloads/
│   └── web3_global.gd       # Central singleton manager
├── modules/
│   ├── wallet.gd           # Wallet management
│   ├── contracts.gd        # Contract interactions
│   └── bignum.gd          # Big number handling
└── utils/
    ├── abi_converter.gd    # ABI to GDScript conversion
    └── js_web3_init.gd    # JavaScript bridge initialization
```

#### Key Components

1. **Web3Global Singleton**
   * Central access point for all SDK functionality
   * Manages instances of wallet and contract systems
   * Automatically initialized through Godot's autoload system
2. **JavaScript Bridge**
   * Handles communication between GDScript and Web3/Ethereum JavaScript APIs
   * Manages async operations and promise handling
   * Provides type conversion and data formatting
3. **Contract System**
   * Generates GDScript interfaces from contract ABIs
   * Manages contract state and interactions
   * Handles transaction processing and event listening

### Web3 Integration

The SDK integrates with Web3 through Godot's JavaScript bridge, providing seamless access to blockchain functionality.

#### JavaScript Bridge Implementation

```gdscript
extends Node
class_name JsWeb3Node

var window
var provider
var _ethers
var console
var contract
var signer

func _ready() -> void:
    if OS.has_feature("web"):
        window = JavaScriptBridge.get_interface('window')
        _ethers = JavaScriptBridge.get_interface("ethers")
        provider = window.current_provider
```

#### Promise Handling

The SDK includes a robust promise handling system for asynchronous operations:

```gdscript
func wait_till(promise, waittime = 0.05):
    var state = "None"
    while true: 
        state = await PromiseState(promise)
        await get_tree().create_timer(waittime).timeout
        if state in ["fulfilled", "rejected"]:
            break
```

### Contract Management

The contract management system provides a flexible way to interact with smart contracts.

#### Contract Loading

```gdscript
class_name ContractManager

func smartcontract(contract_address, contract_abi):
    contract_abi = Json.stringify(contract_abi)
    var javascript_code = """
        const contract = new ethers.Contract(
            "%s",
            %s,
            window.signer
        );
    """ % [contract_address, contract_abi]
    JavaScriptBridge.eval(javascript_code)
```

#### ABI Conversion

The SDK automatically converts contract ABIs to GDScript classes:

```gdscript
static func convert_abi_to_gdscript(contract_name: String, address: String, abi: Array) -> String:
    # Generate contract class
    var output = HEADER_TEMPLATE.format({
        "class_name": contract_name.capitalize(),
        "address": address,
        "abi": abi_string
    })
    
    # Generate methods for each ABI entry
    for item in abi:
        if item.type == "function":
            output += generate_function(item)
```

### Wallet System

The wallet system manages blockchain account connections and interactions.

#### Key Features

1. **Multiple Chain Support**

```gdscript
const SUPPORTED_CHAINS = {
    "ethereum": "0x1",    # Ethereum Mainnet
    "polygon": "0x89",    # Polygon Mainnet
    "Sei": "0x531",      # Sei Mainnet
    "Sei-devnet": "0xAE3F3",
    "Sei-testnet": "0x530"
}
```

2. **Wallet Connection Management**

```gdscript
func connect_wallet() -> void:
    if not OS.has_feature("web"):
        emit_signal("wallet_error", "Wallet connection only available in web builds")
        return
    await reconnect()
```

3. **Balance Tracking**

```gdscript
func get_balance() -> void:
    if not is_wallet_connected:
        emit_signal("connection_failed", "Wallet not connected")
        return
    await wait_till(window.ethereum.request({
        "method": 'eth_getBalance',
        "params": [wallet_address, "latest"]
    }))
```

### BigNum System

The BigNum system handles large numbers and precise calculations needed for cryptocurrency operations.

#### Features

1. **Initialization and Basic Operations**

```gdscript
class_name BigNum

func _init(_value: Variant=0) -> void:
    if _value is BigNum:
        value = _value.value
    elif typeof(_value) == TYPE_STRING:
        value = _value
    else:
        value = str(_value)
```

2. **Mathematical Operations**

```gdscript
# Addition
func add(_value: BigNum):
    value = _add_strings(actual_value, _value.actual_value)

# Multiplication
func mult(_value: BigNum):
    value = _multiply_strings(actual_value, _value.actual_value)
```

3. **String-Based Calculations**

```gdscript
static func _add_strings(str1: String, str2: String) -> String:
    # Pad numbers to equal length
    while str1.length() < str2.length():
        str1 = "0" + str1
    while str2.length() < str1.length():
        str2 = "0" + str2

    var carry = 0
    var result = ""

    # Add digits from right to left
    for i in range(str1.length() - 1, -1, -1):
        var digit_sum = int(str1[i]) + int(str2[i]) + carry
        result = str(digit_sum % 10) + result
        carry = digit_sum / 10

    if carry > 0:
        result = str(carry) + result

    return result
```

#### Usage Example

```gdscript
# Create BigNum instances
var amount = BigNum.new("1000000000000000000")  # 1 ETH
var fee = BigNum.new("50000000000000000")      # 0.05 ETH

# Perform calculations
amount.sub(fee)  # Subtract fee from amount
amount.mult(BigNum.new("2"))  # Double the amount
```

The BigNum system ensures precise handling of large numbers without floating-point precision issues, which is crucial for blockchain operations.

#### Best Practices

1. Always use BigNum for currency calculations
2. Handle string conversion appropriately for display
3. Consider gas fees and decimals in calculations
4. Use the built-in mathematical operations rather than implementing your own
5. Remember to handle potential overflow cases

This documentation covers the core concepts of the Blockchain SDK. Each component is designed to work seamlessly together while maintaining modularity for flexibility and maintainability.
