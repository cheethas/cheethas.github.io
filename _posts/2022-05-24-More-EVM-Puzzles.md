---
layout: post
title: More EVM Puzzles
---

It's been a while since fvictorio's first release of [Evm Puzzles](https://github.com/fvictorio/evm-puzzles). For those of us who wanted more of them daltyboy11 of goldfinch has blessed has come to the rescue, releasing 10 more puzzles extension to the original set. He's done a fantastic job retaining a lot of the same character as the original set!  
  In his own words, "these ones are harder and more focused on the CREATE and CALL opcodes" (and in my opinion a whole lot of fun).

In this article, I aim to create a guide that will help fill in any missing understanding for those stuck on any of the puzzles. I'm assuming that you have completed the first set prior over @ [EVM-PUZZLES](https://github.com/fvictorio/evm-puzzles). If you haven't completed them I recommend having a look at some of the resources listed on [devpill.me](https://www.devpill.me/docs/smart-contract-development/evm-deep-dive/) as well as having a read through the [evm.codes](https://www.evm.codes/) website, then give the first challenges a go. 

To get started, clone the [10 more evm puzzles](https://github.com/daltyboy11/more-evm-puzzles). Move your cwd into the cloned folder and install hardhat (`npm i --save-dev hardhat`). Much like the first challenges this set consists of a great interactive cli to guide you through the puzzles - to boot it up run `npx hardhat play`.

Full steps:
```shell
git clone https://github.com/daltyboy11/more-evm-puzzles
cd more-evm-puzzles
npm install --save-dev hardhat
npx hardhat play
```


## Puzzle(1)1
Straight off the bat, we are given a list of opcodes that represent an Ethereum contract. On the far left, we have the byte position of this line's opcode. In the middle, we have the opcode itself and on the far right, we have the opcode's mnemonic. To help illustrate each puzzle I have added a stack column that contains the state of the stack for each operation, as well as an action column to provide any additional comments. 

The goal remains the same as in the original Evm Puzzles. Provide the msg.value and (or) msg.data value that will complete the execution of the contract (reach STOP without REVERTing). 
```
############
# Puzzle 1 #
############
                                Stack                              Actions
00      36      CALLDATASIZE    [ cds ] 
01      34      CALLVALUE       [ val, cds ]                       
02      0A      EXP             [ ( val^cds) ]                     ( Place val^cds onto the stack )    
03      56      JUMP                                               ( Jump to PC (val^cds) )
04      FE      INVALID
05      FE      INVALID
06      FE      INVALID
07      FE      INVALID
08      FE      INVALID
09      FE      INVALID
0A      FE      INVALID
0B      FE      INVALID
0C      FE      INVALID
0D      FE      INVALID
0E      FE      INVALID
0F      FE      INVALID
10      FE      INVALID
11      FE      INVALID
12      FE      INVALID
13      FE      INVALID
14      FE      INVALID
15      FE      INVALID
16      FE      INVALID
17      FE      INVALID
18      FE      INVALID
19      FE      INVALID
1A      FE      INVALID
1B      FE      INVALID
1C      FE      INVALID
1D      FE      INVALID
1E      FE      INVALID
1F      FE      INVALID
20      FE      INVALID
21      FE      INVALID
22      FE      INVALID
23      FE      INVALID
24      FE      INVALID
25      FE      INVALID
26      FE      INVALID
27      FE      INVALID
28      FE      INVALID
29      FE      INVALID
2A      FE      INVALID
2B      FE      INVALID
2C      FE      INVALID
2D      FE      INVALID
2E      FE      INVALID
2F      FE      INVALID
30      FE      INVALID
31      FE      INVALID
32      FE      INVALID
33      FE      INVALID
34      FE      INVALID
35      FE      INVALID
36      FE      INVALID
37      FE      INVALID
38      FE      INVALID
39      FE      INVALID
3A      FE      INVALID
3B      FE      INVALID
3C      FE      INVALID
3D      FE      INVALID
3E      FE      INVALID
3F      FE      INVALID
40      5B      JUMPDEST
41      58      PC               [ pc ]
42      36      CALLDATASIZE     [ cds, pc ]
43      01      ADD              [ (cds + pc) ]                        ( Jump to PC (cds+pc) )
44      56      JUMP             
45      FE      INVALID
46      FE      INVALID
47      5B      JUMPDEST         
48      00      STOP             fin

? Enter the value to send: (0) 
? Enter the calldata to send: (0) 
```
The first opcode that we encounter is CALLDATASIZE, this opcode places the size (in bytes) of the msg.data parameter and pushes it to the stack. Similarly, CALLVALUE pushes the msg.value parameter onto the stack. You've likely come across these before.

EXP - The exponent opcode tales two inputs - the first item in the stack being the base, and the second the exponent, pushing (a^b) back onto the stack.
The PC or program counter opcode ( unsurprisingly ) returns the current value of the program counter.

The goal of this challenge is to provide CALLDATA that the JUMP operations at byte 0x03 and 0x44 execute successfully, updating the PC to 0x40 and 0x47 respectively.

The easiest way to solve this puzzle is to work backward from the JUMPDEST on 0x47. To reach this code the input to the JUMP operation jumps to the location ( CALLDATASIZE + PC ). PC pushes the current value of the program counter onto the stack, by looking at the counter on the left we can see that it will be 0x41. Leaving the CALLDATASIZE value that we must pass to be 0x47-0x41 = 0x6 bytes. 

We can be certain that any variation of 6-byte calldata is required to pass this challenge. For simplicity, I will select 0x000000000000.

Now we know the value of CALLDATASIZE ( 6 ) we can work backwards again to get the CALLVALUE required. The first jump operation must go to byte 0x40, as the result of CALLVALUE ^ CALLDATASIZE = 0x40. 0x40 is 64 in decimal, and CALLDATASIZE is 6; therefore to satisfy this equation CALLVALUE must be 2.

This leaves our inputs to be:

Value: 2
Calldata: 0x000000000000 -> 6 bytes

View the solution in [evm.codes](https://www.evm.codes/playground?callValue=2&unit=Wei&callData=0x000000000000&codeType=Bytecode&code='36340A56yyyy5B58360156x5B00'~xxxFEy~~~xzz%01xyz~_).

That's the first puzzle completed. Only nine more to go...

## Puzzle (1)2
Onto the second puzzle. 

```
############
# Puzzle 2 #
############
                                     Stack                               Action
00      36        CALLDATASIZE       [ cds ]
01      6000      PUSH1 00           [ 00, cds ]
03      6000      PUSH1 00           [ 00, 00, cds ]
05      37        CALLDATACOPY                                           ( Pop 3 values off stack, store calldata in memory )
06      36        CALLDATASIZE       [ cds ]
07      6000      PUSH1 00           [ 00, cds ]
09      6000      PUSH1 00           [ 00, 00, cds ]
0B      F0        CREATE             [ dep_addr ]                        ( deploys calldata to a new contract )
0C      6000      PUSH1 00           [ 00, dep_addr ]
0E      80        DUP1               [ 00, 00, dep_addr ]
0F      80        DUP1               [ 00, 00, 00, dep_addr ]
10      80        DUP1               [ 00, 00, 00, 00, dep_addr ]
11      80        DUP1               [ 00, 00, 00, 00, 00, dep_addr ]
12      94        SWAP5              [ dep_addr, 00, 00, 00, 00, 00 ]
13      5A        GAS                [ gas, dep_addr, 00, 00, 00, 00, 00 ]
14      F1        CALL               1                                   ( call deployed contract store result in start of memory )
15      3D        RETURNDATASIZE     [ rds ]
16      600A      PUSH1 0A           [ 0xA, rds ]
18      14        EQ                 [ (0xA == rds ? 1 : 0) ]
19      601F      PUSH1 1F           [ 1F, (0xA == rds ? 1 : 0) ]                         
1B      57        JUMPI                                                   ( jump to end if returndatasize == 0xA)
1C      FE        INVALID
1D      FE        INVALID
1E      FE        INVALID
1F      5B        JUMPDEST
20      00        STOP               fin.

? Enter the calldata:
```

This challenge aims to deploy a contract that will cause the JUMPI at byte 0x1B to succeed. To do this a call to the contract must have RETURNDATASIZE opcode must have a value of 0xA. We must deploy a contract that returns a 10 byte value. 

If you've completed the first ten challenges you have become acquainted with how contracts are deployed in the EVM. If you remember most of it you can skip this next paragraph, but if you're feeling a bit rusty look no further as I'll attempt to explain it works below. 

When the CREATE opcode is called, it consumers 3 items from the stack:

1. value - The value in wei to be sent to the created account.
2. offset - The byte offset in memory in bytes that contains the contract creation code.
3. size - The size in bytes of the contract creation code.

This contract creation code is what is executed within a contract's constructor. Within it, arbitrary instructions can be executed from the new context. The most important part of this constructor code is the RETURN value. As whatever data is returned from the constructor code will determine what is stored as the new contract's bytecode.  

The purpose of the snippet below is to return `0x69ffffffffffffffffffff600052600a6016f3` - our new contract's execution code. To do this we push the full runtime code into memory with PUSH19 and then return it.  

```
PUSH19 0x69ffffffffffffffffffff600052600a6016f3
PUSH1 0
MSTORE
PUSH1 0x20
PUSH1 0x13
PUSH1 0x20
SUB
RETURN
```

Our deployed contract will contain the following code:

```
PUSH10 0xffffffffffffffffffff 
PUSH1 0
MSTORE
PUSH1 0xa
PUSH1 0x16
RETURN
```
This will return the 10-byte value we require. You can confirm this on [evm.codes](https://www.evm.codes/playground?unit=Wei&codeType=Mnemonic&code='w0vxzzzzz%20~yMSTORE~xa~x16yRETURN'~ywvzffffy%5CnwPUSH1v%200%01vwyz~_).

Solution:
0x7269ffffffffffffffffffff600052600a6016f360005260206013602003f3

You can step through the execution of the entire challenge [here](https://www.evm.codes/playground?unit=Wei&callData=0x7269ffffffffffffffffffff600052600a6016f360005260206013602003f3&codeType=Bytecode&code='z37zF0~80808080945AF13D600a14601F57FEFEFE5B00'~6000z36~~%01z~_)


## Puzzle (1)3
```
############
# Puzzle 3 #
############
                                   Stack                                    Action
00      36        CALLDATASIZE     [ cds ]
01      6000      PUSH1 00         [ 00, cds ]
03      6000      PUSH1 00         [ 00, 00, cds ]
05      37        CALLDATACOPY                                               ( Store calldata in memory )
06      36        CALLDATASIZE     [ cds ]
07      6000      PUSH1 00         [ 00, cds ]
09      6000      PUSH1 00         [ 00, 00, cds ]
0B      F0        CREATE           [ cont_addr ]                             ( deploy calldata into a contract ) 
0C      6000      PUSH1 00         [ 00, cont_addr ]
0E      80        DUP1             [ 00, 00, cont_addr ] 
0F      80        DUP1             [ 00, 00, 00, cont_addr ]
10      80        DUP1             [ 00, 00, 00, 00, cont_addr ]
11      93        SWAP4            [ cont_addr, 00, 00, 00, 00 ]
12      5A        GAS              [ gas, cont_addr, 00, 00, 00, 00 ]
13      F4        DELEGATECALL                                               ( Delegate call contact )
14      6005      PUSH1 05         [ 05 ]
16      54        SLOAD            [ storage[5] ]                            ( load value in storage slot 5 )
17      60AA      PUSH1 AA         [ AA, storage[5] ]
19      14        EQ               [ ( AA == storage[5] ? 1 : 0) ] 
1A      601E      PUSH1 1E         [ 1E, ( AA == storage[5] ? 1 : 0) ]
1C      57        JUMPI                                                      ( jump to end; if 0xAA == storagep[5] )
1D      FE        INVALID
1E      5B        JUMPDEST
1F      00        STOP

? Enter the calldata: 
```
The contract's execution code will contain `0x69ffffffffffffffffffff600052600a6016f3`:

In this code, we encounter a new OPCODE: DELEGATECALL. DELETGATECALL is similar to the call opcode, however the call is executed within the calling contract's context. A good mental model that helps me wrap my head around it is to think that the code is copied from the callee contract into the caller contract, then executed there. All storage, memory, msg value, and sender parameters are that of the calling contract. 
I can't do justice to explaining DELEGATECALL wihtin this short article; however, [noxx has written an excellent article](https://noxx.substack.com/p/evm-deep-dives-the-path-to-shadowy-a5f?token=eyJ1c2VyX2lkIjozMTkzMTU5MCwicG9zdF9pZCI6NTM1MzE3ODksIl8iOiJsYVd3YyIsImlhdCI6MTY1MzMzMjgwMSwiZXhwIjoxNjUzMzM2NDAxLCJpc3MiOiJwdWItNzc0NzUxIiwic3ViIjoicG9zdC1yZWFjdGlvbiJ9.Yrsh5DNMrn4HOcyZ0NIt1ElXVEsKYxs8FNm_VO8qH44&s=r) that explains the concept beautifully. 

To pass this challenge we must make the storage value within the current contract have a value of AA. This is where DELEGATECALL comes in, if we can deploy a contract that performs SSTORE on slot 5 with the value 0xAA, DELEGATECALL will ensure that SSTORE sets the slots within the calling contract. Leaving SLOAD to read the correct value. 

The deployed contract code would look something like this:
```
PUSH1 0xAA
PUSH1 0x5
SSTORE
```
When DELEGATECALL executes this code it will update the code within the parent contract! Exactly what we need. (Once again you can have a peep in [evm.codes](https://www.evm.codes/playground?callValue=2&unit=Wei&callData=0x000000000000&codeType=Mnemonic&code='~AA%5Cn~5%5CnSSTORE'~PUSH1%200x%01~_))

Of course we then have to write the contract deployment [code](https://www.evm.codes/playground?callValue=2&unit=Wei&callData=0x000000000000&codeType=Mnemonic&code='w5y60AA600555~00zMSTORE~05~1BzRETURN'~zw1yz%5Cny%200xwPUSH%01wyz~_):
```
                             Stack                                          Action
PUSH5 0x60AA600555           [ 60AA600555 ]
PUSH1 0                      [ 0, 60AA600555 ]
MSTORE                                                                      ( Place contract runtime code in memory )
PUSH1 0x5                    [ 5 ]
PUSH1 0x1b                   [ 1b, 5 ]                                      ( memory is little endian - offset require) 
RETURN                                                                      ( Deploy 0x60AA600555 )
```

That leaves us with the final solution code.

0x6460AA6005556000526005601bf3

You can run this solution in [evm.codes](https://www.evm.codes/playground?unit=Wei&callData=0x6460AA6005556000526005601bf3&codeType=Bytecode&code='y37yF0~808080935AF4z0554zaa14z1e57fe5b00'~z00z60y36~~%01yz~_): Notice how the local value updates after the execution of DELEGATECALL.

## Puzzle (1)4
```
############
# Puzzle 4 #
############
                                    Stack                              Action
00      30        ADDRESS           [ addr ]
01      31        BALANCE           [ bal ] 
02      36        CALLDATASIZE      [ cds ] 
03      6000      PUSH1 00          [ 00, cds, bal ] 
05      6000      PUSH1 00          [ 00, 00, cds, bal ] 
07      37        CALLDATACOPY      [ bal ]                            ( Place calldata in memory )
08      36        CALLDATASIZE      [ cds, bal ] 
09      6000      PUSH1 00          [ 00 cds bal 
0B      30        ADDRESS           [ addr, 00, cds, bal ] 
0C      31        BALANCE           [ bal, 00, cds, bal ] 
0D      F0        CREATE            [ created_contract, bal ]
0E      31        BALANCE           [ bal_after, bal_before ]
0F      90        SWAP1             [ bal_before, bal_after ]
10      04        DIV               [ (bal_before/bal_after) ]
11      6002      PUSH1 02          [ 02, (bal_before/bal_after) ]
13      14        EQ                [ 02 == (bal_before/bal_after) ]
14      6018      PUSH1 18          [ 18, (eq result) ]
16      57        JUMPI
17      FD        REVERT
18      5B        JUMPDEST
19      00        STOP

? Enter the value to send: 0
? Enter the calldata: 
```

The aim of this puzzle is to send a value to the contract such that the balance of this contract ends up as half of that that was sent when it was created. 
In essence, we have to burn half of the ether sent during creation. The easiest way to do that is to send it to the burn address in the constructor! 

At byte 0x01 the balance of the contract is first read. This will be equal to the value provided in the value parameter. The balance of the deployed contract is checked again at byte 0xE, right after creation.

Our creation code will look something like [this:](https://www.evm.codes/playground?callValue=4&unit=Wei&codeType=Bytecode&code='60008080808060024704905AF1'_)
```
                           Stack                                       Action
60 00     PUSH1 0          [ 00 ]
80        DUP1             [ 00, 00 ] 
80        DUP1             [ 00, 00, 00 ] 
80        DUP1             [ 00, 00, 00, 00 ] 
80        DUP1             [ 00, 00, 00, 00, 00 ]
60 02     PUSH1 02         [ 02, 00, 00, 00, 00, 00 ] 
47        SELFBALANCE      [ bal, 02, 00, 00 ,00 ,00 ,00 ]
04        DIV              [ (bal/2), 00, 00, 00, 00, 00 ]
90        SWAP1            [ 00, (bal/2), 00, 00, 00, 00 ]            
5A        GAS              [ gas, 00, (bal/2), 00, 00, 00, 00 ]
F1        CALL                                                        ( Send (bal/2) to the zero address)
```

Calldata: 0x60008080808060024704905AF1
Value: Any value that is a multiple of 2!

[See solution here!](https://www.evm.codes/playground?callValue=20&unit=Wei&callData=0x60008080808060024704905AF1&codeType=Bytecode&code='303136~0~03736~03031F0319004~214601857FD5B00'~600%01~_)

## Puzzle (1)5
```
############
# Puzzle 5 #
############
                                    Stack                                           Action
00      6020      PUSH1 20          [ 20 ]
02      36        CALLDATASIZE      [ cds, 20 ]
03      11        GT                [ (cds > 0x20) ? 1 : 0 ]
04      6008      PUSH1 08          [ 08,(cds > 0x20) ? 1 : 0 ]
06      57        JUMPI                                                            ( jump if calldatasize is > 32 bytes ) 
07      FD        REVERT            
08      5B        JUMPDEST          
09      36        CALLDATASIZE      [ cds ]
0A      6000      PUSH1 00          [ 00, cds ]
0C      6000      PUSH1 00          [ 00, 00, cds ]
0E      37        CALLDATACOPY                                                     ( calldata is copied into memory )
0F      36        CALLDATASIZE      [ cds ]
10      59        MSIZE             [ size_of_allocated_memory, cds ]
11      03        SUB               [ ( soam-cds ) ]
12      6003      PUSH1 03          [ 03, ( soam-cds ) ]
14      14        EQ                [ ( 03 == ( soam - cds ) ? 1 : 0 )
15      6019      PUSH1 19          [ 19, ( 03 == ( soam - cds ) ? 1 : 0 )
17      57        JUMPI                                                            ( jump if the size of hot memory  minus the calldata is equal to 3 ) 
18      FD        REVERT
19      5B        JUMPDEST
1A      00        STOP              fin

? Enter the calldata: 
```
To pass the first JUMPI instruction in this challenge calldata sent must be larger than 32-bytes. After the first JUMPI the calldata is copied into memory. So far very vanilla stuff! 

The main mechanic in this challenge is understanding how the evm allocates new memory. The EVM provides 3 opcodes to interact with memory. MLOAD consumes an offset and loads 32-bytes from memory. MSTORE stores 32-bytes into memory and MSTORE8 stores a single byte into memory. The catch with this challenge is around the MSTORE allocates memory in 32-byte chunks. If you were to store 2-bytes into memory, 32-bytes would still be allocated. 

The MSIZE opcode reads the size amount of allocated or hot memory. Therefore, even if you only store 5 bytes MSIZE would return 32-bytes as the size of the allocated memory. Now we have this knowledge we can pass this challenge.

As the first JUMPI requires that more than 32-bytes of memory are passed in as calldata. This calldata is then stored in memory. To complete this challenge you must provide calldata that is 3 bytes smaller than the allocated memory. Remember how memory is allocated in 32 byte chunks? Therefore we can provide any size of calldata that is 32 + (32\*x) - 3-bytes long.

This will do: 
0x60008080806002335af160005260206000f3000000000000000000000000000060008080806002335af160005260206000f30000000000000000000000 (61-bytes)

[Code here](https://www.evm.codes/playground?unit=Wei&callData=0x60008080806002335af160005260206000f3000000000000000000000000000060008080806002335af160005260206000f30000000000000000000000&codeType=Bytecode&code='60203611~857FD5B36~0~037365903~314601957FD5B00'~600%01~_).

## Puzzle (1)6
```
############
# Puzzle 6 #
############
00      7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0      PUSH32 FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0                             
21      34                                                                      CALLVALUE
22      01                                                                      ADD
23      6001                                                                    PUSH1 01
25      14                                                                      EQ
26      602A                                                                    PUSH1 2A
28      57                                                                      JUMPI
29      FD                                                                      REVERT
2A      5B                                                                      JUMPDEST
2B      00                                                                      STOP

? Enter the value to send: (0) 
```
To pass this challenge that comparison value of 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0 + CALLVALUE must equal 1. At first glance that doesn't seem quite right. How can go from such a large number to 1? If you are familiar with solidity before version 0.8.0 you will have lived through the pain of integer overflow and underflow, getting around it with libraries like SafeMath. However, today it is your friend and you will use it to solve this challenge. 

Provide CALLVALUE such that 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0 + x = 1.
Since the least significant byte is set to 0 we have to add 17 to make this happen - Give it a go yourself.

[View solution here](https://www.evm.codes/playground?callValue=17&unit=Wei&codeType=Bytecode&code='7yyyy03401600114602a57fd5b00'~zzzzzffy~~%01yz~_).

## Puzzle (1)7
```
############
# Puzzle 7 #
############
                                   Stack
00      5A        GAS              [ start_GAS ]
01      34        CALLVALUE        [ cv,  start_GAS ]
02      5B        JUMPDEST         
03      6001      PUSH1 01         [ 01, cv, start_GAS ]
05      90        SWAP1            [ cv,  01, start_GAS ]
06      03        SUB              [ (cv - 01), start_GAS ]                            ( callvalue - 1 )
07      80        DUP1             [ (cv - 01), (cv - 01), start_GAS ]                       
08      6000      PUSH1 00         [ 00, (cv - 01), (cv - 01), start_GAS ]      
0A      14        EQ               [ (00 == (cv - 01) ? 1 : 0, (cv - 01), start_GAS ]  ( callvalue - 1 == 0 )
0B      6011      PUSH1 11         [ 11, (00 == (cv - 01) ? 1 : 0, (cv-01), start_GAS ]
0D      57        JUMPI                                                                ( if (cv - 1) == 0 go to line 0x11 )
0E      6002      PUSH1 02         [ 02, (cv - 01), start_GAS ] 
10      56        JUMP                                                                 ( jump back to line 0x2 )
11      5B        JUMPDEST
12      5A        GAS              [ current_GAS, (cv -01), start_GAS ] 
13      90        SWAP1            [ (cv -01), current_GAS, start_GAS ]
14      91        SWAP2            [ start_GAS, current_GAS, (cv-01) ]
15      03        SUB              [ (start_GAS - current_GAS), (cv-01) ]
16      60A6      PUSH1 A6         [ A6, (start_GAS - current_GAS), (cv-01) ]
18      14        EQ               [ ( A6 == (start_GAS - current_GAS ), (cv-01) ]
19      601D      PUSH1 1D         [ 1D, ( A6 == (start_GAS - current_GAS ), (cv-01) ]
1B      57        JUMPI                                                                ( jump if (start gas - current gas) == A6 )
1C      FD        REVERT
1D      5B        JUMPDEST
1E      00        STOP            fin

? Enter the value to send: (0) 
```
I love the mechanics of this challenge. This is the first time that we have seen a loop implementation in bytecode. 
The best way I can demonstrate this code is to write it out in solidity.

```
contract Puzzle7 {
    fallback() {
        uint256 startGas = msg.gasLimit;
        
        for (uint256 i = msg.value; i == 0; i--) {}
        if ((startGas - msg.gasLeft) != 166) revert() 
    }
}
```
Note: This is not an exact representation but I hope it illustrates the solution somewhat. 

The idea of this challenge is to provide a CALLVALUE that causes the forloop to execute enough times such that 166 (0xA6) gas is spent.

To spend 166 gas we have to execute the loop 4 times. Give it a go in [evm.codes](https://www.evm.codes/playground?callValue=4&unit=Wei&codeType=Bytecode&code='5a345b60019003806000146011576002565b5a90910360a614601d57fd5b00'_).

## Puzzle (1)8
```
############
# Puzzle 8 #
############
                                   Stack                                         Action
00      34        CALLVALUE        [ cv ]
01      15        ISZERO           [ (cv == 0) ]
02      19        NOT              [ !(cv == 0) ]
03      6007      PUSH1 07         [ 07, !(cv == 0) ]
05      57        JUMPI                                                          Jump to 0x7 if !(cv == 0)
06      FD        REVERT
07      5B        JUMPDEST         
08      36        CALLDATASIZE     [ cds ]
09      6000      PUSH1 00         [ 00, cds ]
0B      6000      PUSH1 00         [ 00, 00, cds ]
0D      37        CALLDATACOPY                                                   ( Calldata copied into memory )
0E      36        CALLDATASIZE     [ cds ]
0F      6000      PUSH1 00         [ 00, cds ]
11      6000      PUSH1 00         [ 00, 00, cds ]                               
13      F0        CREATE           [ cont_addr ]                                 ( Contract created with calldata )
14      47        SELFBALANCE      [ bal, cont_addr ]                      
15      6000      PUSH1 00         [ 00, bal, cont_addr ]
17      6000      PUSH1 00         [ 00, 00, bal, cont_addr ]
19      6000      PUSH1 00         [ 00 ,00, 00, bal, cont_addr ]
1B      6000      PUSH1 00         [ 00, 00, 00, 00, cv, cont_addr ]
1D      47        SELFBALANCE      [ bal, 00, 00, 00, 00, cv, cont_addr ]
1E      86        DUP7             [ cont_addr, bal, 00, 00, 00, 00, cv, cont_addr ]
1F      5A        GAS              [ gas, cont_addr, bal, 00, 00, 00, 00, cv, cont_addr ]
20      F1        CALL             [ 01, bal, cont_addr ]                         ( Call deployed contract no calldata)
21      6001      PUSH1 01         [ 01, 01, bal, cont_addr ] 
23      14        EQ               [ 01, bal, cont_addr ]                         ( Did the contract call succeed )
24      6028      PUSH1 28         [ 28, bal, cont_addr ]                               
26      57        JUMPI                                                           ( Jump If succeeded )
27      FD        REVERT
28      5B        JUMPDEST         
29      47        SELFBALANCE      [ bal_after, bal_before ]
2A      14        EQ               [ (bal_after==bal_before) ] 
2B      602F      PUSH1 2F         
2D      57        JUMPI                                                           ( Jump if balance before == balance after )                    
2E      FD        REVERT  
2F      5B        JUMPDEST
30      00        STOP             fin
```

Have you ever wondered what `payable` does when you add it to a function definition in solidity? By default all functions in solidity begin with bytecode similar to the first few bytes of this challenge ( CALLVALUE, ISZERO, NOT, PUSH X, JUMPI, REVERT ), this code exists as a safety rail to prevent users of smart contract from sending ether to functions that are not intended to receive ether. It exists for the sake of user protection and security, however it adds ( redundant? ) checks that cost gas. Adding the payable descriptor actually removes this code, allowing the contract to receive ether without reverting. In essence, this contract refuses to receive ether.

The snippet then goes on to create a contract containing the calldata provided. 
Followed by calling the crated contract with the full balance of the current contract, checks to see if the execution succeeded, then checks if the balance before sending is the same as the balance after sending.

At first instinct we can perform the same code as in challenge 4, however; this contract does not receive ether so transfering ether to this account will not work!

The EVM does provide a way to send ether to contracts that do not receive ether and it is through SELFDESTRUCT.

Here is the description from evm.codes:  
*The current account is registered to be destroyed, and will be at the end of the current transaction. The transfer of the current balance to the given account cannot fail. In particular, the destination account code (if any) is not executed, or, if the account does not exist, the balance is still added to the given address.*

Unlike the CALL opcode, no code is executed with SELFDESTRUCT. Therefore the balance is force added to the account.

Here is what the deployed bytecode will look like: 
```
00  33   CALLER
01  FF   SELFDESTRUCT
```

As perusual, we have to also provide the contract deployment code that will return our deployed bytecode. ([evm.codes example](https://www.evm.codes/playground?callValue=4&unit=Wei&codeType=Mnemonic&code='v2wx33FFzzz~yMSTORE~x2~x1eyRETURN'~yv1wz%20%20y%5Cnw%200vPUSH%01vwyz~_))
```
PUSH2 0x33FF      code for self destruct (CALLER; SELFDESTRUCT)
PUSH1 0x00
MSTORE
PUSH1 0x02
PUSH1 0x1E
RETURN
```

callvalue -> 0x6133FF6000526002601Ef3
[Solution](https://www.evm.codes/playground?unit=Wei&callData=0x6133FF6000526002601Ef3&codeType=Bytecode&code='341519y07z36x3736xf047xx47865af1y0114y28z4714y2fz00'~y00z57fd5by60x~~%01xyz~_).

## Puzzle (1)9
```
############
# Puzzle 9 #
############

00      34        CALLVALUE     cv
01      6000      PUSH1 00      00 cv     
03      52        MSTORE        store callvalue (32bytes)
04      6020      PUSH1 20      20
06      6000      PUSH1 00      00 20
08      20        SHA3          keccak(cv)           
09      60F8      PUSH1 F8      f8 keccak(cv)
0B      1C        SHR           keccak(cv) >> f8 (248) == first byte remains
0C      60A8      PUSH1 A8      MSB(keccak(cv)) A8
0E      14        EQ            Equal if keccak MSB is A8
0F      6016      PUSH1 16
11      57        JUMPI
12      FD        REVERT
13      FD        REVERT
14      FD        REVERT
15      FD        REVERT
16      5B        JUMPDEST
17      00        STOP

? Enter the value to send: (0)
```
This introduces us to two new opcodes that we have not seen yet SHA3 and SHR. 
SHA3 takes in two opcodes as arguments, the offset and size of bytes of memory to be hashed. Placing the output onto the stack. SHR is the shift right operation, performing a bitshift by x bytes. 

To get the winning input for this function you have to brute-force until you reach a keccak that has the least significant byte value that is equal to A8. As keecak hashes are uniformly distributed it should theoretically take you less than 256 tries. Give it a go? [(evm.codes link)](https://www.evm.codes/playground?unit=Wei&codeType=Bytecode&code='34600052602060002060F81C60A814601657FDFDFDFD5B00'_)

## Puzzle (2|1)0
```
#############
# Puzzle 10 #
#############

00      6020                                                                    PUSH1 20
02      6000                                                                    PUSH1 00
04      6000                                                                    PUSH1 00
06      37                                                                      CALLDATACOPY
07      6000                                                                    PUSH1 00
09      51                                                                      MLOAD
0A      7FF0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0      PUSH32 F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0
2B      16                                                                      AND
2C      6020                                                                    PUSH1 20
2E      6020                                                                    PUSH1 20
30      6000                                                                    PUSH1 00
32      37                                                                      CALLDATACOPY
33      6000                                                                    PUSH1 00
35      51                                                                      MLOAD
36      17                                                                      OR
37      7FABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABAB      PUSH32 ABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABAB
58      14                                                                      EQ
59      605D                                                                    PUSH1 5D
5B      57                                                                      JUMPI
5C      FD                                                                      REVERT
5D      5B                                                                      JUMPDEST
5E      00                                                                      STOP

? Enter the calldata: 
```

Well done you've made it to the last challenge! This one is some good ol fashioned bitwise operations.  

The two operations in question are located at bytes 0x2B and 0x36. With an AND and OR at each respectively. You provide the input to the first operation through the first 32 bytes of calldata and input for the second operation through the second 32 bytes. The aim of the challenge is to end up with ( CALLDATA[0:32] AND 0xF0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0 ) OR CALLDATA[32:64] == 0xABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABAB. By applying general rules of bitwise operations A AND B OR C can simplify to just C. Where C in this instance is CALLDATA[32:64]. In otherwords, it does not matter what you provide as the first 32 bytes aslong as you provide 0xABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABAB as the second set of bytes.

Heres the final input:
0x0000000000000000000000000000000000000000000000000000000000000000ABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABAB.

[Solution on evm.codes](https://www.evm.codes/playground?unit=Wei&callData=0x0000000000000000000000000000000000000000000000000000000000000000ABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABABAB&codeType=Bytecode&code='xyw7ftttt16xxw177fssss14605d57fd5b00'~vvvvzuuuuy6000x6020wy37y51vf0uabt~~szz%01stuvwxyz~_).

There we have it!! I hope you've learned something about the EVM from reading this article. Maybe I'll create some more challenges like this myself!! 

Many thanks to the brains behind these puzzles, fvictorio and daltyboy11 - they served as a perfect and fun excuse for me to spend more time playing around with the EVM. See y'around.


