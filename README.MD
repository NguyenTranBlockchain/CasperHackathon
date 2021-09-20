# Get Started with Casper

## Setup Environment, developer tools:
```
apt update
mkdir working_space
cd working_space
git clone https://github.com/casper-network/casper-node.git
git clone https://github.com/casper-network/casper-node-launcher.git
```
![](/images/0.0.png)

```
apt install curl
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
cargo --version
```
![](/images/0.1.png)

```
python3 -m venv env
source env/bin/activate

pip install --upgrade pip
pip install jq
pip install supervisor
pip install toml

cd casper-node
make setup-rs
cargo build -p casper-node --release
```
![](/images/0.2.png)

```
source utils/nctl/activate
nctl-compile
```
![](/images/0.3.png)
![](/images/0.4.png)

```
nctl-assets-setup && nctl-start
```

![](/images/0.5.png)


### Install casper-cli
```
cargo +nightly-2021-06-17 install casper-client --locked
casper-client --version
```
![](/images/0.6.png)

### Installing the Casper Crates
```
cargo install cargo-casper
```
![](/images/0.7.png)
## Create and deploy a simple, smart contract with cargo casper and cargo test
```
cargo casper my-project
cd my-project
rustup install $(cat rust-toolchain)
rustup target add --toolchain $(cat rust-toolchain) wasm32-unknown-unknown

cd contract
cargo build --release
```

![](/images/1.0.png)

```
cd ..
make test
```
#### You should see the result like this:

![](/images/1.1.png)

#### Deploy contract:

```
casper-client put-deploy \
    --chain-name casper-net-1 \
    --node-address http://localhost:11101 \
    --session-path /working_space/my-project/contract/target/wasm32-unknown-unknown/release/contract.wasm \
    --secret-key /working_space/casper-node/utils/nctl/assets/net-1/faucet/secret_key.pem \
    --payment-amount 5000000000000 
```

![](/images/1.2.png)

##### Check deploy status:

```
casper-client get-deploy --node-address http://localhost:11101 b714253d77fad534dabe2229431c7ddd50fc26ce7801b2a87dc702989fd70b6c
```

![](/images/1.3.png)


##  Complete one of the existing tutorials for writing smart contracts
### A Counter Contract Tutorial 

```
git clone https://github.com/casper-ecosystem/counter
cd counter
```

```
casper-client get-state-root-hash --node-address http://localhost:11101
```

![](/images/2.0.png)

### Get account hash
![](/images/2.1.png)

```
casper-client query-state \
    --node-address http://localhost:11101 \
    --state-root-hash d07877bfffdf12265aa13173232392827be0983ca400ffb5286ca365cc9efa7f \
    --key account-hash-bc649423502a04c91c10713ead12868918b9d16961029d96f0d44298c79354d0 
```
![](/images/2.2.png)

```
make prepare
make test
```

![](/images/2.3.png)

### Deploy
```
casper-client put-deploy \
    --node-address http://localhost:11101 \
    --chain-name casper-net-1 \
    --secret-key /working_space/casper-node/utils/nctl/assets/net-1/faucet/secret_key.pem \
    --payment-amount 5000000000000 \
    --session-path ./target/wasm32-unknown-unknown/release/counter-define.wasm
```
![](/images/2.4.png)

```
casper-client get-state-root-hash --node-address http://localhost:11101
casper-client query-state \
	--node-address http://localhost:11101 \
	--state-root-hash 4f4087aadac822e7c95c017a5b33bd334668c94405ff48e4bc72eb33864e376e \
	--key account-hash-bc649423502a04c91c10713ead12868918b9d16961029d96f0d44298c79354d0 \
	-q counter/count
```

![](/images/2.5.png)

### Increase number
```
casper-client put-deploy \
    --node-address http://localhost:11101 \
    --chain-name casper-net-1 \
    --secret-key /working_space/casper-node/utils/nctl/assets/net-1/faucet/secret_key.pem \
    --payment-amount 5000000000000 \
    --session-name "counter" \
    --session-entry-point "counter_inc"
```

![](/images/2.6.png)


### Check status
```
casper-client get-state-root-hash --node-address http://localhost:11101
casper-client query-state \
	--node-address http://localhost:11101 \
	--state-root-hash 6117866daa9274f922c0ee652b816c30aba7a164905341a7855aa5ee009950f3 \
	--key account-hash-bc649423502a04c91c10713ead12868918b9d16961029d96f0d44298c79354d0 \
	-q counter/count
```

![](/images/2.7.png)


## Demonstrate key management concepts by modifying the client in the Multi-Sig tutorial to address one of the additional scenarios
```
git clone https://github.com/casper-ecosystem/keys-manager
cd keys-manager

rustup install $(cat rust-toolchain)
rustup target add --toolchain $(cat rust-toolchain) wasm32-unknown-unknown
cd contract
cargo build --release
```

![](/images/3.0.png)
![](/images/3.1.png)

```
cd keys-manager/client/
touch .env
vim .env
```
Put content to .env file
```
BASE_KEY_PATH=/working_space/casper-node/utils/nctl/assets/net-1/faucet/
NODE_URL=http://localhost:11101/rpc
```

![](/images/3.2.png)

```
npm i
touch second.js
```

Add command to package.json
```
"start:second": "node -r dotenv/config ./src/second.js"
```

Add content to second.js
```
const keyManager = require('./key-manager');
const TRANSFER_AMOUNT = process.env.TRANSFER_AMOUNT || 2500000000;

(async function () {
    let deploy;
  
    let masterKey = keyManager.randomMasterKey();
    let masterAcc = masterKey.deriveIndex(1);    
    let deployAcc = masterKey.deriveIndex(2);    

    console.log("\n0.1 Fund main account.\n");
    await keyManager.fundAccount(masterAcc);
    await keyManager.printAccount(masterAcc);
    
    console.log("\n[x]0.2 Install Keys Manager contract");
    deploy = keyManager.keys.buildContractInstallDeploy(masterAcc);
    await keyManager.sendDeploy(deploy, [masterAcc]);
    await keyManager.printAccount(masterAcc);

    // 1. Set masterAcc's weight to 2
    console.log("\n1. Set faucet's weight to 2\n");
    deploy = keyManager.keys.setKeyWeightDeploy(masterAcc, masterAcc, 2);
    await keyManager.sendDeploy(deploy, [masterAcc]);
    await keyManager.printAccount(masterAcc);
    
    // 2. Set Keys Management Threshold to 2.
    console.log("\n2. Set Keys Management Threshold to 2\n");
    deploy = keyManager.keys.setKeyManagementThresholdDeploy(masterAcc, 2);
    await keyManager.sendDeploy(deploy, [masterAcc]);
    await keyManager.printAccount(masterAcc);
    
    // 3. Set Deploy Threshold to 1.
    console.log("\n3. Set Deploy Threshold to 1.\n");
    deploy = keyManager.keys.setDeploymentThresholdDeploy(masterAcc, 1);
    await keyManager.sendDeploy(deploy, [masterAcc]);
    await keyManager.printAccount(masterAcc);
    
    // 4. Add first new key with weight 1 (only deployAccount).
    console.log("\n4. Add first new key with weight 1.\n");
    deploy = keyManager.keys.setKeyWeightDeploy(masterAcc, deployAcc, 1);
    await keyManager.sendDeploy(deploy, [masterAcc]);
    await keyManager.printAccount(masterAcc);
})();
```

### Run command:
```
npm run start:second
```

### Result:
![](/images/3.3.png)
```
0.1 Fund main account.

Signed by: account-hash-bc649423502a04c91c10713ead12868918b9d16961029d96f0d44298c79354d0
Deploy hash: bcb6edc282635f19ddc7ebde38a669841bd8078d5172dc608fb2a0fdab3179c5
Deploy result:
{ deploy:
   { hash:
      'bcb6edc282635f19ddc7ebde38a669841bd8078d5172dc608fb2a0fdab3179c5',
     header:
      { account:
         '01618c8dda5d508dc541ac961675d0517aa73c1872534e96bc33d8776eb055389d',
        timestamp: '2021-09-20T07:20:20.851Z',
        ttl: '30m',
        gas_price: 1,
        body_hash:
         '0ffdc3408c31f921323d5c0434ad2ebf9579b95e367c26ebcacdf762f24a60dc',
        dependencies: [],
        chain_name: 'casper-net-1' },
     payment: { ModuleBytes: [Object] },
     session: { Transfer: [Object] },
     approvals: [ [Object] ] } }

[x] Current state of the account:
{ _accountHash:
   'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
  namedKeys: [],
  mainPurse:
   'uref-efb01318301ca6e695eb41ab1399101b1aab729d72660e54b570acd7aa0596a0-007',
  associatedKeys:
   [ { accountHash:
        'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
       weight: 1 } ],
  actionThresholds: { deployment: 1, keyManagement: 1 } }

[x]0.2 Install Keys Manager contract
Signed by: account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631
Deploy hash: 105d714893e6d9b7f5fd7dde5a7cb39ecfb59aff97d3274921b7d83faf8e0d84
Deploy result:
{ deploy:
   { hash:
      '105d714893e6d9b7f5fd7dde5a7cb39ecfb59aff97d3274921b7d83faf8e0d84',
     header:
      { account:
         '02021f71dc7f185e5e5756756d7d170858098e226b76adc87d9e7ca98ad87fcfd7d2',
        timestamp: '2021-09-20T07:21:26.390Z',
        ttl: '30m',
        gas_price: 1,
        body_hash:
         '03eb697e9a28602a621ca213a40e47b6a73c98300dd98957bcee6716d071d59c',
        dependencies: [],
        chain_name: 'casper-net-1' },
     payment: { ModuleBytes: [Object] },
     session: { ModuleBytes: [Object] },
     approvals: [ [Object] ] } }

[x] Current state of the account:
{ _accountHash:
   'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
  namedKeys:
   [ { name: 'keys_manager',
       key:
        'hash-35a3bdab99b9fc13e8416e154846e7922af245f40b75eb905151c8968f233a98' },
     { name: 'keys_manager_hash',
       key:
        'uref-5c9b5f39437bfb93201cc87095dc5e33e77f3c4fdcffb2131dbeabeef068288f-007' } ],
  mainPurse:
   'uref-efb01318301ca6e695eb41ab1399101b1aab729d72660e54b570acd7aa0596a0-007',
  associatedKeys:
   [ { accountHash:
        'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
       weight: 1 } ],
  actionThresholds: { deployment: 1, keyManagement: 1 } }

1. Set faucet's weight to 2

Signed by: account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631
Deploy hash: 45dddaed7f9ece6afeff08dbb25cfb0ba9ef09c91d41db26a80f85bffe65e6b3
Deploy result:
{ deploy:
   { hash:
      '45dddaed7f9ece6afeff08dbb25cfb0ba9ef09c91d41db26a80f85bffe65e6b3',
     header:
      { account:
         '02021f71dc7f185e5e5756756d7d170858098e226b76adc87d9e7ca98ad87fcfd7d2',
        timestamp: '2021-09-20T07:22:32.098Z',
        ttl: '30m',
        gas_price: 1,
        body_hash:
         'ad0747d115e31d7c8676ac3ab4d9d17a12ba8bbc1b844fe90d83f69b58023827',
        dependencies: [],
        chain_name: 'casper-net-1' },
     payment: { ModuleBytes: [Object] },
     session: { StoredContractByName: [Object] },
     approvals: [ [Object] ] } }

[x] Current state of the account:
{ _accountHash:
   'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
  namedKeys:
   [ { name: 'keys_manager',
       key:
        'hash-35a3bdab99b9fc13e8416e154846e7922af245f40b75eb905151c8968f233a98' },
     { name: 'keys_manager_hash',
       key:
        'uref-5c9b5f39437bfb93201cc87095dc5e33e77f3c4fdcffb2131dbeabeef068288f-007' } ],
  mainPurse:
   'uref-efb01318301ca6e695eb41ab1399101b1aab729d72660e54b570acd7aa0596a0-007',
  associatedKeys:
   [ { accountHash:
        'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
       weight: 2 } ],
  actionThresholds: { deployment: 1, keyManagement: 1 } }

2. Set Keys Management Threshold to 2

Signed by: account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631
Deploy hash: d080d42405073d92076b916bdd05ca9d3c426573de70f9bd5dc53defc5249b65
Deploy result:
{ deploy:
   { hash:
      'd080d42405073d92076b916bdd05ca9d3c426573de70f9bd5dc53defc5249b65',
     header:
      { account:
         '02021f71dc7f185e5e5756756d7d170858098e226b76adc87d9e7ca98ad87fcfd7d2',
        timestamp: '2021-09-20T07:23:36.577Z',
        ttl: '30m',
        gas_price: 1,
        body_hash:
         '9357475174b44041d5a89a9091cded242fedc567af82c7fa920d5ec13d5c7f74',
        dependencies: [],
        chain_name: 'casper-net-1' },
     payment: { ModuleBytes: [Object] },
     session: { StoredContractByName: [Object] },
     approvals: [ [Object] ] } }

[x] Current state of the account:
{ _accountHash:
   'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
  namedKeys:
   [ { name: 'keys_manager',
       key:
        'hash-35a3bdab99b9fc13e8416e154846e7922af245f40b75eb905151c8968f233a98' },
     { name: 'keys_manager_hash',
       key:
        'uref-5c9b5f39437bfb93201cc87095dc5e33e77f3c4fdcffb2131dbeabeef068288f-007' } ],
  mainPurse:
   'uref-efb01318301ca6e695eb41ab1399101b1aab729d72660e54b570acd7aa0596a0-007',
  associatedKeys:
   [ { accountHash:
        'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
       weight: 2 } ],
  actionThresholds: { deployment: 1, keyManagement: 2 } }

3. Set Deploy Threshold to 1.

Signed by: account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631
Deploy hash: cf2a0ffb7bbf02fdbf96e00078e4aa5ada2c1a747781a5dd4aeb3a30d0bb0be0
Deploy result:
{ deploy:
   { hash:
      'cf2a0ffb7bbf02fdbf96e00078e4aa5ada2c1a747781a5dd4aeb3a30d0bb0be0',
     header:
      { account:
         '02021f71dc7f185e5e5756756d7d170858098e226b76adc87d9e7ca98ad87fcfd7d2',
        timestamp: '2021-09-20T07:24:42.962Z',
        ttl: '30m',
        gas_price: 1,
        body_hash:
         '4807b024a8e1e7c7d1da5a9101a21f38303d4689905d88d4585eec852fd7216b',
        dependencies: [],
        chain_name: 'casper-net-1' },
     payment: { ModuleBytes: [Object] },
     session: { StoredContractByName: [Object] },
     approvals: [ [Object] ] } }

[x] Current state of the account:
{ _accountHash:
   'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
  namedKeys:
   [ { name: 'keys_manager',
       key:
        'hash-35a3bdab99b9fc13e8416e154846e7922af245f40b75eb905151c8968f233a98' },
     { name: 'keys_manager_hash',
       key:
        'uref-5c9b5f39437bfb93201cc87095dc5e33e77f3c4fdcffb2131dbeabeef068288f-007' } ],
  mainPurse:
   'uref-efb01318301ca6e695eb41ab1399101b1aab729d72660e54b570acd7aa0596a0-007',
  associatedKeys:
   [ { accountHash:
        'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
       weight: 2 } ],
  actionThresholds: { deployment: 1, keyManagement: 2 } }

4. Add first new key with weight 1.

Signed by: account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631
Deploy hash: 04f19458d53aaece522c6581b5948a8f047cb2a3e4064c2418125c02163823a4
Deploy result:
{ deploy:
   { hash:
      '04f19458d53aaece522c6581b5948a8f047cb2a3e4064c2418125c02163823a4',
     header:
      { account:
         '02021f71dc7f185e5e5756756d7d170858098e226b76adc87d9e7ca98ad87fcfd7d2',
        timestamp: '2021-09-20T07:25:48.339Z',
        ttl: '30m',
        gas_price: 1,
        body_hash:
         '15f85b4f689b5873b83f3f8452d3c7bf445607bcbe31b295091ea598f1c95801',
        dependencies: [],
        chain_name: 'casper-net-1' },
     payment: { ModuleBytes: [Object] },
     session: { StoredContractByName: [Object] },
     approvals: [ [Object] ] } }

[x] Current state of the account:
{ _accountHash:
   'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
  namedKeys:
   [ { name: 'keys_manager',
       key:
        'hash-35a3bdab99b9fc13e8416e154846e7922af245f40b75eb905151c8968f233a98' },
     { name: 'keys_manager_hash',
       key:
        'uref-5c9b5f39437bfb93201cc87095dc5e33e77f3c4fdcffb2131dbeabeef068288f-007' } ],
  mainPurse:
   'uref-efb01318301ca6e695eb41ab1399101b1aab729d72660e54b570acd7aa0596a0-007',
  associatedKeys:
   [ { accountHash:
        'account-hash-c7b27fd1645ff6b015cb27021c2416d94c7d256142f8aff641cf99ccc9372631',
       weight: 2 },
     { accountHash:
        'account-hash-d1375b91cdd5bfb950cffaf3117e647da9d5bd789b35c8313293c3986f843d63',
       weight: 1 } ],
  actionThresholds: { deployment: 1, keyManagement: 2 } }
```

## Learn to transfer tokens to an account on the Casper Testnet
![](/images/4.0.png)
![](/images/4.1.png)
![](/images/4.2.png)

```
casper-client transfer \
    --node-address http://128.199.116.202:7777 \
    --secret-key secret_key.pem \
    --amount 2500000000 \
    --payment-amount 10000 \
    --target-account 017d96b9a63abcb61c870a4f55187a0a7ac24096bdb5fc585c12a686a4d892009e \
    --chain-name casper-test \
    --transfer-id 1999
```
![](/images/4.3.png)

![](/images/4.4.png)

## Learn to Delegate and Undelegate on the Casper Testnet
![](/images/5.0.png)
![](/images/5.1.png)
![](/images/5.2.png)
![](/images/5.3.png)

![](/images/5.4.png)
![](/images/5.5.png)
![](/images/5.6.png)
![](/images/5.7.png)
![](/images/5.8.png)