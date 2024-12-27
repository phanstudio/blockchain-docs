---
icon: hand-wave
cover: https://gitbookio.github.io/onboarding-template-images/header.png
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Welcome

## Welcome to the Contract Manager

This documentation will guide you through the process of using the Contract Manager, a potent tool within the Godot Blockchain SDK. The Contract Manager simplifies interactions with smart contracts, enabling you to seamlessly **query contract data, execute contract functions, and handle events emitted by your contracts.**

### Core Features:

*   **Smart Contract Instantiation:** Effortlessly create instances of your smart contracts within your Godot project using the provided `smartcontract` function. This function takes the contract's address and ABI as input.

    ```gdscript
    func smartcontract(contract_address, contract_abi):
        contract_abi = Json.stringify(contract_abi)

        var javascript_code = """
        const contract = new ethers.Contract(
            "%s",
            %s,
            window.signer
        );

        window.contract = contract // for testing
        """ % [contract_address, contract_abi]

        JavaScriptBridge.eval(javascript_code)

        contract = window.contract

        console.log(contract)

        delete_globals("contract")

        return contract
    ```
*   **Secure Execution (`runsafely`)**: The `runsafely` function provides a secure and reliable way to interact with your smart contracts. It includes built-in checks for wallet connection status, sufficient balance, and gas estimation, reducing the risk of transaction failures.

    ```gdscript
    func runsafely(contractmethod, args1:Array=[], _method:String= "query"): # execute or query
        if not processing:
            processing = true
            var runlogs
            var args:String = arr_to_str(args1)
            window.contractmethod = contractmethod.estimateGas
            var error = handelDefualtErrors(args)
            if not error:
                var javascript_code = """
                async function checkWillFailAsync() {
                    try {
                        const gasEstimate = await window.contractmethod(%s);
                        return {
                            willFail: false,
                            error: null,
                            gasEstimate: gasEstimate.toString()
                        };
                    } catch (error) {
                        return {
                            willFail: true,
                            error: error.message,
                            gasEstimate: null
                        };
                    }
                }
                window.result = checkWillFailAsync
                """%[args]
                JavaScriptBridge.eval(javascript_code);
                await wait_till(window.result().then(jsreturn))
                delete_globals("contractmethod")
                delete_globals("result")
                runlogs = jsreturnvalue
                jsreturnvalue = null
            else:
                runlogs = create_jsobj(error)
            if not runlogs.willFail: # add return values for success
                await run(contractmethod, args, _method)
                if output_logs["error"] == null:
                    # add a delay before this
                    return output_logs["output"]
                return ERROR
            updateoutput(runlogs.error)
            return ERROR
        updateoutput("still processing a transaction")
        return ERROR
    ```
*   **Simplified Data Handling**: Functions are provided to convert data types commonly used in smart contracts, like `BigNum` (used for large integers) and converting between units and decimals.

    ```gdscript
    func parseUnit(number, token_decimals=18):
        number = str(number)

        if number.begins_with("."):
            number = "0" + number

        var zero_filler = int(token_decimals)

        var decimal_index = number.find(".")

        var bignum = number

        if decimal_index != -1:
            var segment = number.right(-(decimal_index+1) )

            zero_filler -= segment.length()

            bignum = bignum.erase(decimal_index,1)

            for zero in range(zero_filler):
                bignum += "0"

        var zero_parse_index = 0

        if bignum.begins_with("0"):
            for digit in bignum:
                if digit == "0":
                    zero_parse_index += 1

                else:
                    break

        if zero_parse_index > 0:
            bignum = bignum.right(-zero_parse_index)

        if bignum == "":
            bignum = "0"

        return bignum+"n"
    ```
* **Event Handling**: Capture and respond to events emitted by your contracts with signal connections or by examining the results of the `runsafely` function when using the `"execute"` mode. This allows your game logic to dynamically react to on-chain events.
*   **Asynchronous Operations and Promises:** The SDK utilizes promises to manage asynchronous operations such as connecting to wallets and waiting for transaction confirmations. This ensures that your game doesn't freeze while waiting for blockchain operations to complete.

    ```gdscript
    func wait_till(promise, waittime= 0.05): # add time limit to this

        var state = "None"

        while true:
            state = await PromiseState(promise)
            await get_tree().create_timer(waittime).timeout
            if state in ["fulfilled", "rejected"]:
                break
    ```

### Getting Started

1. **Install the SDK:** Begin by installing the Godot Blockchain SDK to your project.
2. **Import Classes:** Import necessary classes like `ContractManager` and `JsWeb3Node` into your Godot scripts.
3. **Initialize `Web3Global`:** Make sure you have a `Web3Global` node in your scene tree to manage the SDK's core components.
4. **Connect a Wallet:** Establish a connection to a user's cryptocurrency wallet to enable interaction with the blockchain.
5. **Instantiate Your Contracts:** Create instances of your smart contracts, providing the contract address and ABI.
6. **Interact with Contracts:** Utilize the `runsafely` function to securely execute contract methods or query contract data.
7. **Handle Events:** Implement event listeners to respond to specific events emitted by your contracts.

You can find detailed guides and examples within the SDK documentation to help you get started with each of these steps. Let's begin building your blockchain-powered Godot game!

### Jump right in

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Getting Started</strong></td><td>Learn about the Plugin</td><td></td><td></td><td><a href="broken-reference">Broken link</a></td></tr><tr><td><strong>Basics</strong></td><td>Learn the basics of GitBook</td><td></td><td></td><td><a href="broken-reference">Broken link</a></td></tr></tbody></table>
