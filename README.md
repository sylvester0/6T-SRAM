# 1 kb 6T SRAM

## 1. Introduction

This repository contains the schematic level design and verification of a 1 kb static random access memory (SRAM) based on a conventional 6 transistor (6T) bitcell and its peripheral circuitry.

The memory core is a 32 x 32 array of 6T cells. All peripheral blocks required for reliable read and write access are included:

- Row decoder (5 to 32)
- Column decoder
- Column multiplexer
- Bitline precharge and equalization
- Write driver
- Sense amplifier
- Top level control and 1 bit I/O interface

The design is intended as a reference for educational and research work in SRAM design, emphasizing transistor level implementation and transient simulation of each block and of the complete 1 kb array.

---

## 2. Top level architecture

### 2.1 Block diagram

The top level architecture follows the classic synchronous SRAM organization:

- **32 x 32 6T bitcell array**  
- **Row decoder (5 to 32)** that asserts exactly one wordline WL[0:31].  
- **Column decoder** that selects one column multiplexer corresponding to the requested bit position.  
- **Column multiplexer** that connects the selected bitline pair to the global bitline pair for I/O.  
- **Precharge and equalization** that precharges both bitline (BL) and bitline bar (BLB) to VDD and equalizes them before each access.  
- **Write driver** that applies the input data pattern onto BL and BLB during a write cycle.  
- **Sense amplifier** that senses the differential voltage on BL and BLB during a read and produces a rail to rail single ended output.  

The overall schematic of the 1 kb SRAM and its periphery is shown in the project slide deck, page 22 and 23.

### 2.2 Address and data interface

At the top level the memory exposes:

- `ADDR[9:0]`  
  - `ADDR[4:0]` select the row through the row decoder (one hot WL[0:31])  
  - `ADDR[9:5]` select the column through the column decoder and column MUX  
- `DIN` single bit data input  
- `DOUT` single bit data output  
- Control signals  
  - `CLK` main clock (if used in your testbench)  
  - `WE` write enable (active high)  
  - `RE` read enable or sense enable (active high)  
  - `PR` precharge enable (active high)  

> Edit names to match your actual symbols.

---

## 3. 6T SRAM bitcell

### 3.1 Schematic

The bitcell is a standard 6T structure:

- Two cross coupled inverters form a storage latch  
  - Pull up PMOS transistors: `PM1`, `PM2`  
  - Pull down NMOS transistors: `NM3`, `NM4`  
- Two NMOS access transistors connect internal storage nodes `Q` and `QB` to the bitlines `BL` and `BLB` under control of the wordline `WL`.  

### 3.2 Transistor sizing guidelines

To satisfy both read stability and write ability constraints, devices are sized according to:

- Pull up PMOS weaker than access NMOS  
- Access NMOS weaker than pull down NMOS  

In terms of aspect ratios:

- (W/L)\_pullup < (W/L)\_access << (W/L)\_pulldown

Intuition:

- **Write operation**  
  Access devices must be strong enough to overpower the pull up devices and flip the internal node when a new value is written.  
- **Read operation**  
  Pull down devices must be strong enough to hold the stored node low while the opposite node experiences only a small disturbance through the access device.

### 3.3 Bitcell operation

**Write cycle**

1. Precharge is disabled so that the write driver can actively drive the bitlines.  
2. Write driver forces `BL` and `BLB` to the desired logic levels.  
3. Corresponding wordline `WL` goes high, turning on access transistors.  
4. Internal nodes `Q` and `QB` are overwritten by the stronger bitline drivers.  
5. Wordline returns low, isolating the cell and storing the new data.

**Read cycle**

1. Both bitlines are precharged high and equalized.  
2. Write driver is disabled and its outputs are placed in high impedance.  
3. Selected wordline `WL` is asserted high.  
   - If the stored bit is `1`: one internal node is high, the opposite node is low and discharges its connected bitline slightly through the corresponding pull down NMOS.  
   - If the stored bit is `0`: the roles of `BL` and `BLB` are reversed.  
4. The small differential voltage between `BL` and `BLB` is sensed and amplified by the sense amplifier.  
5. After sensing completes, the wordline is deasserted and precharge prepares the bitlines for the next operation.

---

## 4. Bitline precharge and equalization

### 4.1 Schematic

The precharge circuit consists of:

- Two PMOS transistors (`M0`, `M1`) that connect `BL` and `BLB` to `VDD` when `PR` is asserted.  
- One equalization transistor (`M2`) that shorts `BL` and `BLB` together when `PR` is asserted, equalizing their voltage.

This topology is shown in the slide deck on page 4.

### 4.2 Operation

- When `PR = 1`  
  - Both bitlines are charged up to `VDD`.  
  - Equalization device forces `V(BL) ≈ V(BLB)`.  
- When `PR` returns low  
  - Bitlines are isolated from `VDD` and from each other.  
  - The array is ready for a read or write operation.

Because bitlines have large parasitic capacitance, the precharge PMOS devices are sized wide enough to supply sufficient current and meet the required access time.

### 4.3 Precharge testbench

A dedicated testbench instantiates the precharge cell, drives `PR`, and monitors `BL` and `BLB`:

- `PR` toggles periodically.  
- Bitline waveforms show both `BL` and `BLB` charging to `VDD` during the precharge phase and remaining equalized.  
- When `PR` is low, bitlines are free to be pulled by the array or write driver.

---

## 5. Write driver

### 5.1 Concept

The write driver converts the single ended input `DIN` and the write enable `WE` into complementary bitline levels on `BL` and `BLB`.

Key ideas:

- When `WE = 1`, the write driver overrides the precharged bitlines and drives a strong differential signal.  
- When `WE = 0`, the write driver outputs are high impedance so that the bitlines can float during read and precharge phases.

### 5.2 Schematic behavior

From the schematic in the slide deck:

- `DIN` and its complement control two pass devices `NM1` and `NM2`.  
- For `DIN = 1` and `WE = 1`  
  - `BL` is driven high.  
  - `BLB` is driven low.  
- For `DIN = 0` and `WE = 1`  
  - `BL` is driven low.  
  - `BLB` is driven high.

### 5.3 Write driver testbench

The write driver testbench:

- Sweeps `DIN` between 0 and 1 while asserting and deasserting `WE`.  
- Observes that  
  - when `WE = 1`, `BL` and `BLB` follow the expected complementary pattern,  
  - when `WE = 0`, bitlines remain undisturbed.

---

## 6. Sense amplifier

### 6.1 Purpose

Sense amplifiers are critical to achieve fast and energy efficient reads:

- Bitlines carry small voltage differences after a short wordline pulse.  
- The sense amplifier converts this small differential (`ΔV` on `BL` versus `BLB`) into a full swing digital output `OUT`.  

### 6.2 Schematic

The sense amplifier uses a differential structure with:

- Cross coupled inverters that form a regenerative latch.  
- Input transistors connected to `BL` and `BLB`.  
- A sense enable signal `SE` that activates the latch.

The schematic is shown in the slide deck on page 10.

### 6.3 Operation

1. Before sensing, `BL` and `BLB` are precharged high. `SE` is low so the latch is inactive.  
2. During a read, the selected cell slightly discharges one of the bitlines.  
3. When `SE` is asserted, the latch is enabled.  
4. Positive feedback quickly amplifies the small bitline difference and resolves the state to a stable full swing level.  
5. The single ended output `OUT` is taken from one side of the latch.

### 6.4 Sense amplifier testbench

The sense amplifier testbench:

- Applies small differential patterns to `BL` and `BLB` that match realistic bitline behavior.  
- Pulses `SE` and records `OUT`.  
- Confirms:

  - Correct polarity of `OUT` for each input bitline pattern.  
  - Adequate sensing speed for the targeted clock period.  
  - Restoration of `OUT` when `SE` is deasserted.

---

## 7. Row decoder

### 7.1 Hierarchical structure

The row decoder is a 5 to 32 decoder realized hierarchically:

- One **2 to 4 decoder** generates four high level select lines.  
- Four **3 to 8 decoders** receive these select lines and the remaining address bits and generate 32 one hot outputs.

In summary:

- Inputs: `A[4:0]`  
- Outputs: `WL[31:0]`  
- Property: For any valid 5 bit address exactly one wordline is asserted high.

### 7.2 2 to 4 decoder

- Takes inputs `A1`, `A0`.  
- Produces `Y0` to `Y3`.  
- Only one `Y` is active for any input combination.  
- Implemented using CMOS NAND and NOR logic plus inverters to ensure clean rail to rail wordline levels.

### 7.3 3 to 8 decoder

- Takes inputs `A2`, `A1`, `A0`.  
- Produces `Y0` to `Y7`.  
- Implemented using series parallel transistor networks that realize minterms of the input address.

### 7.4 Combining to 5 to 32 decoder

- The 2 to 4 decoder selects which 3 to 8 block is active.  
- Each 3 to 8 output is logically combined with its corresponding 2 to 4 output to form a unique row select.  
- Final outputs drive the wordlines of the 32 rows in the array.

---

## 8. Column decoder and column multiplexer

### 8.1 Column decoder

The column decoder selects one column multiplexer out of the 32 columns along the selected row.

- Inputs: upper address bits `ADDR[9:5]`.  
- Outputs: `COL_SEL[31:0]` control signals, each enabling one column MUX.  
- Only one `COL_SEL` is high at a time.

The schematic of the column decoder is shown in the slide deck on page 19.

### 8.2 Column multiplexer

The column multiplexer connects one selected column’s bitline pair to the global bitline pair used by the write driver and sense amplifier:

- Each column has a local `BL_i` and `BLB_i`.  
- Pass devices controlled by `COL_SEL[i]` connect `BL_i` and `BLB_i` to global `BL` and `BLB`.  
- When `COL_SEL[i] = 0`, that column is electrically isolated.

The layout in the slide deck (page 20) shows an array of identical MUX slices, one per column.

---

## 9. One bit SRAM slice

Before integrating the full 1 kb array, a 1 bit SRAM slice is constructed and verified:

- Contains a single 6T bitcell.  
- Includes its precharge, write driver, sense amplifier, and local decoding logic.  
- Exposes the same top level signals (`DIN`, `DOUT`, `WE`, `RE`, `PR`, address subset) but only for one physical location.

The simulation shows correct behavior for repeated read and write cycles:

- Writes store the input data inside the bitcell.  
- Reads correctly reproduce the stored value at `DOUT` without upsetting the cell state.

---

## 10. 1 kb SRAM integration

### 10.1 Array organization

- 32 rows x 32 columns of identical 6T cells.  
- Row and column decoders generate one hot selects.  
- Column MUX network connects one bitline pair at a time to the I/O path.  
- Precharge, write driver, and sense amplifier are shared per global bitline pair.

### 10.2 Top level schematic

The complete schematic (page 22 and 23 of the slide deck) shows:

- Central 32 x 32 cell array.  
- Row decoder below or to the side, feeding all wordlines.  
- Column decoder and MUX array alongside the columns.  
- Precharge block placed near the global bitlines.  
- Write driver and sense amplifier connected at the end of the global bitlines.

### 10.3 Full memory simulation

A top level testbench verifies functionality across multiple addresses:

1. Precharge is applied.  
2. A sequence of addresses and data values are written while `WE = 1`.  
3. Later, the same addresses are read back with `WE = 0` and `RE` or `SE` asserted.  
4. `DOUT` is checked against the expected stored data.

Waveforms for the full 1 kb memory simulation are shown in the slide deck on the final page and confirm correct read and write behavior.

---
