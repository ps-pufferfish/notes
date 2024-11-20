### Coder's Log
[26/06/2024] Do some tests for load instruction
	Plans next to be able to decode every load instruction from raw byte
### Terminology Reference 
- T-Cycle: clock cycle, one clock tick
- M-Cycle: Machine cycle, cpu cycle, 4 clock ticks
- IR: Instruction Register

### Current concerns
- lot of semi duplicated code in decode stuff
- how to get better test error messages
### Old Recent thoughts
 - Maybe Kotlin could be cool?
	 - It has infix operators that, among other things, define operations for Short, Byte
	 - Note to self, ask mike what he thinks about it. Could be cool to get kotlin experience anyway.
#### FUCK!
- Instruction register IR is a thing, kinda already got that, no sweat
- forgot about the IE (interrupt enable register)
- 16 bit data bus is output only from the cpu
- 8 bit bidirectional data bus from the cpu is also there
- ALU is 8 bit only. Noice
- there's a 16 bit IDU (increment/decrement unit)
- Timings are wacky
	- Gameboy parelellises fetch/execute
	- fetch/decode of next instruction happens during last M-Cycle of current instruction
- Parallel Streams of work:
	- Clock
	- Address Bus
	- Data Bus
	- IDU operation
	- ALU opperation
	- Misc operation
		- 16 bit write to register after 2 byte loads from mem
		- condition check on memory
		- IME <- 1 ? (RETI)
		- PC <- addr (RST)
		- IME <- 0 (HALT)
		- IME <- 1 (IME)
![[Pasted image 20240822081639.png|400]]
### Common instruction actions
  - Register access / indirect memory access
### Design Decisions
 - CPU cycle
	 - Fetch
		 - load 16bits at PC into cpu
	 - Decode
		 - Convert into semantic operation
		 - Operation doesn't have any behaviour on it, just a description really
		 - e.g. ADD(A, B)
		 - Add implmented generally, instances are singletons
		 - Some sort of DI might be good. Want to learn how it works
	 - Execute
		 - Get result of operation on the data
		 - Possibly needs extra steps for instructions that have 16bit operands
		 - 8 bit operands are already loaded
	 - Store result
		 - Operation has a destination
		 - Store the operation result in that destination
	 - Increment PC
		 - Not sure when this should happen, not sure if it matters, probably not. Probably happens before store result because some instructions store into PC.
		 - PC += 1
		 - if 8 bit operand, PC += another 1
		 - if 16 bit operand, PC += another 2
 - Might implement a clock to regulate timing
	 - 4 clock cycles per cpu cycle
	 - Can have cpu 'increment' it after fetch/decode/execute/store, blocking op untill time is right
	 - initially don't need it, but keep in mind.
	 - More investigation needed for larger cycle ops, esp PUSH/POP etc...
	 - may be useful for testing. Can check state mid cpu cycle
	 - see [[#FUCK!]]

### Testing
 - Test Decode against online json.
	 - need decent toString on all operations
	 - if we implement a compiler, can do round trip encode/decode
 - Need to test operation implementation independent of fetch/decode
	 - If clock is implemented, test cycles are correct
	 - Potentially separate out arithmetic ops from others


```
(a xor b) xor c_in = res
X xor c = res
c = (¬x and res) or (¬res and x)
c = x xor res
c_in = (a xor b) xor res


(a and b) or ((a xor b) and c) = c_out

c_out = (a and b) or ((a xor b) and ((a xor b) xor res))
c_out = (a and b) or ((a xor b) and ¬res)





0000_0000
0000_0001  ->  1111_1110 + 1
---------
1111_1111




res = ¬((a xor b) xor c_in)  // res = a - b (- c_in)1


adj = 
	1 if Carry and positive
	-1 if not Carry and negative
	0 otherwise


11111111
10101010
10101001
```