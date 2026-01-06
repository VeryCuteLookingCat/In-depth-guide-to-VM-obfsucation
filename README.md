# Introduction
> All examples are for educational purposes to understand obfuscation from a defensive perspective.

The core concept behind a VM-based obfuscator is a custom virtual CPU that interprets instructions at runtime. Many advanced obfuscators often add additional layers of protection. Some advanced techniques include: Control flow obfuscation, Data encryption, Packing, Polymorphism, Anti tampering and Mutations. I will explain the full extent of my knowledge on this subject.
## The basics
A VM-based obfuscator may have different components to it depending on the language. For instance, lua has 2 main parts: the CPU and the deserializer. For something like assembly (ASM), the obfuscator usually uses a virtual CPU, although it might also need to decrypt virtualized code. The main goal of this specific obfuscation is to convert bytecode into custom, virtual instructions. 
## The CPU
What exactly does the virtual CPU do? The virtual CPU mimics the behavior of a real cpu, but interprets the obfuscators custom bytecode instead of the programs. It translates the custom instructions into real CPU operations. In Lua they use [OPCodes](https://www.lua.org/source/5.1/lopcodes.h.html) to represent what action the cpu should perform. They store any values needed for later use into the 'Stack' ( This is a simplification, but in practice it behaves more like a register file. ) which is typically a table in basic obfuscators. Let's say you pass in OPCode 0, with instructions 0, 1 and 2. The virtual CPU will find what OPCode 0 correlates to ( for this example, addition ) and execute it. The output will push the sum of 1 and 2 into stack spot 0. This is all done at runtime, so a poorly coded obfuscator could cause a program to run slower. The instructions passed are similar to writing your own mini language inside of another.
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
A deserializer is the main component in an obfuscated script that converts custom bytecode into executable instructions. The custom bytecode is typically encrypted, modified, and mutated by obfuscators to make reverse engineering more difficult. This decoding process often involves complex mathematical operations and markers embedded within the bytecode. Typically, the deserializer reads an initial bit or group of bits that indicate the size or type of the next instruction. It then reads the required number of subsequent bits to reconstruct the instruction, before moving to the next marker. This process repeats until the entire bytecode stream has been processed.
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
## Control Flow Obfuscation
Control flow obfuscation is usually one of the most annoying parts to reverse engineer. It tampers with the linear flow of code to add chaos. Instead of having the CPU process a list of opcodes in order, it swaps the order of the code around. Think of taking a block of code, putting it in a bag, tossing it around, and the contents become obfuscated.

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
All advanced obfuscators use some form of anti-debugging. These techniques can be more diverse and harder to categorize than others. There are many different ways to check for debuggers. The most basic example would be IsDebuggerPresent. Another very common check is to scan through all processes to see if any match a known registry of debuggers, but this could be slow and inefficient. But for those interested, some advanced techniques would check for hardware breakpoints, using timers to check for discrepancies, and making SEH (Structured Exception Handling) traps.
## Packing ( Binaries )
Packing is typically performed after obfuscation. In Lua, It converts the entire program into a single compressed or encoded string that is later unpacked and executed at runtime. But in the context of binaries, it converts the entire program into compressed sections ( ex: .upx ). The unpacker decompresses the packed section into memory and transfers RIP (Register Instruction Pointer) to it. (The RIP is a CPU register that holds the address of the next instruction to execute) When transferring the RIP into the unpacked memory, The processor executes the code in memory. The primary goal of packing is to reduce size, hide the internal structure, and make static analysis significantly more difficult. This is done via an unpacker that replaces 'entry' or 'WinMain'. The unpacker decodes the string into the original program and executes it. The most popular packer is [UPX](https://github.com/upx/upx). In short packing helps with: reducing file size, hiding program structure, and avoid static analysis and naive static signatures.
## Mutations
Mutations can affect every part of an obfuscated binary or script. They often provide the highest level of security among obfuscation techniques. Mutations involve maintaining a pool of different variations of CPUs, deserializers, bytecode formats, and other components. This approach introduces chaos, randomness, and unpredictability into the obfuscated output. For example, Variation #1 of the CPU could be paired with Variation #6 of the deserializer, ensuring that no two outputs share any form of similarity.
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
Every obfuscated build could generate a different version of the same logic. This interferes with static and ( In select few cases) runtime analysis, It changes the function instructions and doesn't retain consistency across builds.
## Polymorphism vs Mutations
In this educational context, polymorphism and mutations are treated as distinct layers. Polymorphism refers to localized, semantic equivalent transformations of individual functions or logic blocks. Mutations refer to structural modification of the virtual machine itself, including instruction encoding, dispatch logic and execution flow.
Polymorphism – Breaks function-level signatures while preserving VM invariants; easier to cluster behaviorally.
Mutations – Alters VM structure, breaking tooling assumptions and CFG recovery; analysts must relearn execution schematics.
## Why?
Why Obfuscate? Obfuscators are used in many real world scenarios, either for good or bad. Many legitimate examples use obfuscators for: DRM ( Digital Rights Management ) for games/programs, Anti-Cheats to hide integrity and functionality, Protection of IP ( Intellectual Property ), and more. But an illegitimate example could be Malware ( Malicious Software ).
# Blue Teaming

## What's the point?
VM-Based obfuscation might be used in reasons listed in "Why?" but it's disproportionately used in malware. If an unsigned ( more often than signed ) program uses VM-Based obfuscation, this should set off major red flags during static analysis. To reiterate: Obfuscation is not malicious by default, but it **raises analysis cost**, which itself is a risk signal. So what is the point? Reason #1:
### polymorphism between builds 
This means that no two builds are alike, the code at a bytecode level is different from 2 builds, they might contain similar structure but no simple PE Header fingerprinting, file hashing, or signature scanning will be effective in detecting the static binary. During static analysis, the only key detail you will notice between builds depending on the level of sophistication of the obfuscator is the structure. But extremely advanced obfuscators will have mutations that will reorganize the structure of the bytecode execution.
### Hidden Functionality
Many ( But not all ) obfuscators will throw in anti-debugging as mentioned in "Anti Debugging", but the reason anti debugging exists is to hide true functionality. The most common scenario that will occur when debugging this form protection is a program exit. But in extreme cases, malware might switch functionality to look more legitimate than it seems. What this translates to: Malware might swap functionality, or stop execution if it detects a debugger. Some pipelines that malware takes when debugging is detected might be: fake legitimate activity, delay payloads, or environmental-based logic swaps. But obfuscation might also hide functions from the IAT table and create references at runtime, This prevents IAT fingerprinting.
## So, How do I handle obfuscated exploits?
This question heavily depends on a lot of factors: The level of the obfuscator, the publicity of it, the depth of the obfuscator, and many more. The best and simplest way to analyze obfuscated malware is runtime analysis, Using websites such as VirusTotal ( Not recommended, there are way better options that provide more detailed reports ), Any.Run, Triage, or hybrid analysis to name a few. This is the best option for implementing YARA Rules to prevent any spread temporarily ( Assuming that the infected system was properly quarantined and a fragile fix was needed until proper investigation was done ). Any YARA rules should be targeting: In-memory decoded configuration data, shellcode stubs or unpacking loops, abnormal PE sections mapped at runtime, and API hashing routines. **YARA is a stopgap, not a solution** if you're planning on writing YARA rules **never** (yes, there are some EXTREMELY rare edge cases) do the following: write rules on packed sections, hash the file, overfit strings.
### Target the weak link the chain
Obfuscation protects implementation details, not attacker intent. Defensive strategies should focus on intent-bearing artifacts. To prevent different strains of obfuscated malware, target the weakest link. No matter the protection on the binary ALL malware have these common weak links:
### Persistence
Nearly all malware (excluding wipers and one-shot payloads) need persistence in order to maintain control over the system. Using the analysis results from earlier, identify and remove the persistence mechanism of the malware. The malware can be adding/modifying the registry, making scheduled tasks, dropping a payload into an autorun folder, but persistence = control.
### Communication
Whether it's usermode malware using pipenames to communicate with drivers or send data back to its C2 server, Most malware needs to communicate. In a majority of cases, these communication methods are static or hardcoded. Adding rules to detect these static names or ips could be a decent solution for one specific strain. But analyzing the structure of the messages and scanning for consistencies across content, or destination is more effective. Even when endpoints rotate, protocol structure and message intent often remain consistent.
### Mutexes
A lot of malware strains just leave mutex names inside of memory, this is extremely easy to detect and often times static. This is mostly seen in low‑sophistication malware, but when present, mutexes provide extremely high‑confidence detection with minimal cost.
### Suspicious PE Headers
I'm well aware the amount of times I've stated "This breaks static signature scanning" but the earliest signs of obfuscation are still linked to the PE headers. A common similarity across a vast majority of obfuscated malware is that they contain a PE header section with an unusual name and a high entropy. High entropy in a PE section (8+) is unusual compared to typical programs (4–6). Alone, it isn’t proof of obfuscation, but when combined with short/unusual section names or runtime anomalies, it’s a strong signal. Signature detection via PE headers is very very possible and the best project I've seen implement this is [Detect It Easy](https://github.com/horsicq/Detect-It-Easy/tree/master/db/PE).
