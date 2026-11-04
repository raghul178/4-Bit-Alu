# Design and Implementation of a 4-Bit CMOS Arithmetic Logic Unit (ALU) Using Transistor-Level Design

A transistor-level CMOS design of a 4-bit ALU supporting eight arithmetic, logic and shift operations, with multiplexer-based output selection and four status flags (Zero, Negative, Carry, Overflow). The design is simulated in LTspice at a 1.8 V supply.

> **How to read this document:** items marked `[VERIFY FROM ORIGINAL DESIGN]` could not be confirmed from the handwritten design notes and must be checked against the original schematics before publishing. Boxes marked `[INSERT ... HERE]` are empty image placeholders (paths are placeholders, not existing files).

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Project Specifications](#2-project-specifications)
3. [Tools and Technologies](#3-tools-and-technologies)
4. [ALU Architecture](#4-alu-architecture)
5. [Operation Selection Table](#5-operation-selection-table)
6. [Individual Circuit Design and Implementation](#6-individual-circuit-design-and-implementation)
   - [6.1 4-Bit Adder](#61-4-bit-adder)
   - [6.2 4-Bit Subtractor](#62-4-bit-subtractor)
   - [6.3 4-Bit AND](#63-4-bit-and)
   - [6.4 4-Bit OR](#64-4-bit-or)
   - [6.5 4-Bit NOT](#65-4-bit-not)
   - [6.6 4-Bit XOR](#66-4-bit-xor)
   - [6.7 4-Bit Logical Left Shift](#67-4-bit-logical-left-shift)
   - [6.8 4-Bit Logical Right Shift](#68-4-bit-logical-right-shift)
7. [8:1 Multiplexer Design](#7-81-multiplexer-design)
8. [Status Flag Design](#8-status-flag-design)
   - [8.1 Zero Flag](#81-zero-flag-z)
   - [8.2 Negative Flag](#82-negative-flag-n)
   - [8.3 Carry Flag](#83-carry-flag-c)
   - [8.4 Overflow Flag](#84-overflow-flag-v)
9. [Final ALU Integration](#9-final-alu-integration)
10. [Transistor Count Analysis](#10-transistor-count-analysis)
11. [Simulation and Verification](#11-simulation-and-verification)
12. [Results and Conclusion](#12-results-and-conclusion)
13. [Future Enhancements](#13-future-enhancements)
14. [Repository Structure](#14-repository-structure)
15. [License](#15-license)
16. [Author and Contributors](#16-author-and-contributors)

---

## 1. Project Overview

### Introduction

An Arithmetic Logic Unit (ALU) is the computational core of a processor. This project implements a **4-bit ALU** that takes two 4-bit operands, **A** and **B**, and a 3-bit operation-select code, and produces a 4-bit result **Y** together with four status flags.

### Purpose

To design every block of the ALU at the **transistor level** (static CMOS) instead of using pre-built behavioural models, and to verify the complete design through LTspice transient simulation.

### Objectives

- Design a 4-bit ripple-carry adder and a 4-bit ripple-borrow subtractor from full-adder / full-subtractor cells derived through K-maps.
- Design 4-bit bitwise AND, OR, NOT and XOR blocks.
- Implement 4-bit logical left and right shift operations.
- Design a 4-bit-wide 8:1 multiplexer that selects one of the eight operation results.
- Design Zero, Negative, Carry and Overflow flag circuits.
- Package each block as a hierarchical symbol and integrate them into a complete ALU.
- Verify all operations and flags in LTspice.

### Features and Design Highlights

- Eight operations selected by three select lines (S2, S1, S0).
- All operation blocks work in parallel; the multiplexer selects the final result.
- Hierarchical, symbol-based design: each block has its own pin layout.
- Four status flags: Z, N, C, V.
- Carry and overflow flags are gated so they are active only for addition and subtraction.

### Transistor-Level CMOS Implementation (Brief)

Each Boolean function is realised as a static CMOS gate: a **PMOS pull-up network** connected to VDD and a complementary **NMOS pull-down network** connected to GND. The PMOS network conducts when the output must be HIGH, and the NMOS network conducts when it must be LOW; the two networks are never ON together in steady state. Complex functions (full adder, multiplexer, flags) are built from series/parallel combinations of these networks and from inverters.

---

## 2. Project Specifications

| Parameter | Value |
|---|---|
| Data width | 4 bits |
| Operand inputs | A[3:0], B[3:0] |
| Result output | Y[3:0] |
| Select inputs | S2, S1, S0 |
| Number of operations | 8 |
| Status flags | Z, N, C, V |
| Supply voltage (VDD) | 1.8 V |
| Logic LOW | 0 V |
| Logic HIGH | 1.8 V |
| Design style | Static CMOS, transistor level |
| Technology / MOSFET model |
| Simulator | LTspice |

---

## 3. Tools and Technologies

| Tool | Purpose |
|---|---|
| LTspice | Schematic capture, transient simulation, waveform viewing |
| Hand derivation (K-maps / Boolean algebra) | Logic minimisation of the full adder, full subtractor and flags |
| Git / GitHub | Version control and project presentation |

---

## 4. ALU Architecture

The ALU has the following interface:

| Signal | Width | Direction | Description |
|---|---|---|---|
| A | 4 | Input | Operand A (A3 is the MSB) |
| B | 4 | Input | Operand B (B3 is the MSB) |
| S2, S1, S0 | 3 | Input | Operation select |
| Y | 4 | Output | ALU result |
| Z, N, C, V | 1 each | Output | Zero, Negative, Carry, Overflow flags |
| VDD, GND | 1 each | Power | Supply and ground |

**Architecture summary**

1. A and B are applied **in parallel** to all operation blocks (adder, subtractor, AND, OR, XOR; NOT and the shifters use A only).
2. Each block produces a 4-bit result.
3. **Four 8:1 multiplexers** (one per output bit, packaged as one 4-bit 8×1 multiplexer) select the result chosen by S2 S1 S0 and drive Y0–Y3.
4. The **flag circuit** observes Y, the select lines and the carry/borrow signals (C3, Cout, B3, Bout) from the arithmetic blocks.

```text
        A[3:0] ──┬──► Adder ─────────┐
        B[3:0] ──┤                   │
                 ├──► Subtractor ────┤
                 ├──► AND ───────────┤
                 ├──► OR ────────────┤   4-bit 8:1 MUX ──► Y[3:0] ──► Flags ──► Z, N, C, V
                 ├──► NOT (A only) ──┤        ▲                         ▲
                 ├──► XOR ───────────┤        │                         │
                 ├──► Left Shift ────┤   S2 S1 S0              C3, Cout, B3, Bout
                 └──► Right Shift ───┘
```

> <img width="985" height="866" alt="image" src="https://github.com/user-attachments/assets/bb73bb0d-ac14-4352-a679-51c5d2cfa4d8" />


<!-- INSERT ARCHITECTURE IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Complete ALU block diagram">
</p>

<!-- END ARCHITECTURE IMAGE -->

<br>

---

## 5. Operation Selection Table

| S2 | S1 | S0 | Operation |
|---|---|---|---|
| 0 | 0 | 0 | Addition |
| 0 | 0 | 1 | Subtraction |
| 0 | 1 | 0 | AND |
| 0 | 1 | 1 | OR |
| 1 | 0 | 0 | NOT |
| 1 | 0 | 1 | XOR |
| 1 | 1 | 0 | Logical Left Shift |
| 1 | 1 | 1 | Logical Right Shift |

| Operation | Expected 4-bit output behaviour |
|---|---|
| Addition | Y = A + B (4-bit sum; carry-out on C, overflow on V) |
| Subtraction | Y = A − B (4-bit difference; borrow-out used for C, overflow on V) |
| AND | Y[i] = A[i] · B[i] |
| OR | Y[i] = A[i] + B[i] |
| NOT | Y[i] = ¬A[i] (operates on A only) |
| XOR | Y[i] = A[i] ⊕ B[i] |
| Logical Left Shift | Y = A << 1, LSB filled with 0, MSB discarded |
| Logical Right Shift | Y = A >> 1, MSB filled with 0, LSB discarded |

---

## 6. Individual Circuit Design and Implementation

### 6.1 4-Bit Adder

#### 1. Functionality

The adder is a **4-bit ripple-carry adder** made of four cascaded full adders (F.A.1 – F.A.4). The carry-out of each stage feeds the carry-in of the next. Inputs are A0–A3, B0–B3 and the initial carry C0; outputs are S0–S3 and Cout. The carry into the MSB stage (C3) is also needed by the overflow flag.

Pin interface of the hierarchical symbol (from notes): inputs A0–A3, B0–B3, C0, VDD, GND; outputs S0–S3, Cout, C3 .

#### 2. Truth table (full adder)

| A | B | Cin | Sum (S) | Cout |
|:-:|:-:|:---:|:-------:|:----:|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 |

#### 3. K-map

K-maps for **Sum** and **Carry** were derived (rows: A; columns: B·Cin).

> <img width="1018" height="407" alt="image" src="https://github.com/user-attachments/assets/25d598ed-9b99-4179-a480-061ca5bb829c" />



<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Full adder sum K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

> <img width="1052" height="401" alt="image" src="https://github.com/user-attachments/assets/e9e14f38-8808-4e9a-9b12-b9d7d2c1e903" />


<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Full adder carry K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

#### 4. Boolean expressions

Sum (sum of minterms):

$$S = A\overline{B}\,\overline{C_{in}} + \overline{A}\,\overline{B}\,C_{in} + \overline{A}B\overline{C_{in}} + ABC_{in}$$

Carry:

$$C_{out} = AB + AC_{in} + BC_{in} = AB + C_{in}(A + B)$$

Sum expressed using the carry (used for the CMOS implementation):

$$S = ABC_{in} + \overline{C_{out}}\,(A + B + C_{in})$$

#### 5. CMOS implementation

The carry is generated by a complex static CMOS gate implementing $AB + C_{in}(A+B)$ (PMOS and NMOS networks are series/parallel duals of each other). The sum reuses the **inverted carry** through the expression $S = ABC_{in} + \overline{C_{out}}(A+B+C_{in})$, which reduces transistor count compared with building the sum independently. Four such cells are chained for the 4-bit adder.

#### 6. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit ripple-carry adder (4 full adders) | 56 | 56 | **112** |

The notes show a first figure of 120 crossed out and **112** written as the final value. 112 ÷ 4 = 28 transistors per full adder.

#### 7. Circuit schematic

> <img width="1563" height="766" alt="image" src="https://github.com/user-attachments/assets/0d62ef40-8c19-45c0-bf8c-01b5b0a26519" />


<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Full adder CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> <img width="623" height="870" alt="image" src="https://github.com/user-attachments/assets/81aa83e0-2ecb-45f8-8fc6-96f531584040" />


<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="4-bit ripple carry adder connection">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> <img width="470" height="837" alt="image" src="https://github.com/user-attachments/assets/35028cb4-f5ea-4e35-9f98-baead45d90c6" />


<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="4-bit adder hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

#### 8. LTspice verification

> <img width="1315" height="603" alt="image" src="https://github.com/user-attachments/assets/43a555f6-603f-49d8-a5d8-9c649309a4e0" />


<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Adder simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

### 6.2 4-Bit Subtractor

#### 1. Functionality

The subtractor is a **4-bit ripple-borrow subtractor** made of four cascaded full subtractors (F.S.). The borrow-out of each stage feeds the borrow-in of the next. Inputs are A0–A3, B0–B3 and Bin; outputs are D0–D3 and Bout. The borrow into the MSB stage (labelled **B3** on the notes' pin layout) is also output for the overflow flag.

> **Naming note:** in this document **B3 (borrow signal)** means the borrow entering the MSB stage and is an *output pin of the subtractor*. It is **not** the operand bit B3. The operand bit is written B[3] / "operand B3" where ambiguity could arise. `

#### 2. Truth table (full subtractor, A − B − Bin)

| A | B | Bin | Difference (D) | Bout |
|:-:|:-:|:---:|:--------------:|:----:|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 1 |
| 0 | 1 | 0 | 1 | 1 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 |
| 1 | 1 | 1 | 1 | 1 |

#### 3. K-map

> <img width="857" height="492" alt="image" src="https://github.com/user-attachments/assets/aa46ccf3-3da0-4d01-9f2c-6fe5fb18c02c" />


<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Full subtractor difference K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

> <img width="712" height="492" alt="image" src="https://github.com/user-attachments/assets/382902d9-b229-438e-b095-67cae6ca4f7d" />


<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Full subtractor borrow K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

#### 4. Boolean expressions

Difference:

$$D = A \oplus B \oplus B_{in}$$

The notes also contain a second, carry-style form of $D$ that is hard to read: 

Borrow:

$$B_{out} = B\,B_{in} + \overline{A}\,(B + B_{in})$$

#### 5. CMOS implementation

The borrow is generated by a complex static CMOS gate implementing $B\,B_{in} + \overline{A}(B + B_{in})$. The difference output is the three-input XOR of A, B and Bin, built using the borrow signal in the same way the adder reuses its carry. 

#### 6. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit ripple-borrow subtractor (4 full subtractors) | 67 | 67 | **134** |

134 does not divide evenly by 4 (33.5), so the four stages may not be identical or the count includes extra gates (e.g., inverters).

#### 7. Circuit schematic

> <img width="1901" height="832" alt="image" src="https://github.com/user-attachments/assets/fda7420a-24ad-47a8-a866-87d333186ea9" />


<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Full subtractor CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> <img width="508" height="862" alt="image" src="https://github.com/user-attachments/assets/3e290322-3856-489c-8570-4e78522e3d8d" />


<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="4-bit ripple borrow subtractor connection">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> <img width="708" height="860" alt="image" src="https://github.com/user-attachments/assets/fe8edd5a-beb0-47e5-9d54-29fbadcafb08" />


<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="4-bit subtractor hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

#### 8. LTspice verification

> <img width="1363" height="621" alt="image" src="https://github.com/user-attachments/assets/215e4834-c98a-4371-a299-6ed8dc82549c" />


<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Subtractor simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

### 6.3 4-Bit AND

#### 1. Functionality

Bitwise AND of A and B: each output bit is HIGH only when both corresponding input bits are HIGH. Inputs A0–A3, B0–B3; outputs AND0–AND3.

#### 2. Truth table (per bit)

| A[i] | B[i] | AND[i] |
|:----:|:----:|:------:|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |


#### 3. Boolean expression

$$AND_i = A_i \cdot B_i, \quad i = 0,1,2,3$$

#### 4. CMOS implementation

Each bit is a 2-input AND gate: a NAND gate (two series NMOS, two parallel PMOS) followed by an inverter. `[VERIFY FROM ORIGINAL DESIGN — confirm gate structure]`

#### 5. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit AND | 12 | 12 | **24** |

(24 ÷ 4 = 6 transistors per bit, consistent with NAND + inverter. )

#### 6. Circuit schematic

><img width="1881" height="691" alt="image" src="https://github.com/user-attachments/assets/61811286-4ed4-40c3-96b0-5cb5673bdde5" />

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="AND CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> <img width="680" height="843" alt="image" src="https://github.com/user-attachments/assets/a3630faa-5212-4c3b-9138-3a4c86b383dd" />


<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="4-bit AND hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>


---

### 6.4 4-Bit OR

#### 1. Functionality

Bitwise OR of A and B: each output bit is HIGH when at least one corresponding input bit is HIGH. Inputs A0–A3, B0–B3; outputs OR0–OR3.

#### 2. Truth table (per bit)

| A[i] | B[i] | OR[i] |
|:----:|:----:|:-----:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

#### 3. K-map

> **[INSERT OR K-MAP HERE (OPTIONAL)]** · suggested file: `or_kmap.png`

<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="400" alt="OR K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

#### 4. Boolean expression

$$OR_i = A_i + B_i, \quad i = 0,1,2,3$$

#### 5. CMOS implementation

Each bit is a 2-input OR gate: a NOR gate (two series PMOS, two parallel NMOS) followed by an inverter. `[VERIFY FROM ORIGINAL DESIGN — confirm gate structure]`

#### 6. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit OR | `[VERIFY]` | `[VERIFY]` | **24** |

(24 ÷ 4 = 6 transistors per bit, consistent with NOR + inverter. `[VERIFY]`)

#### 7. Circuit schematic

> **[INSERT OR TRANSISTOR-LEVEL CIRCUIT HERE]** · suggested file: `or_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="OR CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT OR HIERARCHICAL SYMBOL HERE]** · suggested file: `or_4bit_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="4-bit OR hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

#### 8. LTspice verification

> **[INSERT OR LTSPICE WAVEFORM HERE]** · suggested file: `or_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="OR simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

### 6.5 4-Bit NOT

#### 1. Functionality

Bitwise complement of A. Only operand A is used. Inputs A0–A3; outputs NOT0–NOT3.

#### 2. Truth table (per bit)

| A[i] | NOT[i] |
|:----:|:------:|
| 0 | 1 |
| 1 | 0 |

#### 3. K-map

> **[INSERT NOT K-MAP HERE (OPTIONAL)]** · suggested file: `not_kmap.png`

<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="300" alt="NOT K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

#### 4. Boolean expression

$$NOT_i = \overline{A_i}, \quad i = 0,1,2,3$$

#### 5. CMOS implementation

Each bit is a standard CMOS inverter: one PMOS (source to VDD) and one NMOS (source to GND) sharing gate and drain.

#### 6. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit NOT | `[VERIFY]` | `[VERIFY]` | **8** |

(8 ÷ 4 = 2 transistors per bit, i.e., one PMOS and one NMOS per inverter.)

#### 7. Circuit schematic

> **[INSERT NOT TRANSISTOR-LEVEL CIRCUIT HERE]** · suggested file: `not_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="NOT CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT NOT HIERARCHICAL SYMBOL HERE]** · suggested file: `not_4bit_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="4-bit NOT hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

#### 8. LTspice verification

> **[INSERT NOT LTSPICE WAVEFORM HERE]** · suggested file: `not_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="NOT simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

### 6.6 4-Bit XOR

#### 1. Functionality

Bitwise exclusive-OR of A and B: each output bit is HIGH when the corresponding input bits differ. Inputs A0–A3, B0–B3; outputs XOR0–XOR3.

#### 2. Truth table (per bit)

| A[i] | B[i] | XOR[i] |
|:----:|:----:|:------:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

#### 3. K-map

> **[INSERT XOR K-MAP HERE (OPTIONAL)]** · suggested file: `xor_kmap.png`

<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="400" alt="XOR K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

#### 4. Boolean expression

$$XOR_i = A_i \oplus B_i = A_i\overline{B_i} + \overline{A_i}B_i, \quad i = 0,1,2,3$$

#### 5. CMOS implementation

Each bit is a static CMOS XOR gate (complementary PMOS/NMOS networks, with inverted inputs generated internally). `[VERIFY FROM ORIGINAL DESIGN — gate topology]`

#### 6. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit XOR | `[VERIFY]` | `[VERIFY]` | **56** |

(56 ÷ 4 = 14 transistors per bit. `[VERIFY]`)

#### 7. Circuit schematic

> **[INSERT XOR TRANSISTOR-LEVEL CIRCUIT HERE]** · suggested file: `xor_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="XOR CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT XOR HIERARCHICAL SYMBOL HERE]** · suggested file: `xor_4bit_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="4-bit XOR hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

#### 8. LTspice verification

> **[INSERT XOR LTSPICE WAVEFORM HERE]** · suggested file: `xor_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="XOR simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

### 6.7 4-Bit Logical Left Shift

#### 1. Functionality

Shifts A one position toward the MSB. Bit A3 is discarded and a 0 is inserted at the LSB. Inputs A0–A3; outputs LS0–LS3.

Worked example from the notes: `1101 << 1 = 1010`.

#### 2. Truth / logic table

| Output | Source |
|---|---|
| LS0 | 0 |
| LS1 | A0 |
| LS2 | A1 |
| LS3 | A2 |
| (A3) | shifted out |

#### 3. K-map

Not applicable: each output is a direct wire or constant. 

> **[INSERT LEFT SHIFT K-MAP HERE (OPTIONAL — NOT NEEDED)]** · suggested file: `left_shift_kmap.png`

#### 4. Boolean expressions

$$LS_0 = 0,\quad LS_1 = A_0,\quad LS_2 = A_1,\quad LS_3 = A_2$$

#### 5. Implementation

In the notes' schematic, each output is driven by a circular source/buffer symbol (appears to be a voltage-controlled voltage source) that copies the selected input bit, with a resistor (value written as about 10 GΩ) from the discarded bit A3 to ground and a common GND. This suggests the shifter is built from ideal buffer elements rather than MOSFETs. `[VERIFY FROM ORIGINAL DESIGN — element type, resistor value, and whether a transistor-level version exists]`

#### 6. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit left shift | `[VERIFY]` | `[VERIFY]` | `[VERIFY FROM ORIGINAL DESIGN — no count in notes]` |

#### 7. Circuit schematic

> **[INSERT LEFT SHIFT CIRCUIT HERE]** · suggested file: `left_shift_circuit.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Left shift circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT LEFT SHIFT HIERARCHICAL SYMBOL HERE]** · suggested file: `left_shift_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="Left shift hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

#### 8. LTspice verification

> **[INSERT LEFT SHIFT LTSPICE WAVEFORM HERE]** · suggested file: `left_shift_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Left shift simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

### 6.8 4-Bit Logical Right Shift

#### 1. Functionality

Shifts A one position toward the LSB. Bit A0 is discarded and a 0 is inserted at the MSB. Inputs A0–A3; outputs RS0–RS3.

Worked example from the notes: `1011 >> 1 = 0101`.

#### 2. Truth / logic table

| Output | Source |
|---|---|
| RS0 | A1 |
| RS1 | A2 |
| RS2 | A3 |
| RS3 | 0 |
| (A0) | shifted out |

#### 3. K-map

Not applicable: each output is a direct wire or constant.

> **[INSERT RIGHT SHIFT K-MAP HERE (OPTIONAL — NOT NEEDED)]** · suggested file: `right_shift_kmap.png`

#### 4. Boolean expressions

$$RS_0 = A_1,\quad RS_1 = A_2,\quad RS_2 = A_3,\quad RS_3 = 0$$

#### 5. Implementation

Same approach as the left shifter: circular buffer/source symbols copy the selected bit to each output, and a resistor (value written as about 100 GΩ) ties the discarded bit A0 to ground. `[VERIFY FROM ORIGINAL DESIGN — element type and resistor value]`

#### 6. Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit right shift | `[VERIFY]` | `[VERIFY]` | `[VERIFY FROM ORIGINAL DESIGN — no count in notes]` |

#### 7. Circuit schematic

> **[INSERT RIGHT SHIFT CIRCUIT HERE]** · suggested file: `right_shift_circuit.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Right shift circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT RIGHT SHIFT HIERARCHICAL SYMBOL HERE]** · suggested file: `right_shift_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="500" alt="Right shift hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

#### 8. LTspice verification

> **[INSERT RIGHT SHIFT LTSPICE WAVEFORM HERE]** · suggested file: `right_shift_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Right shift simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

## 7. 8:1 Multiplexer Design

### Operation

A $2^N$-to-1 multiplexer has $N$ select lines; for eight inputs, $N = 3$. The notes title this block "8×3 Multiplexer" and draw it as a **4-bit 8×1 multiplexer**: for every output bit, the multiplexer picks one of eight operation results according to S2 S1 S0.

### Inputs and outputs

| Group | Signals |
|---|---|
| Data inputs, bit 0 | D0 … D7 = results of Add, Sub, AND, OR, NOT, XOR, LS, RS for bit 0 |
| Data inputs, bit 1 | same eight results for bit 1 |
| Data inputs, bit 2 | same eight results for bit 2 |
| Data inputs, bit 3 | same eight results for bit 3 |
| Select inputs | S2, S1, S0 (shared by all four bit slices) |
| Outputs | Y0, Y1, Y2, Y3 |
| Power | VDD, GND |

In the notes' symbol the 32 data pins are grouped per bit (adder sum, subtractor difference, AND, OR, NOT, XOR, LS, RS for bit 0, then bit 1, and so on). `[VERIFY FROM ORIGINAL DESIGN — exact pin names; the notes label the XOR pins "XNOR"]`

### Selection truth table

| S2 | S1 | S0 | Selected input | Operation |
|:--:|:--:|:--:|:--------------:|-----------|
| 0 | 0 | 0 | D0 | Addition |
| 0 | 0 | 1 | D1 | Subtraction |
| 0 | 1 | 0 | D2 | AND |
| 0 | 1 | 1 | D3 | OR |
| 1 | 0 | 0 | D4 | NOT |
| 1 | 0 | 1 | D5 | XOR |
| 1 | 1 | 0 | D6 | Left shift |
| 1 | 1 | 1 | D7 | Right shift |

### K-map

> **[INSERT 8:1 MUX K-MAP HERE]** · suggested file: `mux_8to1_kmap.png`

<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="8:1 MUX K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

### Boolean expression

$$Y = D_0\overline{S_2}\,\overline{S_1}\,\overline{S_0} + D_1\overline{S_2}\,\overline{S_1}S_0 + D_2\overline{S_2}S_1\overline{S_0} + D_3\overline{S_2}S_1S_0 + D_4S_2\overline{S_1}\,\overline{S_0} + D_5S_2\overline{S_1}S_0 + D_6S_2S_1\overline{S_0} + D_7S_2S_1S_0$$

(The notes' last term is written with "S7" in place of S1; it is corrected to $S_1$ here.)

> **[INSERT 8:1 MUX BOOLEAN EXPRESSION HERE (OPTIONAL IMAGE OF HANDWRITTEN DERIVATION)]** · suggested file: `mux_8to1_expression.png`

<!-- INSERT EXPRESSION IMAGE BELOW -->

<p align="center">
  <img src=" " width="700" alt="8:1 MUX Boolean expression">
</p>

<!-- END EXPRESSION IMAGE -->

<br>

### CMOS implementation

Each output bit is implemented as an eight-way selector driven by decoded forms of S2, S1, S0 (true and complemented select lines). The same three select lines drive all four bit slices. `[VERIFY FROM ORIGINAL DESIGN — decoder + pass/AND-OR structure used]`

### Transistor count

| Component | PMOS count | NMOS count | Total |
|---|---:|---:|---:|
| 4-bit 8:1 multiplexer section | `[VERIFY]` | `[VERIFY]` | **398** |

398 is the figure recorded in the notes for the complete 4-bit multiplexer section. It is not evenly divisible by four, so shared select-decoding logic may be included. `[VERIFY FROM ORIGINAL DESIGN — counting method]`

### How four 8:1 MUXes generate the four output bits

Bit slice *i* (i = 0…3) receives the eight operation results for bit *i* on D0–D7. With the shared select code S2 S1 S0, slice *i* drives $Y_i$. Four slices therefore produce Y0, Y1, Y2, Y3 simultaneously, all selecting the same operation.

### Circuit and symbol

> **[INSERT 8:1 MUX TRANSISTOR-LEVEL CIRCUIT HERE]** · suggested file: `mux_8to1_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="8:1 MUX CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT 8:1 MUX HIERARCHICAL SYMBOL HERE]** · suggested file: `mux_8to1_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="8:1 MUX hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

### LTspice verification

> **[INSERT 8:1 MUX WAVEFORM HERE]** · suggested file: `mux_8to1_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="8:1 MUX simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

## 8. Status Flag Design

All four flags are designed in a single hierarchical block.

**Pin layout of the flag block (from notes)**

| Type | Pins |
|---|---|
| Inputs | Y0, Y1, Y2, Y3, C3, Cout, B3, Bout, SELECT2, SELECT1, SELECT0 |
| Outputs | Z, N, C, V |
| Power | VDD, GND |

Here **B3 is the borrow entering the MSB stage of the subtractor** and **Bout is its final borrow-out**; **C3** is the carry entering the MSB stage of the adder and **Cout** its final carry-out. They are signals from the arithmetic blocks, not operand bits.

**Total transistors recorded for the overall flag section: 70.**

> **[INSERT FLAG BLOCK HIERARCHICAL SYMBOL / PIN LAYOUT HERE]** · suggested file: `flags_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Flag block hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

---

### 8.1 Zero Flag (Z)

The zero flag is active for all operations. It is HIGH when every result bit is 0.

$$Z = \overline{Y_3 + Y_2 + Y_1 + Y_0}$$

**Implementation:** a 4-input NOR gate. The PMOS pull-up network is four PMOS transistors in **series** (all four must be ON, i.e., all Y inputs LOW, to pull Z to VDD). The NMOS pull-down network is four NMOS transistors in **parallel** (any HIGH Y input pulls Z to GND). The schematic in the notes therefore shows 4 PMOS + 4 NMOS = 8 transistors. `[VERIFY FROM ORIGINAL DESIGN — per-flag counts]`

| Y3 Y2 Y1 Y0 | Z |
|:--:|:--:|
| 0000 | 1 |
| any other | 0 |

> **[INSERT ZERO FLAG K-MAP HERE]** · suggested file: `zero_flag_kmap.png`

<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Zero flag K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

> **[INSERT ZERO FLAG CMOS CIRCUIT HERE]** · suggested file: `zero_flag_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Zero flag CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

---

### 8.2 Negative Flag (N)

The negative flag works for all operations and checks the MSB of the ALU output.

$$N = Y_3$$

In 4-bit two's-complement representation, the MSB carries negative weight (−8), so Y3 = 1 means the result is interpreted as a negative number and Y3 = 0 as non-negative. For logical and shift operations, N simply mirrors the MSB of the result.

**Implementation:** drawn in the notes as a single circular buffer symbol referenced to ground. `[VERIFY FROM ORIGINAL DESIGN — buffer element type and whether it contributes MOSFETs to the 70-transistor flag total]`

> **[INSERT NEGATIVE FLAG CIRCUIT HERE]** · suggested file: `negative_flag_circuit.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Negative flag circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

---

### 8.3 Carry Flag (C)

The carry flag is enabled **only for addition and subtraction**.

$$C = \overline{S_2}\;\overline{S_1}\,\left(\overline{S_0}\,C_{ADD} + S_0\,B_{SUB}\right)$$

| Term | Meaning |
|---|---|
| $\overline{S_2}\,\overline{S_1}$ | Enables the flag only when S2 S1 = 00 (addition or subtraction) |
| $C_{ADD}$ | Carry-out of the adder (Cout) |
| $B_{SUB}$ | Borrow-out of the subtractor (Bout) |
| $\overline{S_0}$ / $S_0$ | Picks addition (S0 = 0) or subtraction (S0 = 1) |

`[VERIFY FROM ORIGINAL DESIGN — confirm that C_ADD = Cout and B_SUB = Bout]`

- **Addition (S2 S1 S0 = 000):** C = C_ADD, the carry out of the MSB stage.
- **Subtraction (001):** C = B_SUB, the borrow out of the MSB stage.
- **Logical and shift operations (S1 = 1 or S2 = 1):** the $\overline{S_2}\,\overline{S_1}$ term is 0, so C is forced to 0; a carry has no meaning for these operations.

**Implementation:** a complex static CMOS gate implementing the expression above (series/parallel PMOS and NMOS networks driven by S2̄, S1̄, S0, S0̄, C_ADD and B_SUB), followed by an output inverter stage. `[VERIFY FROM ORIGINAL DESIGN — exact topology and transistor count]`

> **[INSERT CARRY FLAG K-MAP HERE]** · suggested file: `carry_flag_kmap.png`

<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Carry flag K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

> **[INSERT CARRY FLAG CMOS CIRCUIT HERE]** · suggested file: `carry_flag_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Carry flag CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

---

### 8.4 Overflow Flag (V)

The overflow flag is active for addition and subtraction.

$$V = \overline{S_2}\;\overline{S_1}\,\left(\overline{S_0}\,V_{add} + S_0\,V_{sub}\right)$$

where

$$V_{add} = C_3 \oplus C_4 \qquad V_{sub} = B_3 \oplus B_4$$

with $C_4 \rightarrow C_{out}$ and $B_4 \rightarrow B_{out}$ (as annotated in the notes).

> **Signal-naming clarification:** in $V_{sub}$, **B3 is the borrow entering the MSB stage** and **B4 (= Bout) is the borrow leaving it**. B3 here is **not** the operand bit B[3]. Likewise C3 is the carry into the MSB stage and C4 (= Cout) the carry out of it.

**Addition:** signed overflow occurs when the carry into the MSB differs from the carry out of the MSB, hence $V_{add} = C_3 \oplus C_4$.

**Subtraction:** the same condition expressed in borrows: overflow occurs when the borrow into the MSB differs from the borrow out of the MSB, hence $V_{sub} = B_3 \oplus B_4$.

**Selection and gating:** the second stage selects $V_{add}$ (S0 = 0) or $V_{sub}$ (S0 = 1) and is enabled by $\overline{S_2}\,\overline{S_1}$, so V = 0 for all logical and shift operations.

**Implementation:** three sections (from the notes' schematic): (1) an XOR gate for $V_{add}$ using C3, C4 and their complements; (2) an XOR gate for $V_{sub}$ using B3, B4 and their complements; (3) a selection/gating gate with S2̄, S1̄, S0 and S0̄, followed by an output inverter. `[VERIFY FROM ORIGINAL DESIGN — transistor counts and complement-signal generation]`

> **[INSERT OVERFLOW FLAG K-MAP HERE]** · suggested file: `overflow_flag_kmap.png`

<!-- INSERT K-MAP IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Overflow flag K-map">
</p>

<!-- END K-MAP IMAGE -->

<br>

> **[INSERT ADDITION OVERFLOW CIRCUIT HERE]** · suggested file: `overflow_add_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="700" alt="Addition overflow CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT SUBTRACTION OVERFLOW CIRCUIT HERE]** · suggested file: `overflow_sub_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="700" alt="Subtraction overflow CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

> **[INSERT OVERFLOW FLAG SELECTION CIRCUIT HERE]** · suggested file: `overflow_flag_cmos.png`

<!-- INSERT CIRCUIT IMAGE BELOW -->

<p align="center">
  <img src=" " width="750" alt="Overflow flag selection CMOS circuit">
</p>

<!-- END CIRCUIT IMAGE -->

<br>

---

## 9. Final ALU Integration

### Connections

| Connection | Description |
|---|---|
| Inputs | A0–A3 and B0–B3 feed all operation blocks in parallel (NOT and both shifters use A only); C0 / Bin initial carry/borrow `[VERIFY: tied to 0?]` |
| Selection lines | S2, S1, S0 drive the 4-bit 8:1 multiplexer select inputs and the flag block select inputs (SELECT2, SELECT1, SELECT0) |
| Output connections | Each operation block's output bit *i* goes to its dedicated data input of multiplexer slice *i*; slices output Y0–Y3 |
| Flag connections | Y0–Y3 go to the flag block (Z, N). C3 and Cout from the adder, B3 and Bout from the subtractor go to the flag block (C, V) |
| VDD / GND | Common 1.8 V supply and ground distributed to every block |

### Hierarchical symbol design

Every block (adder, subtractor, AND, OR, NOT, XOR, left shift, right shift, multiplexer, flag block) is packaged as a hierarchical symbol with VDD on top and GND at the bottom, inputs on the left and outputs on the right. The final ALU symbol is formed by instantiating these symbols together.

> **[INSERT COMPLETE INTEGRATED ALU SCHEMATIC HERE]** · suggested file: `alu_integrated_schematic.png`

<!-- INSERT SCHEMATIC IMAGE BELOW -->

<p align="center">
  <img src=" " width="800" alt="Complete integrated ALU schematic">
</p>

<!-- END SCHEMATIC IMAGE -->

<br>

> **[INSERT FINAL ALU HIERARCHICAL SYMBOL HERE]** · suggested file: `alu_hierarchical_symbol.png`

<!-- INSERT SYMBOL IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="Final ALU hierarchical symbol">
</p>

<!-- END SYMBOL IMAGE -->

<br>

> **[INSERT ALU PIN CONFIGURATION HERE]** · suggested file: `alu_pin_configuration.png`

<!-- INSERT PIN CONFIGURATION IMAGE BELOW -->

<p align="center">
  <img src=" " width="600" alt="ALU pin configuration">
</p>

<!-- END PIN CONFIGURATION IMAGE -->

<br>

---

## 10. Transistor Count Analysis

Only counts recorded in the design notes are shown. PMOS/NMOS splits are not recorded in the notes and are left for you to fill in.

| Circuit block | PMOS | NMOS | Total |
|---|---:|---:|---:|
| 4-bit Adder | `[VERIFY]` | `[VERIFY]` | 112 |
| 4-bit Subtractor | `[VERIFY]` | `[VERIFY]` | 134 |
| AND | `[VERIFY]` | `[VERIFY]` | 24 |
| OR | `[VERIFY]` | `[VERIFY]` | 24 |
| NOT | `[VERIFY]` | `[VERIFY]` | 8 |
| XOR | `[VERIFY]` | `[VERIFY]` | 56 |
| Left Shift | `[VERIFY]` | `[VERIFY]` | `[VERIFY — not in notes]` |
| Right Shift | `[VERIFY]` | `[VERIFY]` | `[VERIFY — not in notes]` |
| 8:1 MUX section | `[VERIFY]` | `[VERIFY]` | 398 |
| Zero Flag | 4 *(from schematic)* | 4 *(from schematic)* | 8 *(from schematic)* |
| Negative Flag | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` |
| Carry Flag | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` |
| Overflow Flag | `[VERIFY]` | `[VERIFY]` | `[VERIFY]` |
| *Flag section as a whole (all four flags)* | | | *70 (recorded in notes)* |
| **Grand Total** | `[VERIFY]` | `[VERIFY]` | `[VERIFY — see note]` |

**Counting notes**

- The 398 (multiplexer section) and 70 (overall flag section) are combined-block figures from the notes. The individual flag rows above are therefore **not** additive with the 70; use either the four individual flag counts or the 70, not both.
- Adding only the recorded totals (adder 112 + subtractor 134 + AND 24 + OR 24 + NOT 8 + XOR 56 + MUX 398 + flags 70) gives **826 transistors**, excluding the shifters. This is a provisional sum, not a confirmed ALU total, until the shifter counts and the counting method are verified.
- The adder count first written as 120 was crossed out and replaced by 112 in the notes.

---

## 11. Simulation and Verification

### Simulation environment

| Item | Setting |
|---|---|
| Simulator | LTspice |
| Analysis | Transient |
| Supply voltage | 1.8 V |
| Logic LOW | 0 V |
| Logic HIGH | 1.8 V |
| Input stimulus | `[VERIFY FROM ORIGINAL DESIGN — pulse/PWL sources, period, rise/fall times]` |
| Simulation time / time step | `[VERIFY FROM ORIGINAL DESIGN]` |
| MOSFET models | `[VERIFY FROM ORIGINAL DESIGN]` |

**Selection-signal behaviour:** S2, S1, S0 are stepped through 000 → 111 so that each of the eight operations is applied in turn while A and B hold test patterns. `[VERIFY FROM ORIGINAL DESIGN — actual stimulus sequence]`

### Verification table

Only the two shift examples are known from the notes; the other rows must be filled in from your simulations.

| Operation | Selection | Input A | Input B | Expected output | Simulated output | Status |
|---|---|---|---|---|---|---|
| Addition | 000 | | | | | |
| Subtraction | 001 | | | | | |
| AND | 010 | | | | | |
| OR | 011 | | | | | |
| NOT | 100 | | | | | |
| XOR | 101 | | | | | |
| Left shift | 110 | 1101 | — | 1010 | | |
| Right shift | 111 | 1011 | — | 0101 | | |

### Flag verification

| Flag | Condition tested | Expected | Simulated | Status |
|---|---|---|---|---|
| Z | Y = 0000 | 1 | | |
| N | Y3 = 1 | 1 | | |
| C | Add with carry-out / subtract with borrow-out | 1 | | |
| C | Any logic or shift operation | 0 | | |
| V | Signed overflow in add / subtract | 1 | | |
| V | Any logic or shift operation | 0 | | |

### Waveforms

> **[INSERT ADDER WAVEFORM HERE]** · suggested file: `adder_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="800" alt="Adder waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

> **[INSERT SUBTRACTOR WAVEFORM HERE]** · suggested file: `subtractor_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="800" alt="Subtractor waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

> **[INSERT LOGIC OPERATIONS WAVEFORM HERE]** · suggested file: `logic_operations_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="800" alt="Logic operations waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

> **[INSERT SHIFT OPERATIONS WAVEFORM HERE]** · suggested file: `shift_operations_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="800" alt="Shift operations waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

> **[INSERT STATUS FLAGS WAVEFORM HERE]** · suggested file: `status_flags_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="800" alt="Status flags waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

> **[INSERT COMPLETE ALU SIMULATION WAVEFORM HERE]** · suggested file: `alu_final_waveform.png`

<!-- INSERT WAVEFORM IMAGE BELOW -->

<p align="center">
  <img src=" " width="800" alt="Complete ALU simulation waveform">
</p>

<!-- END WAVEFORM IMAGE -->

<br>

---

## 12. Results and Conclusion

This project implements a 4-bit ALU at the transistor level in CMOS, covering eight operations: addition, subtraction, AND, OR, NOT, XOR, logical left shift and logical right shift. The arithmetic blocks were derived from K-map minimisation of the full adder and full subtractor and cascaded as ripple-carry and ripple-borrow structures. Output selection uses a 4-bit 8:1 multiplexer driven by S2 S1 S0, and four status flags (Zero, Negative, Carry, Overflow) are generated from the result bits, select lines and carry/borrow signals, with carry and overflow gated to the arithmetic operations only.

Functional verification is performed in LTspice at 1.8 V. `[ADD YOUR OBSERVED RESULTS HERE ONCE THE VERIFICATION TABLE IS COMPLETE — no results are claimed in this draft]`

**Learning outcomes**

- Translating Boolean expressions and K-maps into static CMOS pull-up/pull-down networks.
- Hierarchical, symbol-based design of multi-block circuits.
- Multiplexer-based datapath selection and status-flag generation.
- Understanding carry versus borrow semantics and signed overflow detection.

---

## 13. Future Enhancements

- Replace ripple-carry/borrow chains with carry-lookahead or carry-select structures to reduce delay.
- Replace the shift blocks with a fully transistor-level implementation (if currently source-based) and extend to a barrel shifter, including arithmetic shift and rotate.
- Add multiply and compare operations.
- Reduce the multiplexer transistor count (e.g., pass-transistor or transmission-gate multiplexing).
- Characterise propagation delay, power and area; add transistor sizing optimisation.
- Port the design to a full-custom layout flow (schematic, layout, DRC/LVS) and perform post-layout simulation.

---

## 14. Repository Structure

```text
4-bit-CMOS-ALU/
│
├── README.md
│
├── Documentation/
│   ├── ALU_Architecture/
│   ├── Boolean_Expressions/
│   ├── K_Maps/
│   ├── Transistor_Count/
│   └── Verification/
│
├── Circuits/
│   ├── Adder/
│   ├── Subtractor/
│   ├── AND/
│   ├── OR/
│   ├── NOT/
│   ├── XOR/
│   ├── Left_Shift/
│   ├── Right_Shift/
│   ├── Multiplexer/
│   └── Status_Flags/
│
├── Symbols/
│
├── Waveforms/
│
└── LTspice/
    ├── Schematics/
    └── Simulation_Files/
```

---

## 15. License

`[ADD LICENSE HERE — e.g., MIT, CC BY 4.0, or "All rights reserved"]`

---

## 16. Author and Contributors

| Role | Name | Contact |
|---|---|---|
| Author | `[YOUR NAME]` | `[EMAIL / LINKEDIN / GITHUB]` |
| Contributors | `[NAMES, IF ANY]` | |
| Institution / Guide | `[OPTIONAL]` | |
