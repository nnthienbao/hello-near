# Hello world app with NEAR

## **I. Introduce**
- This tutorial will help you create a simple application built on top of NEAR. Through this tutorial, you will create a simple smart contract called Hello World. The smart contract will take as input your name and return a greeting. Let's go

## **II. Knowledge required**
- Smart contracts on NEAR are written in RUST, so you need a basic knowledge of RUST. If you don't know RUST, that's okay because NEAR also supports writing smart contracts on AssemblyScript similar to JavaScript
- For the frontend, NEAR supports writing on React or pure JavaScript. Therefore, basic knowledge of React or JavaScript is required

## **III. Overview**
- Applications on NEAR have two distinct parts – a back-end and front-end:
  - Smart Contract (back-end): Storing and modifying data on the chain. Contracts need to expose methods that allow clients to “view” and “change” state.
  - Interactions with Smart Contract (front-end): You can interact with your contracts or contracts deployed by other people. You do this by using near-api-js Quickstart and Code Snippets in your application.


## **IV. Setup and build**
### **1) Setup**
- We need to create a project to get started. Creating a project on NEAR is very easy with the create-near-app package. You just need to enter the following code in the command line and wait for the project to be created
  - `npx create-near-app [options] [project-name]`
- Options:
  - --frontend=react – use React for your frontend template
  - --contract=rust – use Rust for your smart contract
- Reference: https://github.com/near/create-near-app
### **2) Smart contract code**
- The entire code of the smart contract is located in the directory /contract/src/lib.rs
- You may be surprised by the code here. Don't worry, delete it and add the following code:
  ``` rust
    use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
    use near_sdk::{near_bindgen, setup_alloc};

    setup_alloc!();

    #[near_bindgen]
    #[derive(BorshDeserialize, BorshSerialize)]
    pub struct Welcome {
    }

    impl Default for Welcome {
    fn default() -> Self {
        Self {
        
        }
    }
    }

    #[near_bindgen]
    impl Welcome {
        pub fn get_hello(&self, name: String) -> String {
            let ret = format!("Hello {}", name);
            ret
        }
    }
  ```
- **Analysis**
  - The #[near_bindgen] macro is used on a struct and the function implementations to generate the necessary code to be a valid NEAR contract and expose the intended functions to be able to be called externally
  - Here there is a public method named 'get_hello' which takes a parameter 'name' so that outsiders can interact with our smart contract. This method implements taking a name and returning the message Hello [your name]
  
### **3) Front-end code**
- The entire code of the front-end is located in the directory /src
- **/src/config.js - Network Connection**
  ``` js
  const CONTRACT_NAME = process.env.CONTRACT_NAME ||'hello-smart-contract'
        function getConfig(env) {
        switch (env) {
        case 'production':
        case 'mainnet':
            return {
            networkId: 'mainnet',
            nodeUrl: 'https://rpc.mainnet.near.org',
            contractName: CONTRACT_NAME,
            walletUrl: 'https://wallet.near.org',
            helperUrl: 'https://helper.mainnet.near.org',
            explorerUrl: 'https://explorer.mainnet.near.org',
            }
        case 'development':
        case 'testnet':
            return {
            networkId: 'testnet',
            nodeUrl: 'https://rpc.testnet.near.org',
            contractName: CONTRACT_NAME,
            walletUrl: 'https://wallet.testnet.near.org',
            helperUrl: 'https://helper.testnet.near.org',
            explorerUrl: 'https://explorer.testnet.near.org',
            }
        case 'betanet':
            return {
            networkId: 'betanet',
            nodeUrl: 'https://rpc.betanet.near.org',
            contractName: CONTRACT_NAME,
            walletUrl: 'https://wallet.betanet.near.org',
            helperUrl: 'https://helper.betanet.near.org',
            explorerUrl: 'https://explorer.betanet.near.org',
            }
        case 'local':
            return {
            networkId: 'local',
            nodeUrl: 'http://localhost:3030',
            keyPath: `${process.env.HOME}/.near/validator_key.json`,
            walletUrl: 'http://localhost:4000/wallet',
            contractName: CONTRACT_NAME,
            }
        case 'test':
        case 'ci':
            return {
            networkId: 'shared-test',
            nodeUrl: 'https://rpc.ci-testnet.near.org',
            contractName: CONTRACT_NAME,
            masterAccount: 'test.near',
            }
        case 'ci-betanet':
            return {
            networkId: 'shared-test-staging',
            nodeUrl: 'https://rpc.ci-betanet.near.org',
            contractName: CONTRACT_NAME,
            masterAccount: 'test.near',
            }
        default:
            throw Error(`Unconfigured environment '${env}'. Can be configured in src/config.js.`)
        }
    }
    module.exports = getConfig
  ```
  - **Analysis**
    - The code above defines the data and endpoints required to connect to the NEAR network. The connection information defined here is included for MainNet, TestNet and BetaNet as well as the default LocalNet configuration
  
- **/src/utils.js - Configuration**
  ``` js
    import { connect, Contract, keyStores, WalletConnection } from 'near-api-js'
    import getConfig from './config'

    const nearConfig = getConfig(process.env.NODE_ENV || 'development')

    // Initialize contract & set global variables
    export async function initContract() {
    // Initialize connection to the NEAR testnet
    const near = await connect(Object.assign({ deps: { keyStore: new keyStores.BrowserLocalStorageKeyStore() } }, nearConfig))

    // Initializing Wallet based Account. It can work with NEAR testnet wallet that
    // is hosted at https://wallet.testnet.near.org
    window.walletConnection = new WalletConnection(near)

    // Getting the Account ID. If still unauthorized, it's just empty string
    window.accountId = window.walletConnection.getAccountId()

    // Initializing our contract APIs by contract name and configuration
    window.contract = await new Contract(window.walletConnection.account(), nearConfig.contractName, {
        // View methods are read only. They don't modify the state, but usually return some value.
        viewMethods: ['get_hello'],
        // Change methods can modify the state. But you don't receive the returned value when called.
        // changeMethods: ['set_greeting'],
    })
    }

    export function logout() {
    window.walletConnection.signOut()
    // reload page
    window.location.replace(window.location.origin + window.location.pathname)
    }

    export function login() {
    // Allow the current app to make calls to the specified contract on the
    // user's behalf.
    // This works by creating a new access key for the user's account and storing
    // the private key in localStorage.
    window.walletConnection.requestSignIn(nearConfig.contractName)
    }
  ```
    - **Analysis**
      - This is where you configure the connection to the NEAR network. You can configure the contract interface by inserting a wallet connection and connecting both contract methods.

### **4) Build and Deploy smart contract**
- Step 1: Install near-cli
  - ```
        npm install --global near-cli
    ```
    - Check near-cli install success
    ```
        near --version
    ```
- Step 2: Create an account for the contract
  - Visit NEAR Wallet and make a new account. You’ll be deploying these smart contracts to this new account
  - Enter command line to access to your NEAR account on cli
    ```
        near login
    ```
- Step 3: Build
  - Enter command
    ```
        npm run build
    ```
  - After build. You can see the smart contract is located in the /out/main.wasm directory and the front-end is in the /dist directory
- Step 4: Deploy
  - Enter the following command, the smart contract is deployed to the account you logged in in step 2
    ```
        npm run deploy
    ```

## **V. References**
- https://www.near-sdk.io/
- https://github.com/near/create-near-app
- https://www.rust-lang.org/