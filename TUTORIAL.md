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
$ dao install <org-name> agent
```

## 4. Get the Agent and Voting addresses

After Aragon Agent is installed, let's get its Ethereum address. This command will display a table, and you need to copy the last address on it, which will be the Agent's, and also the Voting's, which will be displayed next to the word `voting`.

```
$ dao apps <org-name>
```

## 5. Allow Voting to operate Agent

Now we will allow the voting app on your DAO to execute actions in behalf of the Agent app, and therefore in behalf of the DAO.

```
$ dao acl create <org-name> <agent-address> EXECUTE_ROLE <voting-address> <voting-address>
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
$ dao act <agent-address> <multisig-address> "confirmTransaction(uint256)" <transaction-id>
```

## 2. Voting to confirm the transaction

Now, you can use key #3 to vote on the transaction by heading to the Aragon client.

After that, wait for a week, and the transaction will be executed!

If you are in a hurry, you can use key #4 to vote as well and expedite the transaction.

```
$ dao act <agent-address> <multisig-address> "confirmTransaction(uint256)" 0 --use-frame
```



# Adding your heirs (WIP)

## 1. Creating a new token

This will create a new ERC20 token. Please keep in mind that tokens are tranferable, and they can have decimals. This can be convenient so that heirs can distribute their voting power in different locations and devices.

```
$ dao token new "Heirs" "HEIRS" 18
```

## 2. Installing a new Token Manager

We will now install a new instance of the Token Manager app in our organization.

```
$ dao install <org-name> token-manager --app-init none
```

Now we can get the address for our newly installed Token Manager.
```
$ dao apps <org-name> --all
```

Then we set this new Token Manager as the controller of the HEIRS token.
```
$ dao token change-controller <heirs-token-address> <heirs-token-manager-address>
```

Now we will allow the first Voting app to add and remove HEIRS tokens.
```
$ dao acl create <org-name> <heirs-token-manager-address> MINT_ROLE <voting-address> <voting-address>
```

Finally, we initialize the Token Manager.
```
$ dao exec <org-name> <heirs-token-manager-address> initialize <heirs-token-address> true 0
```

## 3. Installing a new Voting app


## 3. Giving permission over the Agent to the Voting app

