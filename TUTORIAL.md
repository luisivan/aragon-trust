# Creating a new Aragon Trust

## 1. Create a democracy DAO

Head over to [Aragon](https://app.aragon.org) and create a new DAO with the *Democracy template*. Sensible default for voting parameters:

- Support: 100%
- Quorum: 0%
- Duration: One week

## 2. Install aragonCLI

[aragonCLI](https://hack.aragon.org/docs/cli-intro.html) installs the `dao` command, the tool for Aragon power users.

```
$ npm install -g @aragon/cli
```

*Note: Add `--environment aragon:mainnet` to all the `dao` commands if you are interacting with the Ethereum mainnet*

## 3. Install Agent

[Aragon Agent](https://blog.aragon.one/aragon-agent-beta-release/) allows your Aragon DAO to interact with third-party contracts, such as the Gnosis multisig.

```
$ dao install {ORG_NAME}.aragonid.eth agent
```

## 4. Get the Agent and Voting addresses

After Aragon Agent is installed, let's get its Ethereum address. This command will display a table, and you need to copy the last address on it, which will be the Agent's, and also the Voting's, which will be displayed next to the word `voting`.

```
$ dao apps {ORG_NAME}
```

## 5. Allow Voting to operate Agent

Now we will allow the voting app on your DAO to execute actions in behalf of the Agent app, and therefore in behalf of the DAO.

```
$ dao acl create {ORG_NAME} {AGENT_ADDRESS} EXECUTE_ROLE {VOTING_ADDRESS} {VOTING_ADDRESS}
```

## 6. Create a Gnosis multisig

For that, you need to head to [Gnosis multisig](https://wallet.gnosis.pm), click *Add* and then *Create a new wallet*.

You will add three signers: key #1, key #2, and the Agent app address.

------

That's it! Now off to creating your first transaction using our brand new Aragon Trust.

# Operating the Trust

## 1. Creating a transaction

You can use key #2 to create the first transaction on the multisig. This should be pretty straightforward by using the Gnosis multisig interface.

Once the transaction is signed with key #2, you can now create a vote on your DAO to confirm it and move the funds.

You will need the Ethereum address for your Gnosis multisig, and also the transaction ID of the transaction you want to confirm. Please note that the transaction ID needs to be converted from decimal to hexadecimal, by using a tool like [this one](https://www.rapidtables.com/convert/number/decimal-to-hex.html).

```
dao act {AGENT_ADDRESS} {MULTISIG_ADDRESS} "confirmTransaction(uint256)" {TRANSACTION_ID}
```

## 2. Voting to confirm the transaction

Now, you can use key #3 to vote on the transaction by heading to the Aragon client.

After that, wait for a week, and the transaction will be executed!

If you are in a hurry, you can use key #4 to vote as well and expedite the transaction.

```
dao act {AGENT_ADDRESS} {MULTISIG_ADDRESS} "confirmTransaction(uint256)" 0 --use-frame
```



# Adding your heirs (WIP)

> Note: This is still being documented, you can [track the pull request](https://github.com/luisivan/aragon-trust/pull/2)

