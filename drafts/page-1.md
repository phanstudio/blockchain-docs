# Page 1

## Blockchain SDK for Godot

A comprehensive Godot plugin for managing cryptocurrency wallets and interacting with smart contracts. This SDK provides a modular approach to blockchain integration in Godot games and applications.

### Table of Contents

1. Installation
2. Quick Start
3. Core Components
4. Wallet Management
5. Contract Interaction
6. Utilities
7. Examples

### Installation

1. Download the Blockchain SDK addon
2. Place it in your Godot project's `addons` directory
3. Enable the plugin in Project Settings -> Plugins
4. Add the required JavaScript dependencies to your HTML template:

```html
<script src="https://cdn.jsdelivr.net/gh/ethereum/web3.js/dist/web3.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.umd.min.js"></script>
```

### Quick Start

```gdscript
extends Node

var wallet_manager: Wallet
var contract_manager: ContractManager

func _ready():
    # Access the global Web3 instance
    wallet_manager = Web3Global.wallet_manager
    contract_manager = Web3Global.contract_manager
    
    # Connect to wallet
    wallet_manager.connect("wallet_connected", _on_wallet_connected)
    wallet_manager.connect("balance_updated", _on_balance_updated)
    wallet_manager.connect_wallet()

func _on_wallet_connected(address: String):
    print("Connected to wallet: ", address)
    
func _on_balance_updated(balance: String):
    print("Current balance: ", balance)
```

### Core Components

#### Web3Global

The `Web3Global` autoload provides centralized access to core functionality:

* `wallet_manager`: Handles wallet connections and account management
* `contract_manager`: Manages smart contract interactions

#### Wallet Manager

Handles all wallet-related operations including:

* Connecting/disconnecting wallets
* Network switching
* Balance monitoring
* Account management

#### Contract Manager

Manages smart contract interactions:

* Contract deployment
* Function calls
* Event handling
* Transaction management

### Wallet Management

#### Connecting to a Wallet

```gdscript
func connect_wallet():
    wallet_manager.connect("wallet_connected", _on_wallet_connected)
    wallet_manager.connect("wallet_disconnected", _on_wallet_disconnected)
    wallet_manager.connect("wallet_error", _on_wallet_error)
    wallet_manager.connect_wallet()

func _on_wallet_connected(address: String):
    print("Wallet connected: ", address)
```

#### Supported Networks

The SDK supports multiple networks:

```gdscript
const SUPPORTED_CHAINS = {
    "ethereum": "0x1",  # Ethereum Mainnet
    "polygon": "0x89",  # Polygon Mainnet
    "Sei": "0x531",    # Sei Mainnet
    "Sei-devnet": "0xAE3F3",
    "Sei-testnet": "0x530"
}
```

#### Switching Networks

```gdscript
# Switch to Sei devnet
wallet_manager.switch_network("Sei-devnet")
```

#### Getting Wallet Balance

```gdscript
func check_balance():
    wallet_manager.connect("balance_updated", _on_balance_updated)
    wallet_manager.get_balance()

func _on_balance_updated(balance: String):
    print("Current balance: ", balance)
```

### Contract Interaction

#### Initializing a Contract

```gdscript
var contract: JavaScriptObject
var contract_manager: ContractManager = Web3Global.contract_manager

func init_contract():
    var address = '0x1234...' # Contract address
    var abi = [...] # Contract ABI
    contract = contract_manager.smartcontract(address, abi)
```

#### Calling Contract Functions

```gdscript
# Query (View) Functions
func query_contract():
    var result = await contract_manager.runsafely(
        contract.viewFunction,
        [param1, param2],
        "query"
    )

# Execute (Write) Functions
func execute_contract():
    await contract_manager.runsafely(
        contract.writeFunction,
        [param1, param2],
        "execute"
    )
```

#### Handling Big Numbers

The SDK includes a `BigNum` class for handling large numbers safely:

```gdscript
var number = BigNum.new("1000000000000000000")
number.add(BigNum.new("1"))
print(number.value) # "1000000000000000001n"
```

### Utilities

#### Number Formatting

```gdscript
# Parse unit with decimals
func format_amount(amount: float, decimals: int = 18) -> String:
    return contract_manager.parseUnit(amount, decimals)

# Parse big number to human-readable format
func parse_amount(amount: String, decimals: int = 18) -> float:
    return contract_manager.parseBigNumToNumber(amount, decimals)
```

#### Address Formatting

```gdscript
# Shorten address for display
func format_address(address: String) -> String:
    return contract_manager.shorten_hex(address)
```

### Examples

#### Basic Contract Interaction

```gdscript
extends Node

var test_contract: TestEventContract

func _ready():
    test_contract = TestEventContract.new()
    
    # Create a new phase
    await test_contract.createPhase()
```

#### Precompiled Address Contract

```gdscript
extends Node

var address_contract: PrecompliedAddressContract

func _ready():
    address_contract = PrecompliedAddressContract.new()
    
    # Get EVM address
    var evm_address = await address_contract.getEvmAddr("sei_address")
    print("EVM Address: ", evm_address)
    
    # Get SEI address
    var sei_address = await address_contract.getSeiAddr("0x...")
    print("SEI Address: ", sei_address)
```

#### Event Handling

```gdscript
func handle_contract_event():
    contract_manager.connect("contract_execution_result", _on_contract_result)
    
func _on_contract_result(result):
    if "logs" in result:
        print("Event logs: ", result.logs)
```

### Error Handling

The SDK includes comprehensive error handling:

```gdscript
func safe_contract_call():
    var result = await contract_manager.runsafely(
        contract.function,
        [params]
    )
    
    if result == contract_manager.ERROR:
        print("Error occurred during contract call")
        return
        
    print("Success: ", result)
```

### Best Practices

1. Always check wallet connection status before making contract calls
2. Use try-catch blocks for contract interactions
3. Implement proper error handling for user feedback
4. Use BigNum for handling large numbers
5. Verify transaction receipts for write operations
6. Keep private keys and sensitive data secure
7. Test thoroughly on test networks before mainnet deployment

Remember to handle errors appropriately and provide feedback to users when interacting with the blockchain.
