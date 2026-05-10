# 4-Bit ALU in Verilog

A 4-bit Arithmetic Logic Unit (ALU) designed in Verilog HDL and verified
using a self-checking testbench. Built as an RTL design + verification 
project covering core digital design concepts relevant to VLSI roles.

Simulated on **Icarus Verilog 0.9.7** via [EDA Playground](https://edaplayground.com).

---

## What is an ALU?

An ALU is the computational core of every processor. It takes two operands
and a control signal (opcode), and performs the selected operation instantly
using pure combinational logic — no clock, no memory, no state.

---

## Operations Supported

| Opcode | Binary | Operation | Expression        | Notes                        |
|--------|--------|-----------|-------------------|------------------------------|
| 0      | `000`  | ADD       | `A + B`           | carry_out = 1 on overflow    |
| 1      | `001`  | SUB       | `A - B`           | two's complement subtraction |
| 2      | `010`  | AND       | `A & B`           | bitwise                      |
| 3      | `011`  | OR        | `A \| B`          | bitwise                      |
| 4      | `100`  | NOT       | `~A`              | B is ignored                 |


##Simulation Results:
PASS [1]  | opcode=000 A=3  B=5  | result=8  carry=0
PASS [2]  | opcode=000 A=9  B=7  | result=0  carry=1
PASS [3]  | opcode=000 A=0  B=0  | result=0  carry=0
PASS [4]  | opcode=001 A=8  B=3  | result=5  carry=1
PASS [5]  | opcode=001 A=5  B=5  | result=0  carry=1
PASS [6]  | opcode=001 A=2  B=4  | result=14 carry=0
PASS [7]  | opcode=010 A=12 B=10 | result=8  carry=0
PASS [8]  | opcode=010 A=15 B=0  | result=0  carry=0
PASS [9]  | opcode=010 A=15 B=15 | result=15 carry=0
PASS [10] | opcode=011 A=12 B=3  | result=15 carry=0
PASS [11] | opcode=011 A=0  B=0  | result=0  carry=0
PASS [12] | opcode=011 A=10 B=5  | result=15 carry=0
PASS [13] | opcode=100 A=0  B=x  | result=15 carry=0
PASS [14] | opcode=100 A=15 B=x  | result=0  carry=0
PASS [15] | opcode=100 A=10 B=x  | result=5  carry=0

Results: 15 PASSED, 0 FAILED

---

## Key Design Decisions

**Why `always @(*)`?**
The ALU is purely combinational — no clock involved. The `@(*)` 
sensitivity list means the block re-evaluates the moment any input 
changes, which is how combinational hardware actually behaves.

**Why two's complement for SUB?**
Instead of building a separate subtractor circuit, subtraction reuses 
the adder: `A - B = A + (~B) + 1`. This is exactly how real CPUs 
implement subtraction — one adder handles both ADD and SUB.

**Why does SUB carry_out behave differently?**
For subtraction, carry_out is an inverted borrow flag:
- `carry = 1` means A ≥ B (no borrow needed) 
- `carry = 0` means A < B (borrow occurred, result wraps)

This was caught and fixed during testbench verification — the initial
`{carry_out, result} = A + (~B) + 1` concat produced incorrect carry
behavior for SUB, fixed by computing result and carry_out separately.

**Why a `default` case?**
Opcodes 5, 6, 7 are unused. Without a default, undefined opcodes drive
the output to `X` (unknown) which can corrupt downstream logic and
break testbench `===` comparisons. The default forces a known zero state.

---

## How to Run

1. Go to [edaplayground.com](https://edaplayground.com) and create a free account
2. Create a new playground
3. Set simulator to **Icarus Verilog 0.9.7**
4. Check **"Open EPWave after run"** for waveform viewing
5. Paste `alu_4bit.v` into the design pane
6. Paste `tb_alu.v` into the testbench pane
7. Click **Run**

---

## Concepts Covered

- RTL combinational logic design in Verilog
- Module declaration, port mapping, `localparam`
- `case` statement as a hardware multiplexer
- Two's complement arithmetic
- Testbench methodology — stimulus, instantiation, self-checking
- `===` vs `==` for X/Z-safe comparison
- VCD waveform generation with `$dumpfile` / `$dumpvars`
- Debugging — caught and fixed a real carry logic bug during simulation

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Icarus Verilog 0.9.7 | Simulation |
| EDA Playground | Browser-based IDE |
| EPWave | Waveform viewer |

---

## What I Learned

This project taught me the fundamental mindset shift between software 
and hardware design — you're not writing instructions that execute 
sequentially, you're describing physical circuits that exist 
simultaneously. Every `case` branch is real gate logic that's always 
powered; the opcode just selects which output to forward via a 
multiplexer. The verification process — writing a testbench that 
catches bugs automatically rather than eyeballing waveforms — gave me 
hands-on experience with the RTL + verification workflow used in 
industry VLSI design.

---

## Project Structure
