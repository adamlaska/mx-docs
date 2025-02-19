---
id: sc-meta-cli
title: CLI
---

[comment]: # (mx-context-auto)

## Introduction

As mentioned [before](/developers/meta/sc-meta#standalone-tool-vs-contract-tool), the meta tool comes in two flavors, one for the individual contract (based on the contract's ABI), and the other as a standalone tool.

We will go through the CLI of both of these flavors.


[comment]: # (mx-exclude-context)

## Standalone tool CLI


[comment]: # (mx-context-auto)

### The `all` command

But first, there is a special feature that needs additional explanation: the `sc-meta all ...` command.

The standalone and the contract tools are not completely separate: the standalone tool can control multiple smart contract projects in a single command.

This is where the `all` command comes in: all it does is find all contracts in a given folder and call the contract tool for each of them with the given arguments.

For example:
- `sc-meta all abi` will generate the ABIs for all contracts;
- `sc-meta all build --locked` will build all contracts, with the crate versions given by Cargo.lock;
- `sc-meta all clean` will clean all projects;
- etc.

You can call `sc-meta all help` and see that the CLI docs are almost the same as those of the individual contract tool.

A related command is the `info` command, which just prints a tree with all the contract and contract libraries in a folder, without doing anything to them.

:::info Note
Since the CLI of the individual contract tool is available in the standalone tool, the following two commands are equivalent:

```
cd my-contract
sc-meta all build
```

```
cd my-contract/meta
cargo run build
```
:::

Paramameters:
- `--path`
    - Target directory where to call all contract meta crates.
    - _default_: current directory.
- `--ignore`
    - Ignore all directories with these names.
    - _default_: `target`.
- `--no-abi-git-version
    - Skips loading the Git version into the ABI
- `--target-dir-meta`
    - For the meta crates, allows specifying the target directory where the Rust compiler will build the intermediary files. Sharing the same target directory can speed up building multiple contract crates at once.
    - _default_: uses the workspace, or builds in `meta/target`.
- `--target-dir-all`
    - Overrides both the `--target-dir-meta` and the `--target-dir-wasm` args.


[comment]: # (mx-context-auto)

### Calling `info`

The info command prints an overview of the contracts and libraries and residing under a folder. It also prints their framework versions.

As an example, below is the output of calling it in the example contract folder in the framework:

![sc-meta info screenshot](/developers/sc-meta/sc-meta-info.png "Result of calling sc-meta info in the example contract folder in the framework")

Paramameters:
- `--path`
    - Target directory where to call all contract meta crates.
    - _default_: current directory.
- `--ignore`
    - Ignore all directories with these names.
    - _default_: `target`.


[comment]: # (mx-context-auto)

### Calling `upgrade`

Calling `sc-meta upgrade` will try to automatically alter the code of a contract or group of contracts to make it/them compatible with the latest rust framework version.

The oldest version currently supported is `0.28.0`. Any older than that, and the developer will need to manually upgrade it to `0.28.0`.

It is especially important when upgrading from `0.38` to `0.39.0`, since a lot of changes happened at that point.

:::tip
For projects with multiple contract crates, we recommend upgrading all of them at once. The upgrade algorithm goes step by step, version after version. For some of the major versions, it also checks that the project compiles before moving on. This is to give developers the chance to fix issues manually, if necessary, and not have those issues pile up. If there are local depdencies between contracts, the upgrader will not be able to do the check unless all of them are upgraded together.
:::

:::caution
Generally, we strongly recommend to ensure code versioning or at least a backup of the contract code to avoid the impossibility of reverting permanent changes. This automatic code altering process involved in using `sc-meta upgrade` highly raises this recommendation.
:::

Paramameters:
- `--path`
    - Target directory where to call all contract meta crates.
    - _default_: current directory.
- `--ignore`
    - Ignore all directories with these names.
    - _default_: `target`.
- `--to`
    - Overrides the version to upgrade to.
    - _default_: the last released version.

[comment]: # (mx-context-auto)

### Calling `local-deps`

Calling `sc-meta local-deps` will create in each contract a report of the local dependencies between contracts and libraries. This helps with the reproducible builds, but might be extended in the future for other uses.

Example output (abridged):
```json
{
    "root": "/home/user/multiversx/mx-exchange-sc/",
    "contractPath": "/home/user/multiversx/mx-exchange-sc/dex/pair",
    "commonDependencyPath": "/home/user/multiversx/mx-exchange-sc",
    "dependencies": [
        {
            "path": "common/common_errors",
            "depth": 1
        },
        {
            "path": "common/common_structs",
            "depth": 1
        },
        {
            "path": "common/modules/legacy_token_decode_module",
            "depth": 3
        },
        {
            "path": "common/modules/locking_module",
            "depth": 2
        },
        {
            "path": "common/modules/math",
            "depth": 2
        }
    ]
}
```

Paramameters:
- `--path`
    - Target directory where to call all contract meta crates.
    - _default_: current directory.
- `--ignore`
    - Ignore all directories with these names.
    - _default_: `target`.

[comment]: # (mx-context-auto)

### Calling `new`

<!-- TODO: expand section and move to a separate page -->

Creates a new smart contract project from a standard template.

The tool will replace all necessary names in the project, based on the the project name given by the user. These include:
- the crate name,
- the contract trait name,
- the file name of the main source file.

Paramameters:
- `--template`
    - The contract template to clone. Available options can be retrieve by using [this](/developers/meta/sc-meta-cli#calling-templates)
    - Required.
- `--name`
    - The new name the contract is to receive.
    - _default_: If missing, the template name will be kept.
- `--tag`
    -  The framework version on which the contracts should be created.
    - _default_: The latest released version.
- `--path`
    - Target directory where to create the new contract directory.
    - _default_: current directory.



[comment]: # (mx-context-auto)

### Calling `templates`

This command lists all available templates. As of framework version 0.43.2, they are:

```
crypto-zombies
empty
adder
```

Paramameter:
- `--tag`
    -  The framework version on which the contracts should be created.
    - _default_: The latest released version.


[comment]: # (mx-context-auto)

### Calling `test-gen`

<!-- TODO: expand section and move to a separate page -->

Contracts often have JSON scenario tests associated with them, which normally reside in the `scenarios` folder, under the contract crate root.

In order to execute them as part of the CI, it is helpful to generate a Rust test for each of them. The `test-gen` tool does just that.

These integration tests come in two flavors:
- Rust tests, that exclusively use the Rust debugger infrastructure;
- VM tests that use the Go infrastructure.

An example:

```rust title="adder/tests/adder_scenario_rs_test.rs"
use multiversx_sc_scenario::*;

fn world() -> ScenarioWorld {
    let mut blockchain = ScenarioWorld::new();
    blockchain.set_current_dir_from_workspace("contracts/examples/adder");

    blockchain.register_contract("file:output/adder.wasm", adder::ContractBuilder);
    blockchain
}

#[test]
fn adder_rs() {
    world().run("scenarios/adder.scen.json");
}

#[test]
fn interactor_trace_rs() {
    world().run("scenarios/interactor_trace.scen.json");
}
```

```rust title="adder/tests/adder_scenario_go_test.rs"
use multiversx_sc_scenario::*;

fn world() -> ScenarioWorld {
    ScenarioWorld::vm_go()
}

#[test]
fn adder_go() {
    world().run("scenarios/adder.scen.json");
}

#[test]
fn interactor_trace_go() {
    world().run("scenarios/interactor_trace.scen.json");
}
```

The `world()` definition is expected form the developer, but the tests themselves are generated and updated automatically when calling `sc-meta test-gen`.

:::caution
The tool does not work well with code that is commented-out. In order to temporarily disable a test, annotate it with `#[ignore]`.
:::

Paramameters:
- `--path`
    - Target directory where to call all contract meta crates.
    - _default_: current directory.
- `--ignore`
    - Ignore all directories with these names.
    - _default_: `target`.
- `--create`
    - Creates test files if they don't exist.



---

[comment]: # (mx-exclude-context)

## Individual contract CLI

[comment]: # (mx-context-auto)

### Calling `build`

A build can be triggered by calling either `sc-meta all build` or `cargo run build` in the meta crate of the contract. In fact, the standalone `sc-meta` tool simply forwards the command to the contract meta crate itself.

By default, this command will produce three files for each output contract: the ABI (`<contract>.abi.json`), the contract (`<contract>.wasm`) and a json file with all the used VM EI imported functions (`<contract>.imports.json`). For the multisig example above, the produced files are as follows:

```text
output
├── multisig-full.abi.json
├── multisig-full.imports.json
├── multisig-full.wasm
├── multisig-view.abi.json
├── multisig-view.imports.json
├── multisig-view.wasm
├── multisig.abi.json
├── multisig.imports.json
└── multisig.wasm
```

Arguments:

- `--locked` Uses the version from `Cargo.lock`, without updating. Required for reproducible builds.
- `--wasm-name` followed by name: Replaces the main contract's name with this one. Does nothing for secondary contracts.
- `--wasm-suffix` followed by a suffix: Adds a dash and this suffix to all produced contracts. E.g. `cargo run build --wasm-suffix dbg` on multisig will produce contracts `multisig-dbg.wasm`, `multisig-view-dbg.wasm` and `multisig-full-dbg.wasm`.
- `--wasm-symbols` Does not optimize away symbols at compile time, retains function names, good for investigating the WAT.
- `--no-wasm-opt` Does not apply `wasm-opt` after the build, this retains function names, good for investigating the WAT.
- `--wat` Also generates a WAT file for each of the contract outputs. It does so by calling `wasm2wat`.
- `--mir` Also emit MIR files when building.
- `--llvm-ir` Also emit LL (LLVM) files when building.
- `--no-abi-git-version` Skips loading the Git version into the ABI.
- `--no-imports` Does not generate an EI imports JSON file for each contract, as is the default.
- `--target-dir-wasm` Allows specifying the target directory where the Rust compiler will build the intermediary files. Sharing the same target directory can speed up building multiple contract crates at once.
- `--target-dir` Synonym of `--wasm-target-dir`, used for backwards compatibility.
- `--twiggy-top` Generate a twiggy top report after building.
- `--twiggy-paths` Generate a twiggy paths report after building.
- `--twiggy-monos` Generate a twiggy monos report after building.
- `--twiggy-dominators` Generate a twiggy dominators report after building.

Additional parameters are inherited from `all`, when running out of the standalone tool:
- `--target-dir-meta` For the meta crates, allows specifying the target directory where the Rust compiler will build the intermediary files.
- `--target-dir-all` Overrides both the `--target-dir-meta` and the `--target-dir-wasm` args.

[comment]: # (mx-context-auto)

### Calling `build-dbg`

There is another command, provided for convenience: `cargo run build-dbg`. Calling this is equivalent to `cargo run build --wasm-symbols --no-wasm-opt --wasm-suffix "dbg" --wat --no-imports`. It is ideal for developers who want to investigate the WebAssembly output produced by the compiler.

The output for `build-dbg` in the multisig example would be:

```text
output
├── multisig.abi.json
├── multisig-dbg.wasm
├── multisig-dbg.wat
├── multisig-full.abi.json
├── multisig-full-dbg.wasm
├── multisig-full-dbg.wat
├── multisig-view.abi.json
├── multisig-view-dbg.wasm
└── multisig-view-dbg.wat
```

It accepts all the arguments from `build`, so `--target-dir` works here too.

[comment]: # (mx-context-auto)

### Calling `twiggy`

This command is similar to `build-dbg`, in that it provides a shorthand for building contracts and analyzing their size. It is equivalent to running `cargo run build-dbg --twiggy-top --twiggy-paths --twiggy-monos --twiggy-dominators`.

[comment]: # (mx-context-auto)

### Calling `clean`

Calling `mxpy contract clean <project>` or `cargo run clean` in the meta crate will delete the `output` folder and clean outputs of the Rust crates.

[comment]: # (mx-context-auto)

### Calling `snippets`

Calling `cargo run snippets` in the meta crate will create a project called `interact-rs` in the contract main directory, containing auto-generated boilerplate code for building an interactor for the current contract.

An interactor is a small tool, meant for developers to interact with the contract on-chain. Being written in Rust, it is ideal for quick interactions and tinkering, directly from the contract project. There will be more documentation in the works on this topic.


