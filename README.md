![secret-coins](https://user-images.githubusercontent.com/98821241/267027498-a95bb02e-6943-46ce-8479-3d2bb7a80e43.svg)
![rock-paper-scissors-small](https://user-images.githubusercontent.com/98821241/267026680-cb9ea8c8-3a86-4d3c-86eb-923fc159c96c.png)

# Rock-Paper-Secret Workshop Istanbul

Welcome to the Secret Contract Workshop!
This document serves as an interactive guide to assist you as you follow along with the main presentation on the big screen.

Throughout the workshop, you will learn how to deploy a Secret Contract to your local network with a single node,
enabling you to play the timeless game of Rock, Paper, Scissors. You will also have the opportunity to level up your skills by adding
a new feature to the contract, which will allow a single player to play a match against a computer opponent, which plays at random!

## Your Workshop Hosts
TODO fix alignment
![Alex-profile-small](https://user-images.githubusercontent.com/98821241/268605584-1beb6305-91bc-4e16-9224-e5bf96387106.png)

<body>
    <div class="container">
        <img src="https://user-images.githubusercontent.com/98821241/268605584-1beb6305-91bc-4e16-9224-e5bf96387106.png" alt="Your Image" class="image">
        <p>Your text goes here and will be vertically centered.</p>
    </div>
</body>
</html>

![circle-profile-smallest-est-est](https://user-images.githubusercontent.com/98821241/268605642-122e8106-acfa-4a0e-a7de-e7f71a3338b4.png)
<p align="center">
  Eshel Bar Meir: Scrt Labs Developer. (TODO fun fact?)
</p>

Don't hesitate to ask questions and seek further explanations from our workshop hosts.
They're here to help you understand the material.

## Open Your Environment
We've prepared a [Gitpod environment](https://gitpod.io/new/#https://github.com/scrtlabs/rps/) that includes all the necessary
components, allowing you to focus exclusively on deploying the contracts and setting up the web server for the game. For those preferring to
work locally, the repository of the project is available [here](https://github.com/scrtlabs/rps/).

In the Gitpod menu, choose the editor you'd like to use for contract editing. We recommend selecting a **jetbrains-client**
such as Goland, as it includes a built-in **port forwarding** feature, which will prove useful for reaching the web server later.

When you open your environment, you'll find three tabs: </br>
![tabs](https://user-images.githubusercontent.com/98821241/267066179-1a5c7a11-b10d-4b5e-bdbd-09353a662ab1.png)

1. **Node**: This tabs represents the local node, which operates as the sole participant in our network. It generates new, empty blocks every 6 seconds.
2. **Terminal**: In the second tab, you'll have access to an open terminal.
3. **Faucet**: a service that provides free coins upon request. It can be used to fund new accounts as needed.

## Compile The Contract
Before we try to understand the contract, let's save a bit of time by converting it to something we can upload to our local Secret Network - a .wasm file. Run the following in the Terminal tab:
```bash
make build
# you can inspect the Makefile to see what it does
```

## Contract's Function
Let's review how the contract operates. It accepts three types of messages: 'New Game,' 'Join Game,' and 'Submit Choice,' where choices can be `rock`, `paper`, or `scissors`. With each message, it updates the internal game status for the specified game. After the game concludes, you can query for the winner
![image](https://user-images.githubusercontent.com/98821241/267093633-056c269d-cf1f-4ec3-bcd6-33a528997966.png)

## Upload the Contract
There are two steps for uploading the .wasm file we got in the compilation step.
```bash
make cli-store-contract
# check if we stored the contract successfully
secretcli query compute list-code
```
This step causes the code of the contract to exist in the state of the blockchain.
```bash
make cli-instantiate-contract
# check if we instantiated the contract successfully
secretcli query compute list-contract-by-code 1
```
This step causes the code to be instantiated and receive an address. This way, if we have two contracts with the same code, we only need to store the code once, and instantiate it twice.

## Interact with the Contract
Run the webserver that serves as the front-end interface for the contract:
```bash
cd ui
npm install  # install dependencies
npm run dev  # run the webserver
```
Now we have a webserver running on the environment on port 3000. If Gitpod's port forwarding is working correctly, you should now be able to access [localhost:3000](http://localhost:3000) on your browser. Alternatively, you can reach the front-end by issuing `gp info`, and prepending `3000-` to the `Workspace URL` that is shown there (after `https://`).

Then, on your browser, connect your wallet, and allow Keplr's request to connect to secretdev-1, which is the local node's `chain-id`.

### Receiving coins from the Faucet
You will want to fund your account by a small amount, so you will be able to bet, and to pay the fees associated with the game transactions. Copy the address provided at the top of the screen, and then use it in the following command:
```bash
curl http://localhost:5000/faucet?address=<your_address>
```

Now you can play the game with your friends!

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
>     .add_attribute_plaintext("result", winner);                            # you'll have to pupulate 'winner'
> 
> Ok(resp.add_events(vec![new_evt]))
> ```

You will have to recompile the contract and then store and instantiate it again. This will require you to change the address that is hardcoded inside the code. (Or you can wait for Secret Network's upcoming release that has a Contract upgradability feature!)

### Thanks for joining the workshop and keep on hacking!
