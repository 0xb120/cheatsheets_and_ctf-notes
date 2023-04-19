# Solidity 101

## Definitions

### Setup contract

A setup contract is a special type of smart contract that is used to initialize or configure other contracts. It is often used in conjunction with a factory contract, which is a contract that can create instances of other contracts.

A setup contract typically contains logic that is used to configure the parameters and behavior of the contract instances that are created by the factory contract. For example, a setup contract might contain logic that specifies the permissions of different contract users, the default values for certain variables, or the rules for modifying certain data within the contract.

To use a setup contract, you would first deploy the setup contract to the Ethereum blockchain, and then use the setup contract's functions to configure the parameters of the contract instances that are created by the factory contract. Once the setup contract has been configured, the factory contract can use the setup contract's parameters to create instances of the desired contract with the desired behavior.

Setup contracts are useful because they allow you to reuse the same factory contract to create multiple instances of a contract with different configurations, rather than having to deploy a separate factory contract for each configuration. This can save time and resources, as it reduces the number of contract deployments that are required.

## Debug and print variables inside the console

```js
// remix supports hardhat console.log;

// 1st import as follows
import "hardhat/console.sol";

// then to log use:
console.log("log 1")
```

## Dump and decompile a deployed smart contract

```bash
$ curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_getCode","params":["0x449438AADDd972b602AF87620A08f2695Ce89dF9", "latest"],"id":1}' http://challs.htsp.ro:9001/a4e4e38b-28a9-4545-a53a-ab7f2c814cbf
{"jsonrpc":"2.0","id":1,"result":"0x608060405234801561001057600080fd5b50600436106100365760003560e01c806364d98f6e1461003b578063be0ba8d214610058575b600080fd5b610043610083565b60405190151581526020015b60405180910390f35b60005461006b906001600160a01b031681565b6040516001600160a01b03909116815260200161004f565b60008060009054906101000a90046001600160a01b03166001600160a01b031663799320bb6040518163ffffffff1660e01b8152600401602060405180830381865afa1580156100d7573d6000803e3d6000fd5b505050506040513d601f19601f820116820180604052508101906100fb9190610100565b905090565b60006020828403121561011257600080fd5b8151801515811461012257600080fd5b939250505056fea264697066735822122080dc49e5806f8d3590ff266a78a4c62f713c9f01ca443509d2af8967e9a2a1df64736f6c63430008110033"} 
```

Decompile it using [ethervm.io](https://ethervm.io/decompile):

```c
contract Contract {
    function main() {
        memory[0x40:0x60] = 0x80;
        var var0 = msg.value;
    
        if (var0) { revert(memory[0x00:0x00]); }
    
        if (msg.data.length < 0x04) { revert(memory[0x00:0x00]); }
    
        var0 = msg.data[0x00:0x20] >> 0xe0;
    
        if (var0 == 0x64d98f6e) {
            // Dispatch table entry for isSolved()
            var var1 = 0x0043;
            var1 = isSolved();
            var temp0 = memory[0x40:0x60];
            memory[temp0:temp0 + 0x20] = !!var1;
            var1 = temp0 + 0x20;
        
        label_004F:
            var temp1 = memory[0x40:0x60];
            return memory[temp1:temp1 + var1 - temp1];
        } else if (var0 == 0xbe0ba8d2) {
            // Dispatch table entry for 0xbe0ba8d2 (unknown)
            var1 = 0x006b;
            var var2 = storage[0x00] & (0x01 << 0xa0) - 0x01;
            var temp2 = memory[0x40:0x60];
            memory[temp2:temp2 + 0x20] = var2 & (0x01 << 0xa0) - 0x01;
            var2 = temp2 + 0x20;
            goto label_004F;
        } else { revert(memory[0x00:0x00]); }
    }
    
    function isSolved() returns (var r0) {
        var var0 = 0x00;
        var var1 = storage[var0] & (0x01 << 0xa0) - 0x01 & (0x01 << 0xa0) - 0x01;
        var var2 = 0x799320bb;
        var temp0 = memory[0x40:0x60];
        memory[temp0:temp0 + 0x20] = (var2 & 0xffffffff) << 0xe0;
        var var3 = temp0 + 0x04;
        var temp1 = memory[0x40:0x60];
        var temp2;
        temp2, memory[temp1:temp1 + 0x20] = address(var1).staticcall.gas(msg.gas)(memory[temp1:temp1 + var3 - temp1]);
        var var4 = !temp2;
    
        if (!var4) {
            var temp3 = memory[0x40:0x60];
            var temp4 = returndata.length;
            memory[0x40:0x60] = temp3 + (temp4 + 0x1f & ~0x1f);
            var1 = 0x00fb;
            var3 = temp3;
            var2 = var3 + temp4;
            return func_0100(var2, var3);
        } else {
            var temp5 = returndata.length;
            memory[0x00:0x00 + temp5] = returndata[0x00:0x00 + temp5];
            revert(memory[0x00:0x00 + returndata.length]);
        }
    }
    
    function func_0100(var arg0, var arg1) returns (var r0) {
        var var0 = 0x00;
    
        if (arg0 - arg1 i< 0x20) { revert(memory[0x00:0x00]); }
    
        var temp0 = memory[arg1:arg1 + 0x20];
        var var1 = temp0;
    
        if (var1 == !!var1) { return var1; }
        else { revert(memory[0x00:0x00]); }
    }
}
```


# Tools for assessing smart contract's security
- [myth](https://github.com/ConsenSys/mythril): Mythril is a security analysis tool for EVM bytecode. It detects security vulnerabilities in smart contracts built for Ethereum, Hedera, Quorum, Vechain, Roostock, Tron and other EVM-compatible blockchains. It uses symbolic execution, SMT solving and taint analysis to detect a variety of security vulnerabilities. It's also used (in combination with other tools and techniques) in the MythX security analysis platform.
- [JEB](https://www.pnfsoftware.com/): Decompile and debug binary code. Break down and analyze document files. Android Dalvik, Intel x86, ARM, MIPS, RISC-V, S7 PLC, Java, WebAssembly & Ethereum Decompilers.
- [ethervm.io](https://ethervm.io/decompile)
