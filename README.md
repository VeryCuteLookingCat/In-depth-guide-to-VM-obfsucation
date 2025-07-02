# Introduction
The core concept behind a VM-based obfuscator is a custom virtual CPU that interprets instructions at runtime. This approach is the most basic form of obfuscation. Many advanced obfuscators often add additional layers of protection. Some advanced techniques include: Control flow obfuscation, Data encryption, Packing, Polymorphism, Anti tampering and Mutations. I will explain the full extent of my knowledge on this subject.
## The basics
A VM-based obfuscator may have different components to it depending on the language. For instance, lua it has 2 main parts: the CPU and the deserializer. For something like assembly (ASM), the obfuscator usually uses a virtual CPU, although it might also need to decrypt virtualized code. The main goal of this specific obfuscation is to convert bytecode into custom, virtual instructions. 
## The CPU
What exactly does the virtual CPU do? Well, the virtual CPU is in charge of mimicing a real cpu, but for the specific obfuscators bytecode. It processes the custom instructions into real cpu instructions. In lua they use [OPCodes](https://www.lua.org/source/5.1/lopcodes.h.html) to symbolise what action the cpu should do. They store any values needed for later use into the 'Stack' which is typically a table in basic obfusactors. Let's say you pass in OPCode 0, with instructions 0, 1 and 2. The virtual CPU will find what OPCode 0 coorelates to ( for this example, addition ) and execute it. The output will push the sum of 1 and 2 into stack spot 0.
```lua
function CPU(instructions)
--[[
instructions = {
  {
  0, Opcode
  0, Stack position
  1, Argument #1
  2, Argument #2
  }
}
]]
  local Stack = {}
  for index, inst in pairs(instructions)
      local OPCode = inst[0]
      if(OPCode == 0) then
         Stack[inst[1]] = inst[2] + inst[3] -- Stack[0] = 1+2
      end
  end
end
```
