# EVM-Puzzles-Walkthrough


TODO: need to explain when values are consumed during operations, also need to explain that the value you enter for tx value is in decimal and it gets converted to hexadecimal.

EVM-Puzzles is a fantastic Github repo featuring a collection of challenges involving EVM opcodes, calldata and ….  Each puzzle starts out by giving you a series of opcodes and prompts you to input the correct transaction value that will allow the sequence to run without reverting. This article aims to be a low impact walkthrough for each puzzle, making it easy for anyone with any experience level to fully understand the why and the how behind each solution. This walkthrough will assume that you are familiar with stack machines. If not, first read take a look at [how stack machines work](). There are 10 puzzles, someone with no experience, this should take about 2 hours, someone with some experience this should take about 1 hour and someone with a lot of experience but still wanting to go through the walkthrough, this should take somewhere around 30-40 min. With that note, we are almost ready to get started!

There are a few things you’ll need before getting started.

First, head over to the [EVM-Puzzles repo](), clone the project and set up your local environment. Make sure you have hardhat installed. If you don’t, you can simply enter `npm install --save-dev hardhat` when you are in the root project folder.

Next, if you are newer to the EVM, take a brief look at the [EVM opcodes]() (don’t feel the need to understand everything, just get the general idea).

With all of that out of the way, let’s get started! To start the first puzzle, cd into the root directory of the project and enter npx hardhat play into the terminal.

# Puzzle #1

Let’s take a look at the first puzzle. You are given a series of opcodes that represent a contract. The puzzle prompts you to enter a value to send, or in other words if you sent a transaction to this contract what value would the transaction value need to be for this contract to run without reverting?  Go ahead and give it a shot and then feel free to come back here if you get stuck or want an in depth look at the solution after solving the puzzle.

```js
############
# Puzzle 1 #
############

00      34      CALLVALUE
01      56      JUMP
02      FD      REVERT
03      FD      REVERT
04      FD      REVERT
05      FD      REVERT
06      FD      REVERT
07      FD      REVERT
08      5B      JUMPDEST
09      00      STOP

? Enter the value to send: (0)

```

Ok, now for the explainer. First, we need to know what the [CALLVALUE instruction]()  does. This opcode gets the value of the current call (ie. the transaction value) in wei and pushes that value to the top of the stack. So if we entered a value of 10, before the `CALLVALUE` instruction is evaluated, the stack would look like this.

```js
[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

After the `CALLVALUE` opcode is evaluated and the value we entered is pushed onto the stack, the stack looks like this.

```js
[10 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Next, we need to know what the [JUMP]() instruction does. This opcode consumes the top value on the stack and jumps to the `n`th instruction in the sequence where `n` is the value at the top of the stack. A quick example will make this more clear.  Lets say we have the following sequence.

```js

00      34      CALLVALUE
01      56      JUMP
02      80      DUP1
03      80      DUP1
04      80      DUP1
05      00      STOP

? Enter the value to send: (0)
```

If we enter 4 in as the value to send, the `CALLVALUE` opcode will get the value of the transaction and push that value on the stack. so now our stack looks like this.

[4 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]

Then the `JUMP` opcode consumes the top value on the stack and jumps to the instruction at that number. Since the value on the top of the stack is `4`, the program counter jumps to the fourth instruction and continues.  A `JUMP` opcode must alter the program counter to end up at a `JUMPDEST` instruction. For the above example, we can think of the program looking like this after the `JUMP` instruction is evaluated.

```js
04      80      DUP1
05      00      STOP
```

Now that all of that is clear, lets get back to the puzzle. We need to enter a value so that the program runs without hitting a REVERT instruction.

```js
00      34      CALLVALUE
01      56      JUMP
02      FD      REVERT
03      FD      REVERT
04      FD      REVERT
05      FD      REVERT
06      FD      REVERT
07      FD      REVERT
08      5B      JUMPDEST
09      00      STOP
```

To do this, we can enter a call value of 8, which causes the `CALLVALUE` instruction to push `8` onto the stack where the `JUMP` instruction then consumes that value and jumps to the 8th instruction, skipping all of the REVERT instructions. Nice work, one puzzle down!


# Puzzle #2

Now that you have your feet wet, lets take a look at the second puzzle. Give it a shot on your own and just like before, feel free to come back to check out the solution as well as the explanation. Here is the puzzle.

```js
############
# Puzzle 2 #
############

00      34      CALLVALUE
01      38      CODESIZE
02      03      SUB
03      56      JUMP
04      FD      REVERT
05      FD      REVERT
06      5B      JUMPDEST
07      00      STOP
08      FD      REVERT
09      FD      REVERT

? Enter the value to send: (0) 
```

Just like before, we need to enter a transaction value to send that will cause the program to run without reverting. If we take a look at the sequence of instructions, we can see that we need the `JUMP` opcode to alter the program counter to the 6th instruction. Just like before, the first instruction is call value, so we know that the value we enter will end up on the top of the stack after the first instruction.

Lets take a look at the [CODESIZE instruction](). This opcode gets the size of the code running in the current environment. In this example, we can manually check out the size of the code by looking at how many opcodes there are in the sequence. Each opcode is 1 byte, and in this puzzle we have 10 opcodes meaning that the size of the code is 10 bytes. As an important side note, the EVM uses hex numbers to represent byte code. If you are unfamiliar, check out [how hex numbers]() work. With this in mind, we can know that `a` gets pushed to the stack, representing 10 bytes.

The next opcode we come across is the [SUB instruction](), which takes the first stack element minus the second stack element, consumes both the first and second element and pushes the result to the top of the stack. For example if we had a stack that looked like this.

```js
[3 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Executing the `SUB` instruction would produce the following stack result.

```js
[1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

With this information let's get back to the puzzle. We now know that the program first evaluates the `CALLVALUE` instruction, pushing the value of the transaction on the stack. Then the program evaluates the `CODESIZE` instruction, which pushes `a` (representing 10 bytes) onto the stack. We also know that we need `JUMP` to alter the program counter to the 6th instruction. In essence, we have the following stack after the `CODESIZE` instruction.

```js
[a your_input 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Go ahead and if you haven’t already finished this puzzle, try to use the above information to enter the correct value, or feel free to read on for the last step of the solution.

Since we know the `SUB` instruction is next, we need to enter a value such that `a - your_input` equals `6`, which makes our answer 4.

# Puzzle #3

Get ready to switch gears a little. Instead of entering a transaction value to solve the puzzle, we are going to have to enter calldata. Calldata is a read-only byte-addressable space where the transaction data during a message or call is held. In plain english, this is byte code payload that is attached to a message ([click here to learn more about the anatomy of a transaction in Ethereum]()).

Let’s take a look at the puzzle.

```js

############
# Puzzle 3 #
############

00      36      CALLDATASIZE
01      56      JUMP
02      FD      REVERT
03      FD      REVERT
04      5B      JUMPDEST
05      00      STOP

? Enter the calldata: 
```

For this puzzle, its helpful to know that 1 byte is 8 bits and that numbers 0-255 can represent one byte in the EVM. This puzzle also introduces us to a new opcode called[CALLDATASIZE](). This instruction gets the size of the calldata in bytes and pushes it onto the stack.

With that knowledge, this makes this puzzle pretty straightforward. We will need to pass in a hex number as the calldata such that the `CALLDATASIZE` instruction pushes 4 on the stack. From there, the `JUMP` instruction will jump to the fourth instruction in the sequence, reaching the `JUMPDEST`. To keep it simple, 0xff is usually used to represent 1 byte since ff in hexadecimal evaluates to 255 in decimal format. All we need to do is copy ff four times, making the byte code we should enter: 0xffffffff. Another puzzle down!

# Puzzle #4

Enter bitwise. In this puzzle we see our first `XOR` instruction. As usual, feel free to give it a shot and figure it out on your own. When you’re ready, head back here for the solution and explanation.

```js
############
# Puzzle 4 #
############

00      34      CALLVALUE
01      38      CODESIZE
02      18      XOR
03      56      JUMP
04      FD      REVERT
05      FD      REVERT
06      FD      REVERT
07      FD      REVERT
08      FD      REVERT
09      FD      REVERT
0A      5B      JUMPDEST
0B      00      STOP

? Enter the value to send: (0) 
```

We know that `CALLVALUE` will push the value we enter onto the top of the stack and we can take a look at how many instructions there are to know how big the `CODESIZE` is. In this program, we have 12 instructions, which makes 12 bytes or `c` in hexadecmial, which gets pushed to the stack.  So now our stack looks like this.

[c your_input 0 0 0 0 0 0 0 0 0 0 0 0 0 0]

Let’s take a look at the [XOR instruction](). This instruction evaluates two numbers in their binary representation and returns a `1` in each bit position where the bits of **either, but not both** operands are `1`s. Let’s take a look at quick example. Say that we have two numbers on the top of the stack.

[5 3 0 0 0 0 0 0 0 0 0 0 0 0 0 0]

When executing the `XOR` instruction, we can imagine the two numbers on top of each other in binary representation like this.

```js
00000000000000000000000000000101
00000000000000000000000000000011
```

Then, bit by bit, the two numbers are evaluated against each other. If one bit is a `0` and the other bit is a `1`, the resulting bit will be a `1`, if both bits are `0`s or both bits are `1`s, the resulting number is a `0`. So the result of `5 XOR 3` is this.

```js
00000000000000000000000000000110
```

Back to the puzzle. So we know that we have `c` at the top of the stack and `your_input` at the second position. The `JUMP` instruction that comes after the `XOR` needs to send us to the 10th instruction. So now with all that information, we just need to enter a callvalue so that `c XOR callvalue` results in hexidecimal `10`. Go ahead and give it a shot on your own.

Ok, now for the final steps. We know that we need the result of `XOR` to be `10`, which in binary is represented as `1010`. We also have `c` on the stack, which in binary is represented as `1100`. So now we need to find a number such that `c XOR your_input` results in `1010`, making the number we need to enter `0110`. This evaluates to the hex number `06`. 6 is our answer!


# Puzzle #5

Welcome to the next puzzle, where we are met with a few new opcodes. Feel free to give it a shot. In the meantime, let’s take a look at the sequence of instructions for this puzzle.

```js
############
# Puzzle 5 #
############

00      34          CALLVALUE
01      80          DUP1
02      02          MUL
03      610100      PUSH2 0100
06      14          EQ
07      600C        PUSH1 0C
09      57          JUMPI
0A      FD          REVERT
0B      FD          REVERT
0C      5B          JUMPDEST
0D      00          STOP
0E      FD          REVERT
0F      FD          REVERT

? Enter the value to send: (0) 
```

`DUP1` meet reader, reader meet `DUP1`. The [DUP1 instruction]() is pretty straight forward. It duplicates the value at the 1st position on the stack and pushes the duplicate to the top of the stack. Similarly, DUP2 would duplicate the value at the second position on the stack and push the duplicate value to the top. There are DUP instructions for all positions in the stack (`DUP1-DUP16`).

Taking a look at the first two instructions of the puzzle, first `CALLVALUE` is executed, pushing the value we enter as the transaction value to the top of the stack. Then `DUP1` is executed, duplicating the value we entered and pushing it to the top of the stack. So now after the first two instructions, our stack looks like this.

```js
[your_input your_input 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Then we are met with another new instruction. The [MUL instruction]() takes the first two values on the stack, multiplies them together and pushes the result onto the top of the stack. So in this instance, `your_input` is multiplied by `your_input` and the resulting stack looks like this.

```js
[mul_result 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Next we encounter the [PUSH2 instruction](). This instruction pushes a 2 byte value onto the top of the stack. When you see this instruction, it will always be accompanied by the value that it will push. For example, in our puzzle it is seen as `PUSH2 0100` meaning that it will push the hex number `0100` onto the top of the stack. There are push instructions from `PUSH1` to `PUSH32`.

Coming back to our puzzle, since the next instruction is `PUSH2 0100`, our resulting stack will now look like this.

```js
[0100 mul_result 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Now we encounter the [EQ instruction](). This instruction takes the first two values on the stack, runs an equality comparison and pushes the result on the top of the stack. If the first two values are equal, `1` is pushed to the top and if they are not equal, `0` is pushed to the top instead. Both values at positions 1 and 2 on the stack are consumed as the `EQ` instruction is evaluated.

For simplicity sake, let’s say that the `mul_result` is `0100` so when the `EQ` instruction is evaluated, `1` is pushed to the stack, making our stack now look like this.

```js
[1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

The next instruction that is evaluated is `PUSH1 0C` which pushes `c` to the top of the stack. Following this instruction, we see yet another new instruction. The [JUMPI instruction]() will conditionally alter the program counter. This instruction looks at the the second stack element to know if it should jump or not depending on if it is a `1` or a `0`. Then the first stack element is used to know what position to jump to. The JUMPI instruction consumes both values during this process. So taking a look at our puzzle, after the `PUSH1 0c` instruction, our stack looks like this.

```js
[0c 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

First, the `JUMPI` instruction checks the second stack element and in this case it is 1 indicating that the program should jump. Then `JUMPI` checks the first stack element to know where it should jump to. In this case, the top stack value is `0c` meaning that it will jump to the 12th instruction, which is our `JUMPDEST`. 

And that will complete our puzzle! So with all this information we now know that we need to enter a callvalue so that when it gets duplicated once (making the first two elements on the stack the callvalue), and after the top stack values are multiplied, our result is the hex number `0100`. Feel free to give it a shot from here and see if you can figure it out.

Ok now for the final steps. We can convert `0100` into a decimal number and get 256. Then we can take the square root of 256 since the `DUP1 MUL` is essentially multiplying the number by itself. The resulting number is 16, which is the answer to this puzzle!

# Puzzle #6

5 puzzles down, 5 to go! As usual, give the puzzle a go, then feel free to come back here for the solution and explanation. 

```js
############
# Puzzle 6 #
############

00      6000      PUSH1 00
02      35        CALLDATALOAD
03      56        JUMP
04      FD        REVERT
05      FD        REVERT
06      FD        REVERT
07      FD        REVERT
08      FD        REVERT
09      FD        REVERT
0A      5B        JUMPDEST
0B      00        STOP

? Enter the calldata: 
```

Say hello to the [CALLDATALOAD instruction](). This instruction gets the input data of the sent transaction. There are a few important things to note about this opcode.  `CALLDATALOAD` expects an integer between 0 and 32 at the top of the stack to know what byte to start loading the calldata from. For example, if you send a 32 byte sequence as calldata with a transaction and you push `08` to the top of the stack when you execute `CALLDATALOAD`, the all of the calldata from byte 8 to byte 32 will be pushed onto the top of the stack.

Now back to the puzzle. We can see that there is `PUSH1 00` followed by `CALLDATALOAD` meaning that the calldata will be loaded in starting from byte 0 and the entire 32 byte calldata will be pushed onto the top of the stack. We can see that the `JUMP` instruction needs to alter the program counter to `0a` (ie. the 10th instruction). Feel free to stop here and try to solve the rest of the puzzle. 

Ok let’s go over the final steps. We know that calldata is in hexadecimal, so it might seem intuitive to enter `0x0a` as the calldata to complete the puzzle, but you might have noticed that this doesn’t work. This is because when calldata is sent, since the byte sequence was not 32 bytes, it is padded to the right, so what we thought was `0a`, actually turns into `a00000000000000000000000000000000000000000000000000000000000000`. So what we need to do is pad our `0x0a` with 31 bytes to the left making it `0x000000000000000000000000000000000000000000000000000000000000000a`. There you go, that is our answer!

# Puzzle #7

You know the drill. Give the puzzle a shot and then come back to see the full solution / explanation.

```js
############
# Puzzle 7 #
############

00      36        CALLDATASIZE
01      6000      PUSH1 00
03      80        DUP1
04      37        CALLDATACOPY
05      36        CALLDATASIZE
06      6000      PUSH1 00
08      6000      PUSH1 00
0A      F0        CREATE
0B      3B        EXTCODESIZE
0C      6001      PUSH1 01
0E      14        EQ
0F      6013      PUSH1 13
11      57        JUMPI
12      FD        REVERT
13      5B        JUMPDEST
14      00        STOP

? Enter the calldata: 
```

First things first, we can see `CALLDATASIZE` and know that we will need to pass in calldata with a specific size to solve this puzzle. We will take note of this and come back later. After the size of the calldata is pushed to the stack, there is `PUSH1 00`, and `DUP1` making our stack at this point look like this.

```js
[0 0 calldata_size 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Next we encounter the [CALLDATACOPY instruction](). This instruction copies the input data from the transaction and saves it into memory. This opcode expects three elements at the top of the stack which are `[destOffset offset size]`, in this order. `destOffset` is the byte offset in the memory where the result will be copied. We haven’t talked much about memory at this point and if you want to learn more you can [read about it here](). The abbreviated version is that there is temporary storage allocated during the execution of a function and the `destOffset` tells the program what slot in memory to store the data in. The `offset` dictates where to start copying the calldata from (just like `CALLDATALOAD` does in the last example) and the `size` tells the program how much of the byte sequence to store in memory. During this process, all three of the top elements on the stack are consumed.

With all of this known, let’s revisit our current stack.


```js
[0 0 calldata_size 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

When the `CALLDATALOAD` instruction executes, it will store calldata in memory slot `0`, starting at byte `0`, and storing the size of the entire calldata. Our resulting stack after this instruction looks like this.

```js
[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Immediately after, `CALLDATASIZE` `PUSH1 00` `PUSH1 00` are all executed, making the stack look like we had it before.

```js
[0 0 calldata_size 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Next we are introduced to another new opcode, the [CREATE instruction](). This instruction creates a new account (ie. Contract or EOA). This instruction expects the following three values on the top of the stack `[value offset size]` in this order. `value` is the amount of wei to send to the new account. `offset` is the slot in memory that holds the data we want to send the new account. `size` is the size of the data, which lets us know how much data to read from memory. When the `CREATE` instruction executes, all three values are consumed and the address that the account was deployed to is pushed to the top of the stack. After this opcode executes, our stack looks like this. 

```js
[address_deployed 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Next, we come across the [EXTCODESIZE instruction]() which expects an address on the top of the stack and pushes the size of the code at that address in return. The address at the top of the stack is consumed in this process. Since the size of the code at the address is the same as the `calldata_size` from before, we can note that this number will be back on the top of the stack. See if you can figure out the rest of the puzzle from here.


Now that we understand all of the previous information, we `PUSH1 01` making our stack look like this.

```js
[01 address_code_size 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Directly after, the `EQ` instruction is executed, checking if the top two values are equal and pushing the result on the stack. From there `PUSH1 13` and `JUMPI` are used to get us to the `JUMPDEST`!

So coming all the way back to the beginning of the puzzle, we will need to enter calldata such that the code size is equal to 01 byte! To understand this, we can look at the playground example from the [EXTCODESIZE instruction](https://www.evm.codes/#3b). Here is what the example looks like.

```js
// Creates a constructor that creates a contract with 32 FF as code
PUSH32 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
PUSH1 0
MSTORE
//Opcodes to return 32 ff
PUSH32 0xFF60005260206000F30000000000000000000000000000000000000000000000
PUSH1 32
MSTORE

// Create the contract with the constructor code above
PUSH1 41
PUSH1 0
PUSH1 0
CREATE // Puts the new contract address on the stack

// The address is on the stack, we can query the size
EXTCODESIZE
```

When this code is run, it returns a value of `ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff` which is 32 bytes. If you want, you can check out the opcodes for the [deployed contract code here](https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x60016000526001601f&codeType=Bytecode&code=%2736600080373660006000F0600080808080945AF1600014601B57FD5B00%27_). If we change the return size to 16 bytes instead of 32 bytes, the `EXTCODESIZE` will be `10` which is 16 bytes in hexadecimal.


# Puzzle #8

Welcome to the eigth puzzle. Lets take a look at what is in store.

```js
############
# Puzzle 8 #
############

00      36        CALLDATASIZE
01      6000      PUSH1 00
03      80        DUP1
04      37        CALLDATACOPY
05      36        CALLDATASIZE
06      6000      PUSH1 00
08      6000      PUSH1 00
0A      F0        CREATE
0B      6000      PUSH1 00
0D      80        DUP1
0E      80        DUP1
0F      80        DUP1
10      80        DUP1
11      94        SWAP5
12      5A        GAS
13      F1        CALL
14      6000      PUSH1 00
16      14        EQ
17      601B      PUSH1 1B
19      57        JUMPI
1A      FD        REVERT
1B      5B        JUMPDEST
1C      00        STOP

? Enter the calldata:
```

This one might look more daunting but it is actually pretty simple. First we see a very similar `CALLDATASIZE PUSH1 00 DUP1 CALLDATACOPY CALLDATASIZE PUSH1 00 PUSH1 00 CREATE` which, just like the previous puzzle, creates a new contract from the calldata that you pass in. So right from the start, we know that we will have to enter bytecode for a contract to solve the puzzle. Lets take a quick mental note of what the stack looks like at this point. Since the `CREATE` instruction consumes the top three stack values and pushes the address that the new account was deployed to, our stack now looks like this.


```js
[address_deployed 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```


The next 5 instructions all relate to the [CALL instruction](https://www.evm.codes/#f1). This instruction creates a new sub context and execute the code of the given account, then resumes the current one. Note that an account with no code will return success as true. In plain english, the `CALL` instruction is used to interact with another contract. This opcode expects the stack to have a few values a the top of the stack
`[gas address value argsOffset argsSize retOffset retSize]`, in this order. Lets walk through each of the arguments one by one. `gas` is the amount of gas that will be sent with the message call. `address` is the address that the message will be sent to. `value` is the amount of wei that will be sent with the message. `argsOffset` is the location in memory within the current context (ie. the msg.sender) that will be used as calldata for the message call. `argsSize` is the size of the calldata to send with the message call. `retOffset` is the location in memory within the current context where the return value from the call will be stored. Finally, `retSize` is the size of the return value that will be stored in memory.

Now let's take a look at the puzzle again. The next four opcodes are `PUSH1 00 DUP1 DUP1 DUP1 DUP1`, which makes the stack look like this.


```js
[0 0 0 0 0 address_deployed 0 0 0 0 0 0 0 0 0 0]
```

Then we see the [SWAP5 instruction](). This instruction swap 1st and 6th stack items. There are SWAP instructions for all positions in the stack (`SWAP1`-`SWAP16`). In this case, we `SWAP5` exchanges `0` with `address_deployed` making our stack now in the correct order to match `[gas address value argsOffset argsSize retOffset retSize]`. Here is what our stack looks like now. 


```js
[address_deployed 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

Then we execute the `CALL` instruction, which returns `0` if the sub context reverted and `1` if it was a success. After the `CALL` instruction we can see a `PUSH1 00 EQ` meaning that we need `CALL` to push a `0` onto the stack. Go ahead and give the rest of the puzzle a shot, then feel free to come back to see the rest of the solution.


Ok, so now we know that the `CALL` instruction needs to return `0` which means we need to enter calldata that causes `CALL` to fail. This one is a little bit tricky, to get `CALL` to fail, there are a few ways. Some ways that it can fail are: there is not enough gas, there are not enough values on the stack or if the `CALL` expects a specific return size and gets a different one. In our case, we can't change the values on the stack or the gas, so we will manipulate the third case. Since `CALL` expects a return size of `0` for this puzzle, we need to enter calldata that when executed via `CALL`, the return size is > `0`. While you can enter any bytecode that returns a value when executed, we will use a simple byte code sequence that returns `01`. Here is what that looks like in opcodes: `PUSH1 01 PUSH1 00 MSTORE PUSH1 01 PUSH1 1F RETURN`, and encoded to hexadecimal, `0x60016000526001601ff3`. There you go, `0x60016000526001601ff3` is our answer!



# Puzzle #9
We are in the home stretch, let's take a look at puzzle #9. This puzzle adds one more layer of complexity, requiring you to enter both a callvalue and calldata to solve the puzzle.

```js
############
# Puzzle 9 #
############

00      36        CALLDATASIZE
01      6003      PUSH1 03
03      10        LT
04      6009      PUSH1 09
06      57        JUMPI
07      FD        REVERT
08      FD        REVERT
09      5B        JUMPDEST
0A      34        CALLVALUE
0B      36        CALLDATASIZE
0C      02        MUL
0D      6008      PUSH1 08
0F      14        EQ
10      6014      PUSH1 14
12      57        JUMPI
13      FD        REVERT
14      5B        JUMPDEST
15      00        STOP

? Enter the value to send: (0)

```

We are already familiar with the first two opcodes so we can know that after the `PUSH1 03` instruction, our stack looks like this.

```js
[03 calldata_size 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

The [LT instruction](https://www.evm.codes/#10) runs a comparison on the first two stack values to see if the first stack element is less than the second stack element. If `LT` evaluates to true, `1` is pushed on the stack, otherwise `0` is pushed onto the stack. The two values used in the comparison are consumed in this process. For the sake of the example, let's say that the `CALLDATASIZE` is 4 bytes, so `LT` will push `1` as a result. 

Since the `LT` instruction evaluated to true, the code then jumps to the `JUMPDEST` at instruction `09`. Following the jump, `CALLVALUE` and `CALLDATASIZE` are pushed onto the stack and `MUL` multiplies them together, consuming the values in the process. `PUSH1 08` pushes `08` to the stack and then `EQ` checks if the result of `MUL` equals `08`, consuming the values in the process. `EQ` needs to push a `1` to the stack to enable `JUMPI` to get us to the end of the puzzle.

With all this information, we now know that we need to pass in calldata such that the `CALLDATASIZE` is greater than 3 bytes, and the product of `CALLDATASIZE` and `CALLVALUE` is `08`. 

With some quick math, we can use any combination of integers that evaluate to 8 when multiplied together, as long as `CALLDATASIZE` is greater than 3 bytes. For the walkthrough, we will enter `0x00000001` as the calldata and `2` as the callvalue.


# Puzzle #10
Here it is, the final puzzle. Let's jump in.

```js
#############
# Puzzle 10 #
#############

00      38          CODESIZE
01      34          CALLVALUE
02      90          SWAP1
03      11          GT
04      6008        PUSH1 08
06      57          JUMPI
07      FD          REVERT
08      5B          JUMPDEST
09      36          CALLDATASIZE
0A      610003      PUSH2 0003
0D      90          SWAP1
0E      06          MOD
0F      15          ISZERO
10      34          CALLVALUE
11      600A        PUSH1 0A
13      01          ADD
14      57          JUMPI
15      FD          REVERT
16      FD          REVERT
17      FD          REVERT
18      FD          REVERT
19      5B          JUMPDEST
1A      00          STOP

? Enter the value to send: (0) 
```