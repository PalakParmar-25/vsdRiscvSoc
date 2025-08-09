
## F. Instruction decoding (integer type)
 
| Instruction | Opcode | rd | rs1 | rs2 | funct3 | funct7 | Binary | Description |
|-------------|--------|----|----|-----|--------|--------|--------|-------------|
| `addi sp,sp,-64` | 0010011 | x2 | x2 | - | 000 | - | 11111100000000010000000100010011 | sp = sp + (-64) |
| `add a5,a5,a4` | 0110011 | x15 | x15 | x14 | 000 | 0000000 | 00000000111001111000011110110011 | a5 = a5 + a4 |
| `bge a5,a4,103a2` | 1100011 | - | x15 | x14 | 101 | - | 00000000111001111101000011100011 | Branch if a5 >= a4 |
| `lui a5,0x1c` | 0110111 | x15 | - | - | - | - | 00000000000000011100011110110111 | Load upper immediate: a5 = 0x1c000 |
| `ld a1,0(a5)` | 0000011 | x11 | x15 | - | 011 | - | 00000000000001111011010110000011 | Load doubleword: a1 = *(a5+0) |

#### Register Mapping Reference
- x2 = sp (stack pointer)
- x11 = a1 (argument/return value 1) 
- x14 = a4 (argument 4)
- x15 = a5 (argument 5)

#### Instruction Format Types Represented
- **I-type**: `addi`, `ld` (immediate and load operations)
- **R-type**: `add` (register-register operations)
- **B-type**: `bge` (branch operations)
- **U-type**: `lui` (upper immediate)
