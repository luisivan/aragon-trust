![In Aragon We Trust](https://cdn-images-1.medium.com/max/1400/1*ycnh8TX8JkIor7wflKH3Vw.jpeg)

# Aragon Trust

An Aragon [Trust](https://www.investopedia.com/terms/t/trust.asp) is a kind of Aragon entity in which the trustee is the entity itself.

# Goals

- Stores assets on the [Gnosis multisig](https://github.com/gnosis/MultiSigWallet), which is the most proven smart contract for storing assets, while still having the flexibility of an Aragon entity.
- Allows for painless transfers with hot keys, to reduce the possibility of losing cold keys and the effort it takes to safeguard them.
- Lets the beneficiary immediately access funds if needed, with extra steps. By default, implements a time delay.
- Allows for the beneficiary to define heirs that can withdraw assets collectively (dead man's switch). They can also become a recovery mechanism.



*Disclaimer: This technology makes it easier to implement inheritance, but keep in mind that no technology can ensure that the funds are eventually directed towards the people or causes you originally intend to support*.

# Governance structure

*Note: support percentages and time delays can be tweaked as you want, these are just sensible defaults*



Aragon Trusts are composed of two sub-groups:

- Beneficiary (token symbol HOLD)
- Heirs (token symbol HEIR)



### Multisig

- Signers:
  - Key #1
  - Key #2
  - DAO (via [Aragon Agent](https://blog.aragon.one/aragon-agent-beta-release/))
  
  

### DAO

- **Token (HOLD)**: This token would be given to

  - Key #3: This can be a very hot key
  - Key #4: This can be hot, but should take some time to access it. It acts as a two factor auth

- **Voting (HOLD)**: All keys combined can move funds immediately, one can move funds with one week delay, one can veto the transfer
  - Support: 100%
  - Quorum: 0%
  - Duration: One week
  
- **Token (HEIRS)**: Arbitrary number of tokens
  - 33% of HEIRS tokens are burnt. This is so HEIRS votes always take their full duration, and are not immediately executed. If the beneficiary has passed away, then you need 66% of all remaining HEIRS holders to vote yes to transfer the funds
  
- **Voting (HEIRS)**: All transfers have 1 year delay and the beneficiary can revoke the permission for the HEIRS holders to transfer funds
  - Support: 66%
  - Quorum: 0%
  - Duration: One year
  



### DAO permissions

| App   | Permission     | Grantee        | Permission manager |
| ----- | -------------- | -------------- | ------------------ |
| Agent | Execute action | Voting (HOLD)  | Voting (HOLD)      |
| Agent | Execute action | Voting (HEIRS) | Voting (HOLD)      |

*TODO: Assign permissions to multisig to burn tokens on the DAO, in case a key is compromised. And also to add tokens, in case keys get lost.*

# Threat model

- The two multisig keys cannot get lost or stolen at the same time
- The DAO cannot break while also one multisig key gets lost or stolen
- Always initiate transactions to the multisig with key #2 and confirm with the DAO. Otherwise, attackers can predict that the vote will take one week and go to you the exact moment that you would need to sign the confirmation with key #4
- One of the hot keys in the DAO needs to be stored in a way so it takes you some effort to sign with it, in order to [prevent wrench attacks](https://xkcd.com/538/)
- If one of the keys in the DAO gets stolen, you should immediately burn its token by using the multisig
- The beneficiary needs to revoke the *Execute action* permission from the HEIRS Token Manager if HEIRS holders create a vote when the beneficiary is still alive
- Key #2 needs to be transmitted to the heirs in some way. You can even pre-sign a transaction that moves the funds to another address (e.g. a DAO composed of the people of your choice) and send your heirs a [Shamir Secret](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) that they can put together to execute that transfer

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

---

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

## 1. Creating a new token

## 2. Creating a new Voting app

## 3. Giving permission over the Agent to the Voting app

