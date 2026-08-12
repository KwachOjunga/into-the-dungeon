Features of ARM isa.

Arm instruction set has seven operating modes.
Six are priviledged.

- `abort` - entered when there is failed attempt to access memory
- `fast interrupt request`
- `interrupt request`
- `supervisor` - the mode that the processor is in after reset and is generally the mode that an operating system kernel operates in. 
- `system` - special version of `user` mode that allows read and write access to cpsr
- `undefined` - enters when an unknown instruction in the implementation is used.

The remaining mode is the unpriviledged `user` mode.

User, Protected etc.

```text
                        Data                  ____________________
                        ^ | |--------------->|   Instruction      | 
          ______________| |                  |    Decoder         |
                          |                  |____________________|
                          v
                    __________________
                    |    Sign        |
                    |   Extend       |
                    |________________|
                            |
                            |
                            v
                  ++++++++++++++++++++++++++++++++
                  |             r0-r15            |<---------------------
                  |         Register File         |------               |
                  +++++++++++++++++++++++++++++++++      |              |
                                                         |              |
                                                         |              |
                      ++++++++++++++++++++       |  |    |              |
                      |   Barrel Shifter  |      v  v    v              |
                      +++++++++++++++++++++     ________________        |
                          |                     |     MAC       |       |
                          |  `N`                |_______________|       |
                          v                             |               |
              ___________________________               |               |
              |                         |               v               |
              |       ALU               |<------------------------------|


```


There are 18 registers exposed to the programmer by default - `user` mode.
16 data registers and 2 processor registers.

Data registers : r0-r15
r13 - stack pointer   (sp)
r14 - link register   (lr)
r15 - program counter (pc)

Processor registers: 
cpsr - current program status register
spsr - saved program status register

What does it mea for registers to be banked?

Which registers are visible to the programmer depend upon the current mode of the processor.
