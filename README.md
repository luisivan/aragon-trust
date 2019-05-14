# Aragon Trust

An Aragon [Trust](https://www.investopedia.com/terms/t/trust.asp) is a kind of Aragon entity in which the trustee is the entity itself.

# Goals

- Stores assets on the [Gnosis multisig](https://github.com/gnosis/MultiSigWallet), which is the most proven smart contract for storing assets, while still having the flexibility of an Aragon entity
- Allows for painless transfers with hot or warm keys, to reduce possibility of losing cold keys and the effort it takes to safeguard them
- Lets the beneficiary immediately access funds if needed, with extra steps. By default, implements a time delay
- Allows for the beneficiary to define heirs that can withdraw assets collectively (dead's man switch). They can also become a recovery mechanism

# Governance structure

*Note: support percentages and time delays can be tweaked as you want, these are just sensible defaults*



Aragon Trusts are composed of two sub-groups:

- Beneficiary (token symbol HOLD)
- Heirs (token symbol HEIR)



### Multisig

- Signers:
  - Cold key
  - Warm key
  - Holdings DAO
  
  

### Holdings DAO

- **Token (HOLD)**: This token would be given to

  - One hot key
  - One warm key

- **Voting (HOLD)**: All keys can move funds immediately, one can move funds with one week delay, one can veto the transfer
  
  - Support: 100%
  - Quorum: 0%
  - Duration: One week
  
- **Token (HEIRS)**: Arbitrary number of tokens
  
  - 34% of those tokens are burnt. This is so the beneficiary voting no can make the vote don't pass. If the beneficiary has passed away, then 34% of the HEIRS holders would have >50% of the voting power, and they could pass a vote
  
- **Voting (HEIRS)**: All transfers have 1 year delay and the beneficiary can revoke the permission
  - Support: 66%
  - Quorum: 0%
  - Duration: One year
  
  

### Expenses DAO

- **Token (SPEND)**: Three hot keys
- **Voting (SPEND)**: 2/3 can move funds immediately, one can move funds with one day delay
  - Support: 50%
  - Quorum: 0%
  - Duration: One day



| App                   | Permission     | Grantee              | Permission manager   |
| --------------------- | -------------- | -------------------- | -------------------- |
| Kernel (expenses DAO) | Set app        | Agent (holdings DAO) | Agent (holdings DAO) |
| Tokens (SPEND)        | Mint tokens    | Agent (holdings DAO) | Agent (holdings DAO) |
| Tokens (SPEND)        | Burn tokens    | Agent (holdings DAO) | Agent (holdings DAO) |
| Agent (holdings DAO)  | Execute action | Voting (HOLD)        | Voting (HOLD)        |
| Agent (holdings DAO)  | Execute action | Voting (HEIRS)       | Voting (HOLD)        |



# Threat model

- The 2 multisig keys cannot get lost or stolen at the same time
- The DAO cannot break while also one multisig key gets lost or stolen
- The beneficiary needs to revoke the *Execute action* permission from the heirs if they create a vote when the beneficiary is still alive
- The warm key needs to be transmitted to the heirs in some way. You can even pre-sign a transaction that moves the funds to another address (e.g. a DAO composed of the people of your choice) and send your heirs a [Shamir Secret](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) that they can put together to execute that transfer
- Always initiate transactions to the multisig with the warm key and confirm with the DAO. Otherwise, attackers can predict that the vote will take one week and go to you the exact moment that you would need to sign the confirmation with the warm key
- One of the hot keys in the holding DAO needs to be warm so it takes you some effort to sign with it, in order to [prevent wrench attacks](https://xkcd.com/538/)

# Tutorial (WIP)

Deploy a democracy DAO

Install Agent

```
$ dao install {ORG_NAME}.aragonid.eth agent --environment aragon:mainnet
```

Obtain Agent app address

```
$ dao apps {ORG_NAME} --environment aragon:mainnet
```

Create permission for the Agent app to

```
$ dao acl create {ORG_NAME} {APP_ADDRESS} EXECUTE_ROLE {COLD_DAO} {COLD_DAO}
```

Give permission to Agent to mint and burn tokens on the expenses DAO

```
$ dao act {AGENT_ADDRESS} {H0T_DAO_TOKEN_MANAGER} "mint(address,uint256)" {RECEIVER} {AMOUNT * 1000000000000000000} --use-frame --environment aragon:mainnet

$ dao act {AGENT_ADDRESS} {H0T_DAO_TOKEN_MANAGER} "burn(address,uint256)" {RECEIVER} {AMOUNT * 1000000000000000000} --use-frame --environment aragon:mainnet
```

Revoke permission to the expenses DAO voting app to mint and burn tokens on the expenses DAO

```
$ dao act {AGENT_ADDRESS} {H0T_DAO_ACL} "revokePermission(address,address,bytes32)" {HOT_DAO_VOTING} {H0T_DAO_ACL} keccak256(MINT_ROLE) --use-frame --environment aragon:mainnet

$ dao act {AGENT_ADDRESS} {H0T_DAO_ACL} "revokePermission(address,address,bytes32)" {HOT_DAO_VOTING} {H0T_DAO_ACL} keccak256(BURN_ROLE) --use-frame --environment aragon:mainnet
```
