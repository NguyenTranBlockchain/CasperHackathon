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

### Notes
* Had some issues with following the instructions on [CasperLabs Docs on Getting Started](https://docs.casperlabs.io/en/latest/dapp-dev-guide/setup-of-rust-contract-sdk.html) as it referenced v1.3.2 and I was running client v1.3.3. Found some help on Discord where instructions were given to use `make prepare` `make build-contract` and `make test` instead.
* Also had issues with running `make test` with below output. Euan on Discord helped to troubleshoot and turns out the issue was my project was built inside a shared folder (using Ubuntu 20.04 on Virtualbox on W10). The lesson there is, don't build your project on a shared folder!
```
running 2 tests
test tests::should_error_on_missing_runtime_arg - should panic ... FAILED
test tests::should_store_hello_world ... FAILED

failures:

---- tests::should_error_on_missing_runtime_arg stdout ----
thread 'main' panicked at 'Expected successful execution result, but instead got: [
    Failure {
        error: WasmPreprocessing(
            Deserialize(
                "Invalid magic number at start of file",
            ),
        ),
        effect: ExecutionEffect {
            ops: {},
            transforms: {},
        },
        transfers: [],
        cost: Gas(
            0,
        ),
    },
]',
```
 
## Complete one of the existing tutorials for writing smart contracts
### Output
![](https://i.imgur.com/zIu0wXf.png)


### Notes
* Some trouble at trying to run `casper-client get-state-root-hash --node-address http://localhost:11101` on the [Counter Walkthrough](https://docs.casperlabs.io/en/latest/dapp-dev-guide/tutorials/counter/walkthrough.html), but found someone had same issue on Discord and Maciej suggested to do the following, which seemed to work for me:
```
1. Open this link: https://repo.casperlabs.io/
2. Use the instruction to add our repo to your apt dependencies (assuming your using ubuntu).
3. sudo apt install casper-client
``` 

* Some more trouble following tutorial, this time on the following. Turns out `[ACCOUNT_HASH]` is actually asking for the `faucet a/c key` not the `faucet a/c hash` (although the fact it's asking for the account **hash** does confuse things)
```
casper-client query-state \
    --node-address http://localhost:11101 \
    --state-root-hash [STATE_ROOT_HASH] \
    --key [ACCOUNT_HASH]
```

* Yes, im an amateur :) more issues. this time i get the following when trying to `casper-client put-deploy`, and it's because im following the tutorial and after cloning the *counter* repo, it doesnt tell you to `cd` into it. After `cd counter`,`make prepare`, `make test` and repeating `casper-client put-deploy` ... we're golden (for now lol, a lot to learn here)
* interesting observation in the end was that when i was calling the contract to increase counter, it was successful but the `state_root_hash` didn't change twice, and the `-q "counter/count"` was coming back with zero, but trying it a second time the counter came back as 2. What happened first time? :man-shrugging: 

## Demonstrate key management concepts by modifying the client in the Multi-Sig tutorial to address one of the additional scenarios
I will be running Additional Scenario 5: managing accounts with multiple keys 
### Output
Add my scenario to package.json
![](https://i.imgur.com/LeqO8U6.png)
Create new script for scenario-four
![](https://i.imgur.com/XTTvi2v.png)
Modify the code to match the scenario
![](https://i.imgur.com/1M2U7cP.png)

Run the script
![](https://i.imgur.com/FZGCF9t.png)
![](https://i.imgur.com/8FSgQvc.png)



### Notes
* Surprisingly a lot more straight forward than earlier parts

## Learn to transfer tokens to an account on the Casper Testnet. 
### Output
![](https://i.imgur.com/23kphmg.png)

`deploy_hash": "ed9db2b202edd6bf5e387a80479bc1d36b684792300743008a0a9126513db01a`

![](https://i.imgur.com/nqkaQWu.png)

### Notes
* The instructions didn't specify that we needed to put `--payment-amount 10000` and was getting an error as per below.

![](https://i.imgur.com/4P4ZhVi.png)



## Learn to Delegate and Undelegate on the Casper Testnet. 
### Output
#### Delegating
![](https://i.imgur.com/NeqxEfD.png)
![](https://i.imgur.com/P46tTld.png)
![](https://i.imgur.com/wDTtW5b.png)
![](https://i.imgur.com/0NkLEVc.png)
![](https://i.imgur.com/Wh5PNSl.png)
![](https://i.imgur.com/l7WJb6i.png)
![](https://i.imgur.com/TbaXGvR.png)

#### Undelegating
![](https://i.imgur.com/2jMQ26f.png)
![](https://i.imgur.com/Ji1mA0k.png)
![](https://i.imgur.com/8ugOvpb.png)
![](https://i.imgur.com/8kjbo0F.png)
![](https://i.imgur.com/RJzJu3X.png)
![](https://i.imgur.com/tJ11PGL.png)
