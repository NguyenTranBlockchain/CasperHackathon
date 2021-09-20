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

