# LuckyCompilerHDR — Two-Pass Assembler with GUI

Version 1.0

A Python-based two-pass assembler for a custom 16/32-bit instruction set, bundled with a Tkinter GUI editor. Write assembly source code in the editor, assemble it to binary, and optionally save the output to a `.bin` file.

---

## Features

- **Two-pass assembly**: Pass 1 builds a symbol table; Pass 2 resolves labels and encodes instructions.
- **Peephole optimizer**: Automatically removes redundant `mov` instructions before encoding.
- **GUI editor**: Load, edit, and save `.asm` source files; view assembler output in-app.
- **Binary output**: Save assembled machine code directly to a binary file.

---

## Files

| File | Description |
|---|---|
| `cat'sluckycompilerhdr.py` | Full-featured assembler + GUI (auto-loads `sample.asm` if present) |
| `luckycompu-4k.py` | Compact version of the assembler + GUI |

---

## Requirements

- Python 3.x
- `tkinter` (included with standard Python on Windows/macOS; on Linux: `sudo apt install python3-tk`)

---

## Running

```bash
python "cat'sluckycompilerhdr.py"
# or
python luckycompu-4k.py
```

---

## Instruction Set

### Registers

| Name | Code |
|------|------|
| `rax` | `0x0A` |
| `rbx` | `0x0B` |
| `rcx` | `0x0C` |
| `rdx` | `0x0D` |

### Instruction Formats

| Format | Layout (bits) | Size |
|--------|---------------|------|
| Register–Register (RR) | `Opcode[8] \| SrcReg[4] \| DestReg[4]` | 2 bytes |
| Register–Immediate (RI) | `Opcode[8] \| DestReg[4] \| 0000[4] \| Immediate[16]` | 4 bytes |
| Jump | `Opcode[8] \| 00000000[8] \| TargetAddr[16]` | 4 bytes |

### Opcodes

| Mnemonic | Opcode | Operands | Description |
|----------|--------|----------|-------------|
| `add`    | `0x00` | `dest, src` (RR) | `dest = dest + src` |
| `sub`    | `0x01` | `dest, src` (RR) | `dest = dest - src` |
| `mul`    | `0x02` | `dest, src` (RR) | `dest = dest * src` |
| `div`    | `0x03` | `dest, src` (RR) | `dest = dest / src` |
| `cmp`    | `0x04` | `dest, src` (RR) | Set flags from `dest - src` |
| `mov`    | `0x05` | `dest, src` (RR) | `dest = src` |
| `mov`    | `0x10` | `dest, imm` (RI) | `dest = immediate` |
| `add`    | `0x11` | `dest, imm` (RI) | `dest = dest + immediate` |
| `sub`    | `0x12` | `dest, imm` (RI) | `dest = dest - immediate` |
| `cmp`    | `0x13` | `dest, imm` (RI) | Set flags from `dest - immediate` |
| `mul`    | `0x14` | `dest, imm` (RI) | `dest = dest * immediate` |
| `div`    | `0x15` | `dest, imm` (RI) | `dest = dest / immediate` |
| `jmp`    | `0x20` | `label`    | Unconditional jump |
| `je`     | `0x21` | `label`    | Jump if equal (zero flag set) |
| `jne`    | `0x22` | `label`    | Jump if not equal (zero flag clear) |

Immediates are 16-bit values (`0`–`0xFFFF`), hex (`0x…`) or decimal. Negative values in the range `-32768`–`-1` are accepted and stored as their two's-complement 16-bit representation.

---

## Assembly Language Syntax

```asm
; This is a comment

start:              ; label definition
    mov rax, 10     ; load immediate 10 into rax  (RI form)
    mov rbx, 20     ; load immediate 20 into rbx
    add rax, rbx    ; rax = rax + rbx             (RR form)
    cmp rax, 0x1E   ; compare rax with 30
    je  done        ; jump to 'done' if equal
    jmp start       ; unconditional loop back

done:
    mov rcx, rax    ; copy result into rcx
```

Rules:
- Labels must start with a letter or `_` and contain only letters, digits, and `_`.
- A label may appear on its own line or before an instruction on the same line (`label: instruction`).
- Operands are separated by `,` or whitespace.
- Comments start with `;` and extend to the end of the line.
- Mnemonics and register names are case-insensitive.

---

## Compilation Pipeline

```
Source text
    │
    ▼
Pass 1 — Parse lines, collect labels (instruction-index based), build IR
    │
    ▼
Optimize — Peephole: remove redundant / duplicate mov instructions,
           remap label indices accordingly
    │
    ▼
Pass 2 — Compute byte addresses, resolve labels to byte offsets, encode binary
    │
    ▼
Binary output (.bin)
```

### Peephole Optimizations

1. **Redundant self-move**: `mov rax, rax` → removed.
2. **Consecutive duplicate move**: two identical back-to-back `mov dest, src` → second one removed.

After removal, all label indices are remapped to the surviving instruction positions before Pass 2 runs.

---

## GUI Usage

| Button | Action |
|--------|--------|
| **Load Source** | Open an `.asm` file into the editor |
| **Save Source** | Save the current editor content to a file |
| **New File** | Clear the editor |
| **Assemble** | Run the assembler on the editor content; output shown below |
| **Save Binary** | Re-assemble and write the binary output to a `.bin` file |

The status bar at the bottom shows the current operation result. Assembly messages (symbol table, optimization notes, errors) appear in the **Output** panel.
