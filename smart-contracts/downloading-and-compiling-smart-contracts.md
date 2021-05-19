# Downloading and Compiling Smart Contracts

In this section, we will download a sample contract, compile to it to wasm binary executable.

Please first review the [client setup instructions](https://docs.cosmwasm.com/0.13/getting-started/setting-env.html), and configure and verify a client, either Go CLI or Node.js console.

### Compiling and Testing Contract <a id="compiling-and-testing-contract"></a>

Let's download the repo in which we collect [`cosmwasm-examples` \(opens new window\)](https://github.com/CosmWasm/cosmwasm-examples)and try out an existing simple escrow contract that can hold some native tokens and gives the power to an arbiter to release them to a pre-defined beneficiary. First, clone the repo and try to build the wasm bundle:

```bash
# get the code
git clone https://github.com/CosmWasm/cosmwasm-examples
cd cosmwasm-examples
git fetch
git checkout escrow-0.7.0
cd escrow

# compile the wasm contract with stable toolchain
rustup default stable
cargo wasm
```

After this compiles, it should produce a file in `target/wasm32-unknown-unknown/release/cw_escrow.wasm`. A quick `ls -l` should show around 2MB. This is a release build, but not stripped of all unneeded code. To produce a much smaller version, you can run this which tells the compiler to strip all unused code out:Copy  


```bash
RUSTFLAGS='-C link-arg=-s' cargo wasm
```

This produces a file about 174kB. We use this and another optimiser in the next [last section](https://docs.cosmwasm.com/0.13/getting-started/compile-contract.html#Optimized-Compilation) to produce the final product uploaded to the blockchain. You don't need to worry about running this yourself \(unless you are curious\), but you should have an idea of the final size of your contract this way.

### Unit Tests <a id="unit-tests"></a>

Let's try running the unit tests:

```bash
RUST_BACKTRACE=1 cargo unit-test
```

After some compilation steps, you should see:

```bash
running 5 tests
test contract::tests::cannot_initialize_expired ... ok
test contract::tests::proper_initialization ... ok
test contract::tests::init_and_query ... ok
test contract::tests::handle_refund ... ok
test contract::tests::handle_approve ... ok

test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

`RUST_BACKTRACE=1` will provide you with full stack traces on any error, which is super useful. This only works for unit tests \(which test native rust code, not the compiled wasm\). Also, if you want to know where `cargo wasm` and `cargo unit-test` come from, they are just aliases defined in `.cargo/config`. Take a look there to understand the cargo flags more.

### Optimized Compilation <a id="optimized-compilation"></a>

Smart contract binary size must be as small as possible for reduced gas cost. This will not only cost less on deployment, also for every single interaction. Simply, **optimize production code** using [cosmwasm/rust-optimizer \(opens new window\)](https://github.com/CosmWasm/rust-optimizer). **rust-optimizer** also produces reproducible builds of cosmwasm smart contracts. This means third parties can verify the contract is the actually the claimed code.

```bash
Optimized Compilation
Smart contract binary size must be as small as possible for reduced gas cost. This will not only cost less on deployment, also for every single interaction. Simply, optimize production code using cosmwasm/rust-optimizer (opens new window). rust-optimizer also produces reproducible builds of cosmwasm smart contracts. This means third parties can verify the contract is the actually the claimed code.

Copy
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/rust-optimizer:0.10.7
```
