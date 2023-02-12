---
title: Coding comparison between Cosmwasm and Solidity
date: "2022-03-27T12:00:00.000Z"
template: "post"
draft: false
slug: "cosmwasm-compared-with-solidity"
category: "Web3"
tags:
  - "SmartContract"
  - "DApp"
  - "Terra"
description: "compare Cosmwasm with Solidity from a coding perspective"
socialImage: "/media/terra_luna.jpg"
---

![dapp](/media/dapp.jpg)

[Cosmwasm](https://docs.cosmwasm.com/docs/1.0/) is a smart contract platform.

I developed the fundraiser dApp on Terra blockchain with Cosmwasm.
[fundraiser-dapp-on-terra](https://github.com/Hiroki6/fundraiser-dapp-on-terra)

The fundraiser application is originally  written in the book below. The original one is developed in Solidity. The detail of this application is written in the book, and I highly recommend reading this book below if you are a beginner of smart contracts or DApp development.
[Hands-On Smart Contract Development with Solidity and Ethereum](https://learning.oreilly.com/library/view/hands-on-smart-contract/9781492045250/)

In the official Cosmwasm doc, there is [a comparison doc with Solidity](https://docs.cosmwasm.com/docs/0.14/architecture/smart-contracts/). This doc describes mainly a comparison from a concept and a security perspective.
On the other hand, I will compare the Cosmwasm with Solidity from a coding perspective based on the fundraiser application code in the article.

1. [State management](#1-state-management)
2. [Execute contracts](#2-execute-contracts)
3. [Execute another contract from contract](#3-execute-another-contract-from-contract)
4. [Standard functions/messages](#4-standard-functionslibraries)
5. [Call contract functions](#5-call-contract-functions)

## 1. State management
__Solidity__

In dApp, state modeling is one of the most important aspects. You need to decide which variables are stored on the blockchain or not.
In Solidity, state variables can be simply written as same as general programming. By using some keywords such as `memory` and `storage`, you can control whether variables are stored on the blockchain or not. Moreover, Solidity has some standard types such as `mapping` and `array` so that you can flexibly define your variables.

```sol
contract Fundraiser is Ownable {
    struct Donation {
        uint256 value;
        uint256 date;
    }
    mapping(address => Donation[]) private _donations;

    uint256 public totalDonations;
    uint256 public donationsCount;
}
```

These variables can be easily updated same as variables other programming languages have.

```sol
function donate() public payable {
    Donation memory donation = Donation({
        value: msg.value,
        date: block.timestamp
    });
    _donations[msg.sender].push(donation);
    totalDonations = totalDonations.add(msg.value);
    donationsCount++;

    emit DonationReceived(msg.sender, msg.value);
}
```


__Cosmwasm__

In Cosmwasm, you need to use storage objects when you want to store states in the blockchain. There are some storages such as `Map` and `Item` implemented in [cosmwasm-storage](https://github.com/CosmWasm/cosmwasm/tree/main/packages/storage) and [cw-storage-plus](https://github.com/CosmWasm/cw-plus/tree/main/packages/storage-plus).

Some explanations about storage are written [here](https://docs.cosmwasm.com/tutorials/storage/key-value-store). Compared with the above states of Fundraiser contract in Solidity, the ones in Cosmwasm are defined like below.

```rust
pub struct FundraiserContract<'a> {
    pub donation: Map<'a, Addr, Vec<Donation>>,
    pub total_donations: Item<'a, Uint128>,
}

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct Donation {
    pub value: Uint128,
    pub date: Timestamp,
}
```

An example advanced usage such as using the `IndexedMap` is written in [cw721-base](https://github.com/CosmWasm/cw-nfts/blob/main/contracts/cw721-base/src/state.rs).

These storage objects have some functions to update these.

```rust
self.total_donations
    .update(deps.storage, |total| -> Result<_, ContractError> {
        Ok(total + payment)
    })?;

self.donation.update(
    deps.storage,
    info.sender,
    |donation_opt: Option<Vec<Donation>>| -> Result<_, ContractError> {
        match donation_opt {
            Some(mut donation_map) => {
                donation_map.push(donation);
                Ok(donation_map)
            }
            None => Ok(vec![donation]),
        }
    },
)?;
```

## 2. Execute Contracts

__Solidity__

`constructor` enables to intialize Contract like other languages such as `Java`.
```sol
constructor(
    string memory _name,
    string memory _url,
    string memory _imageURL,
    string memory _description,
    address payable _beneficiary,
    address _custodian
) public {
    name = _name;
    url = _url;
    imageURL = _imageURL;
    description = _description;
    beneficiary = _beneficiary;
    transferOwnership(_custodian);
}
```
Functions in Solidity are categorised mainly into three.

1. only read data from the blockchain (`view` function)

Functions declared `view` can read the data from the blockchain but can't write data. Gas is not consumed when this function is called by external. But if this function is called by internal functions without the 'view' keyword, gas is consumed.
```sol
function myDonationsCount() public view returns(uint256) {
    return _donations[msg.sender].length;
}
```

2. normal functions which don't read and write any data from the blockchain (`pure` function)

Functions declared `pure` can read the data from the blockchain but can't write data. Gas is not consumed when this function is called, even if it's called by internal functions.

```sol
function _sum(uint a, uint b) private pure returns (uint) {
    return a + b;
}
```

3. write data in the blockchain

Functions without the `pure` or `view` can write data into blockchain.
```sol
function setBeneficiary(address payable _beneficiary) public onlyOwner {
    beneficiary = _beneficiary;
}
``` 

__Cosmwasm__

The actor model is the basic model in CosmWasm.

There are mainly three messages in Cosmwasm 
1. InstantiateMsg

This message is called when initialize contracts. It's same as contructor of Solidity

```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct InstantiateMsg {
    pub name: String,
    pub url: String,
    pub image_url: String,
    pub description: String,
    pub beneficiary: String,
    pub custodian: String,
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: InstantiateMsg,
) -> Result<Response, ContractError> {
    FundraiserContract::default().instantiate(deps, env, info, msg)
}
```

2. ExecuteMsg

This message is called when writes the data in the blockchain like functions without `pure` and `view` function of Solidity

```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum ExecuteMsg {
    SetBeneficiary { beneficiary: String },
    Donate {},
    Withdraw {},
}
```

```rust
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: ExecuteMsg,
) -> Result<Response, ContractError> {
    FundraiserContract::default().execute(deps, env, info, msg)
}

pub fn execute(
    &self,
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: ExecuteMsg,
) -> Result<Response, ContractError> {
    match msg {
        ExecuteMsg::SetBeneficiary { beneficiary } => {
            self.set_beneficiary(deps, info, beneficiary)
        }
        ExecuteMsg::Donate {} => self.donate(deps, env, info),
        ExecuteMsg::Withdraw {} => self.withdraw(deps, env, info),
    }
}
```

3. QueryMsg

This message is isent when reads the data from the blockchain like `view` function of Solidity.

```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    GetFundraiser {},
    DonationAmount { address: String },
    MyDonations { address: String },
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn query(deps: Deps, env: Env, msg: QueryMsg) -> StdResult<Binary> {
    FundraiserContract::default().query(deps, env, msg)
}

pub fn query(&self, deps: Deps, _env: Env, msg: QueryMsg) -> StdResult<Binary> {
    match msg {
        QueryMsg::GetFundraiser {} => to_binary(&self.query_fundraiser(deps)?),
        QueryMsg::DonationAmount { address } => {
            to_binary(&self.query_donation_amount(deps, address)?)
        }
        QueryMsg::MyDonations { address } => {
            to_binary(&self.query_my_donations(deps, address)?)
        }
    }
}
```

```rust
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    GetFundraiser {},
    DonationAmount { address: String },
    MyDonations { address: String },
}
```

## 3. Execute another contract from contract
Sometimes one contract calls another contract. In the fundraiser DApp, the `FundraiserFactory` contract calls the constructor of the `Fundraiser` contract to initialize it. 

__Solidity__

You can call another contract like calling other classes in Java.
```sol
contract FundraiserFactory {
    Fundraiser[] private _fundraisers;

    event FundraiserCreated(Fundraiser indexed fundraiser, address indexed owner);

    function createFundraiser(
        string memory name,
        string memory url,
        string memory imageURL,
        string memory description,
        address payable beneficiary
    ) public {
        Fundraiser fundraiser = new Fundraiser(
            name,
            url,
            imageURL,
            description,
            beneficiary,
            msg.sender
        );
        _fundraisers.push(fundraiser);
        emit FundraiserCreated(fundraiser, fundraiser.owner());
    }
}
```

__Cosmwasm__

As I said, Cosmos is based on the Actor model. So, the communication between contracts is also executed via messages.
In the fundraiser-dapp-on-terra, the `FundraiserFactory` contract creates a WasmMsg and adds it as `SubMessage` in the Response. This `SubMessage` will be called, and the Cosmos SDK ensures the transaction.

> A transaction may consist of multiple messages and each one is executed in turn under the same context and same gas limit. If all messages succeed, the context will be committed to the underlying blockchain state and the results of all messages will be stored in the TxResult. If one message fails, all later messages are skipped and all state changes are reverted. This is very important for atomicity. That means Alice and Bob can both sign a transaction with 2 messages: Alice pays Bob 1000 ATOM, Bob pays Alice 50 ETH, and if Bob doesn't have the funds in his account, Alice's payment will also be reverted. This is just like a DB Transaction typically works.
[sdk-context](https://github.com/CosmWasm/cosmwasm/blob/main/SEMANTICS.md#sdk-context)

```rust
let fundraiser_instantiate_msg = fundraiser::msg::InstantiateMsg {
     name: name.clone(),
     url,
     image_url,
     description,
     beneficiary,
     custodian: info.sender.to_string(),
 };

 Ok(Response::new()
     .add_attribute("action", "create_fundraiser")
     .add_attribute("name", name)
     .add_submessages(vec![SubMsg::reply_on_success(
         WasmMsg::Instantiate {
             code_id: config.fundraiser_code_id,
             funds: vec![],
             admin: None,
             label: "".to_string(),
             msg: to_binary(&fundraiser_instantiate_msg)?,
         },
         1,
     )]))
```

### Reply message

When you need to use reponses of calling `WasmMsg` , `reply` message enables it.

```rust
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn reply(deps: DepsMut, env: Env, msg: Reply) -> Result<Response, ContractError> {
    FundraiserFactoryContract::default().reply(deps, env, msg)
}

fn handle_instantiate_fundraiser(
    &self,
    deps: DepsMut,
    msg: Reply,
) -> Result<Response, ContractError> {
    let res: MsgInstantiateContractResponse = Message::parse_from_bytes(
        msg.result.unwrap().data.unwrap().as_slice(),
    )
    // do whatever you want with the res value
}
```

## 4. Standard functions/libraries
There are some useful standard functions and libraries in Solidity and CosmWasm.

__Solidity__

For example, `transfer` function enables to send the Ether to the address.

```sol
function withdraw() public onlyOwner {
    uint256 balance = address(this).balance;
    beneficiary.transfer(balance);
    emit Withdraw(balance);
}
```

[OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) is one of the most popular Solidity libraries. This library provides many useful base Contracts.

__Cosmwasm__

For example, the `BankMsg` enables to send amount to the address.

```rust
let send = BankMsg::Send {
    to_address: fundraiser.beneficiary.to_string(),
    amount,
};

Ok(Response::new()
    .add_message(send)
    .add_attribute("action", "with_draw")
    .add_attribute("owner", fundraiser.owner.to_string())
    .add_attribute("fundraiser", fundraiser.name))
```

[cw-plus](https://github.com/CosmWasm/cw-plus) is one of the most popular libraries for Cosmos SDK. This library provides many useful base Contracts, storages and funtions.

## 5. Call contract functions

The functions of Smart contract are called by external such as front-end application.

__Solidity__

[web3.js](https://web3js.readthedocs.io/en/v1.7.1/) is a Javascript API for Ethereum.

It enables to initialize an instance and execute a smart contract. You can call the function name as defined.

```js
await contract.methods.createFundraiser(
  name,
  url,
  imageURL,
  description,
  beneficiary
).send({ from: accounts[0] })
```
You can also call the value of the state directly from the frontend if this state is declared as `public` state.

```js
const totalDonations = await instance.methods.totalDonations().call();
```

__Cosmwasm__

[terra.js](https://terra-money.github.io/terra.js/) is a Javascript API for Terra.

Messages are sent as JSON.

```js
// ExecuteMsg
const msg = {
    set_beneficiary: {
        beneficiary: beneficiary,
    }
};

const result = await wallet.post({
     fee,
     msgs: [
         new MsgExecuteContract(
             wallet.walletAddress,
             contractAddress,
             msg,
             coins
         ),
     ],
     gas: 'auto'
 });

// QueryMsg
await lcd.wasm.contractQuery<FundraiserResponse>(contractAddress, { get_fundraiser: {}});
```

The schema of messages are created in the contract repository by running `cargo schema`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ExecuteMsg",
  "oneOf": [
    {
      "type": "object",
      "required": [
        "set_beneficiary"
      ],
      "properties": {
        "set_beneficiary": {
          "type": "object",
          "required": [
            "beneficiary"
          ],
          "properties": {
            "beneficiary": {
              "type": "string"
            }
          }
        }
      },
      "additionalProperties": false
    },
    ...
  ]
}
```

## Conclusion
In this article, I briefly compared the Cosmwasm with Solidity from a coding perspective based on the fundraiser application code in the article. There are more things that have to be taken into account when developing DApp, but I hope this article is helpful for developing DApp with CosmWasm.