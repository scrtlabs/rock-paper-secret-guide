![secret-coins](https://user-images.githubusercontent.com/98821241/267027498-a95bb02e-6943-46ce-8479-3d2bb7a80e43.svg)
![rock-paper-scissors-small](https://user-images.githubusercontent.com/98821241/267026680-cb9ea8c8-3a86-4d3c-86eb-923fc159c96c.png)

# Rock-Paper-Secret Workshop Istanbul

Welcome to the Secret Contract Workshop!
This document serves as an interactive guide to assist you as you follow along with the main presentation on the big screen.

Throughout the workshop, you will learn how to deploy a Secret Contract to your local network with a single node,
enabling you to play the timeless game of Rock, Paper, Scissors. You will also have the opportunity to level up your skills by adding
a new feature to the contract, which will allow a single player to play a match against a computer opponent, which plays at random!

## Your Workshop Hosts
![Alex-profile-small](https://user-images.githubusercontent.com/98821241/268605584-1beb6305-91bc-4e16-9224-e5bf96387106.png)
Alex Zaidelson: Scrt Labs CEO. Fun Fact: has three sons, all of them play ice hockey.

![Eshel-profile-small](https://user-images.githubusercontent.com/98821241/268605642-122e8106-acfa-4a0e-a7de-e7f71a3338b4.png)
Eshel Bar Meir: Scrt Labs Developer. Fun Fact: Crazy about Underwater Hockey.

Don't hesitate to ask questions and seek further explanations from our workshop hosts.
They're here to help you understand the material.

## Open Your Environment
We've prepared a [Gitpod environment](https://gitpod.io/new/#https://github.com/scrtlabs/rps/) that includes all the necessary
components, allowing you to focus exclusively on deploying the contracts and setting up the web server for the game.
You will need your GitHub account to use GitPod. For those preferring to
work locally, the repository of the project is available [here](https://github.com/scrtlabs/rps/).

In the Gitpod menu, choose the editor you'd like to use for contract reading.

When you open your environment, you'll find three tabs: </br>
**Jetbrains**:</br>
![tabs](https://github.com/scrtlabs/rock-paper-secret-guide/assets/98821241/4093f520-433c-4f16-a8ab-0f80ef77dc42)
</br>
**VS Code**:</br>
![image](https://github.com/scrtlabs/rock-paper-secret-guide/assets/98821241/b442f81d-6e20-466b-8d8e-b89d2b908c49)

1. **Faucet**: a service that provides free coins upon request. It can be used to fund new accounts as needed.
2. **Terminal**: In the second tab, you'll have access to an open terminal.
3. **Local Secret Node**, which operates as the sole participant in our network. It generates new, empty blocks every 6 seconds.
4. You can always open more terminals if you wish.

## Compile The Contract
Before we try to understand the contract, let's save a bit of time by compiling it to something we can upload to our local Secret Network - a .wasm file. Run the following in the Terminal tab:
```bash
make build
# you can inspect the Makefile to see what it does
```

## Contract's Function
Let's review how the contract operates. It accepts three types of messages: 'New Game,' 'Join Game,' and 'Submit Choice,' where choices can be `rock`, `paper`, or `scissors`. With each message, it updates the internal game status for the specified game. After the game concludes, you can query for the winner.
![image](https://user-images.githubusercontent.com/98821241/267093633-056c269d-cf1f-4ec3-bcd6-33a528997966.png)

### Privacy
This contract function relies on Secret Network's privacy features, which enable and ensure fair gameplay. Messages sent to the contract, in particular the choice of 
Rock, Paper, or Scissors by the player must remain confidential both as the input to the contract, and as the state saved by the contract. If these messages
were public, as is the case with other Smart Contract Networks, the player who played first would expose their choice to their opponent, who can win the bet every 
time.</br> 
Note that the contract does not concern itself with the encryption of the choice saved to storage, nor the decryption of the input. Secret Network takes care of that 
behind the scenes, allowing only encrypted messages to be received by the node.

Moreover, nodes that process transactions on Secret Network are not exposed to the data they are operating on, because Secret Network requires them to execute the
messages inside a secure enclave (more about [Trusted Execution Environments and Intel SGX](https://docs.scrt.network/secret-network-documentation/overview-ecosystem-and-technology/techstack/privacy-technology/intel-sgx)).

## Upload the Contract
There are two steps for uploading the .wasm file we got in the compilation step.
1. Store the contract on the Secret blockchain
    ```bash
    make cli-store-contract
    # check if we stored the contract successfully
    secretcli query compute list-code
    ```
    The output should look like this, showing that the contract was stored successfully:
    ```json
    [
        {
            "code_id": 1,
            "creator": "secret1ap26qrlp8mcq2pg6r47w43l0y8zkqm8a450s03",
            "code_hash": "008627e517b516b24f1b8f6020d98fc4659e68154242b4626c5a65ea570b8ea1"
        }
    ]
    ```
    This step causes the code of the contract to exist in the state of the blockchain.

2.  Instantiate the contract.
    This step causes the code to be instantiated and receive an address. This way, if we have two contracts with the same code, we only need to store the code once, and instantiate it twice.
    ```bash
    make cli-instantiate
    # check if we instantiated the contract successfully
    secretcli query compute list-contract-by-code 1
    ```
    The output should look like this:
    ```json
    [
        {
            "contract_address": "secret1mfk7n6mc2cg6lznujmeckdh4x0a5ezf6hx6y8q",
            "code_id": 1,
            "creator": "secret1ap26qrlp8mcq2pg6r47w43l0y8zkqm8a450s03",
            "label": "rps-game"
        }
    ]
    ```
    

## Interact with the Contract
Now that we have deployed and instantiated the contract, we can interact with it.

Build and run the webserver that serves as the front-end interface for the contract:
```bash
cd ui
npm install  # install dependencies
npm run dev  # run the webserver
```
We now have a webserver running on the environment on port 3000.

On another terminal window, issue the `gp url 3000` command. Copy the URL returned by the gp command (that’s the external URL of our web server running on port 3000), and paste it into your browser.

Then, on your browser, connect your Keplr wallet (install from [here](https://www.keplr.app/download) if you haven't yet).
Next, allow Keplr's request to connect to secretdev-1, which is the local node's `chain-id`.

### Receiving coins from the Faucet
You will want to fund your account by a small amount, so you will be able to bet, and to pay the fees associated with the game transactions.

Copy the address provided at the top of the screen (or from Keplr wallet),
![image](https://github.com/scrtlabs/rock-paper-secret-guide/assets/98821241/12f577f1-fdc1-45cc-a470-f96207e46523)

and then use it in the following command in the terminal:
```bash
curl http://localhost:5000/faucet?address=<your_address>
```

Now you can play the game with your friends! ✊✋✌️

## Bonus: Implement the "Vs Computer" part
At the bottom of the screen, there is also the interface to play against the computer, which currently won't work.
We now challenge you to extend the contract to support this feature! You will have to make the computer choose at random.

> Tip: Here's how easy it is to use random numbers inside Secret Contracts:
> ```rust
> let rand: &Vec<u8> = &env.block.random.unwrap().0;
> ```

> Tip 2: Here's how the frontend expects the contract to return the result of the transaction:
> ```
> let new_evt = Event::new("new_rps_vs_computer_game".to_string())
>     .add_attribute_plaintext("computer_choice", computer_choice.to_str())  # you'll have to populate 'computer_choice'
>     .add_attribute_plaintext("result", winner);                            # you'll have to populate 'winner'
> 
> Ok(resp.add_events(vec![new_evt]))
> ```

You will have to recompile the contract and then store and instantiate it again. This will require you to change the address that is hardcoded inside the code. (Or you can wait for Secret Network's upcoming release that has a Contract upgradability feature!)

### Thanks for joining the workshop and keep on hacking!
