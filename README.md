# Introduction
The core concept behind a VM-based obfuscator is a custom virtual CPU that interprets instructions at runtime. This approach is the most basic form of obfuscation. Many advanced obfuscators often add additional layers of protection. Some advanced techniques include: Control flow obfuscation, Data encryption, Packing, Polymorphism, Anti tampering and Mutations. I will explain the full extent of my knowledge on this subject.
## The basics
A VM-based obfuscator may have different components to it depending on the language. For instance, lua has 2 main parts: the CPU and the deserializer. For something like assembly (ASM), the obfuscator usually uses a virtual CPU, although it might also need to decrypt virtualized code. The main goal of this specific obfuscation is to convert bytecode into custom, virtual instructions. 
## The CPU
What exactly does the virtual CPU do? The virtual CPU mimics the behavior of a real cpu, but interprets the obfuscators custom bytecode instead of the programs. It processes the custom instructions into real cpu instructions. In Lua they use [OPCodes](https://www.lua.org/source/5.1/lopcodes.h.html) to represent what action the cpu should perform. They store any values needed for later use into the 'Stack' which is typically a table in basic obfuscators. Let's say you pass in OPCode 0, with instructions 0, 1 and 2. The virtual CPU will find what OPCode 0 correlates to ( for this example, addition ) and execute it. The output will push the sum of 1 and 2 into stack spot 0. This is all done at runtime, so a poorly coded obfuscator could cause a program to run slower. The instructions passed are similar to writing your own mini language inside of another.
```lua
function CPU(instructions)
--[[
instructions = 
{
  {
  0, Opcode
  0, Stack position
  1, Argument #1
  2, Argument #2
  }
}
]]
  local Stack = {}
  for index, inst in pairs(instructions) do
      local OPCode = inst[1]
      if(OPCode == 0) then
         Stack[inst[2]] = inst[3] + inst[4] -- Stack[0] = 1 + 2
      end
  end
end
```
It is very similar in the context of ASM.
## Deserializer ( LUA )
A deserializer is the main component in an obfuscated script that converts custom bytecode into executable instructions. The custom bytecode is typically encrypted, modified, and mutated by obfuscators to make reverse engineering more difficult. This decoding process often involves complex mathematical operations and markers embedded within the bytecode. Typically, the deserializer reads an initial bit or group of bits that indicate the size or type of the next instruction. It then reads the required number of subsequent bits to fully reconstruct the instruction before moving on to the next instruction bit or marker. This process repeats until the entire bytecode stream has been processed.
## Structure Graph ( LUA )
```
-------------------
|   Deserializer  |
|     Parses      |
| Custom Bytecode |
| Decodes Opcodes |
-------------------
         |
         v
-------------------
|   Virtual CPU   |
| Executes Opcodes|
|Manipulates Stack|
| Mimics Real CPU |
-------------------
```
# Advanced Techniques
## Mutations
Mutations can affect every part of an obfuscated binary or script. They often provide the highest level of security among obfuscation techniques. Mutations involve maintaining a pool of different variations of CPUs, deserializers, bytecode formats, and other components. This approach introduces chaos, randomness, and unpredictability into the obfuscated output. For example, Variation #1 of the CPU could be paired with Variation #6 of the deserializer, ensuring that no two outputs share any form of similarity.
## Control Flow Obfuscation
Control flow obfuscation is usually one of the hardest parts to reverse engineer. It tampers with the linear flow of code to add chaos. Instead of having the CPU process a list of opcodes in order, it swaps the order of the code around. Think of taking a block of code, putting it in a bag, tossing it around, and the contents become obfuscated.

This is possible by having a loop that keeps track of the current iteration. Each time the iteration count changes, a new segment of code is run. For example:
```lua
local inst = 0
local TestVariable = 0
while true do
    if(inst == 0) then
        TestVariable = 2
        inst = 1
    end
    if(inst == 2) then
        TestVariable = TestVariable + 2
        inst = -1
    end
    if(inst == 3) then
      TestVariable = -1
      inst = 2
    end
    if(inst == 1) then
      TestVariable = TestVariable + 2
      inst = 3
    end
    if(inst == -1) then
        break
    end
end

-- TestVariable is equal to 1
--[[
Cleaned:
local TestVariable = 0
TestVariable = 2
TestVariable = TestVariable + 2
TestVariable = -1
TestVariable = TestVariable + 2
]]
```
## Anti Debugging
All advanced obfuscators use some form of anti-debugging. These techniques can be more diverse and harder to categorize than others. There are many different ways to check for debuggers. The most basic example would be _CheckForMyDebugger in MSVC. Another very common check is to scan through all processes to see if any match a known registry of debuggers, but this could be slow and inefficient. But for those interested, some advanced techniques would check for hardware breakpoints, using timers to check for discrepancies, and making SEH (Structured Exception Handling) traps.
## Packing ( Binaries )
Packing is typically performed after obfuscation. In Lua, It converts the entire program into a single compressed or encoded string that is later unpacked and executed at runtime. But in the context of binaries, it converts the entire program into compressed sections ( ex: .upx ). The unpacker decompresses the packed section into memory and transfers RIP (Register Instruction Pointer) to it. (The RIP is a CPU register that holds the address of the next instruction to execute) When transfering the RIP into the unpacked memory, The processor executes the code in memory. The primary goal of packing is to reduce size, hide the internal structure, and make static analysis significantly more difficult. This is done via an unpacker that replaces 'entry' or 'WinMain'. The unpacker decodes the string into the original program and executes it. The most popular packer is [UPX](https://github.com/upx/upx). In short packing helps with: reducing file size, hiding program structure, and avoid static analysis and AV signatures.
## Polymorphism
Polymorphism is often confused with mutations, as they share similar goals. The only difference being: Polymorphism changes the structure of local functions inside of the binary, Mutations change the structure of the vm.
Let's say you have function add(x, y), Polymorphism will convert that function into something harder to read:
```cpp
int add(int a, int b) {
  return a + b;
}
```
An obfuscator will use polymorphism to turn it into something like this:
```cpp
int add(int a, int b) {
  int x1 = a;
  int x2 = b;
  while(x2 != 0) {
    x1++;
    x2--;
  }
  return x1;
}
```
Every obfuscated build could generate a different version of the same logic. this breaks static analysis and signature based detection.
## Why?
Why Obfuscate? Obfuscators are used in many real world scenarios, either for good or bad. Many legitimate examples use obfuscators for: DRM ( Digital Rights Managment ) for games/programs, Anti-Cheats to hide integrity and functionality, Protection of IP ( Intellectual Property ), and more. But an illegitimate example could be Malware ( Malicious Software ).

