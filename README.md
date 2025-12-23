# 1 kb 6T SRAM

Schematic level design and simulation of a 1 kb SRAM built from a standard 6T bitcell and full peripheral circuitry (precharge, decoders, column muxing, write driver, sense amplifier). The design is implemented in Cadence Virtuoso, with simulations validating each block and the integrated memory.

---

# repository contents

```text
.
├─ sram/                         
└─ docs/
   ├─ 6T_SRAM.pdf                  
   └─ figures/                   
      ├─ SRAM.png
      ├─ 1KB_SRAM_Cell_Array.png
      ├─ 1kb_sram_circuitry.png
      ├─ sim_1kb_sram.png
      ├─ sim_wavef_1kb_sram.png
      ├─ precharge.png
      ├─ precharge_tb.png
      ├─ precharge_sim.png
      ├─ write_driver.png
      ├─ write_driver_tb.png
      ├─ write_driver_sim.png
      ├─ senese_amp.png
      ├─ sense_amp_test.png
      ├─ sense_amp_test_sim.png
      ├─ row_decoder.png
      ├─ row_decoder_tb.png
      ├─ row_decoder_sim.png
      ├─ column_decoder.png
      ├─ column_mux.png
      ├─ CM_write_sim.png
      ├─ 2-4_decoder.png
      ├─ 2_4_decoder_tb.png
      ├─ 2_4_decoder_sim.png
      ├─ 3-8_decoder.png
      ├─ 3_8_decoder_tb.png
      ├─ 3_8_decoder_sim.png
      ├─ Design_of_1-bit_SRAM_Cell.png
      ├─ Sim_1-bit_SRAM_Cell.png
      ├─ sram_write.png
      └─ sram_read.png
---

## 1. Top level architecture

### 1.1 1 kb array overview

This project implements a 1 kb SRAM organized as a 32 x 32 6T bitcell array, accessed through row and column selection, and interfaced through shared global bitlines using a column muxing path and a single sense amp and write driver datapath.

**Top level schematic view**
![SRAM top](docs/figures/SRAM.png)

**Array block view**
![1KB SRAM Cell Array](docs/figures/1KB_SRAM_Cell_Array.png)

**Peripheral integration view**
![1kb SRAM circuitry](docs/figures/1kb_sram_circuitry.png)


### 1.2 Addressing and control

The design is documented as a 1 bit I/O memory slice at the top level:

- `ADDR[9:0]` address
  - lower bits select the row via the row decoder and wordlines
  - upper bits select the column via the column decoder and column mux path
- `DIN` data input (write)
- `DOUT` data output (read)
- Control signals (names may vary by testbench and symbol):
  - `PR` precharge enable for BL/BLB
  - `WE` write enable for write driver
  - `SE` or `SA_EN` sense amplifier enable
  - decoded wordline enable and column select enables

---

## 2. Bitline precharge and equalization

Precharge sets a known initial condition before each access by pulling BL and BLB to VDD and equalizing them.

**Schematic**
![precharge](docs/figures/precharge.png)

**Testbench**
![precharge_tb](docs/figures/precharge_tb.png)

**Simulation**
![precharge_sim](docs/figures/precharge_sim.png)

Notes:
- When `PR` is asserted, BL and BLB charge to VDD and remain equalized.
- When `PR` is deasserted, BL and BLB float and are ready for read or write.

---

## 3. Write driver

The write driver forces the global bitlines to complementary values during a write, and is disabled during read and precharge phases.

**Schematic**
![write_driver](docs/figures/write_driver.png)

**Testbench**
![write_driver_tb](docs/figures/write_driver_tb.png)

**Simulation**
![write_driver_sim](docs/figures/write_driver_sim.png)

Notes:
- With `WE` asserted, BL and BLB follow the intended write polarity set by the input data.
- With `WE` deasserted, the driver output is effectively disconnected so precharge and read can operate.

---

## 4. Sense amplifier

During read, the selected cell creates only a small BL versus BLB differential. The sense amplifier regenerates this small difference to a full rail logic output.

**Schematic**
![senese_amp](docs/figures/senese_amp.png)

**Testbench**
![sense_amp_test](docs/figures/sense_amp_test.png)

**Simulation**
![sense_amp_test_sim](docs/figures/sense_amp_test_sim.png)

Notes:
- The sense amp is enabled after a differential develops on BL and BLB.
- Positive feedback resolves the output quickly to a stable logic level.

---

## 5. Decoding and column selection

### 5.1 Row decoder

Selects exactly one wordline for a given row address.

**Row decoder schematic**
![row_decoder](docs/figures/row_decoder.png)

**Row decoder testbench**
![row_decoder_tb](docs/figures/row_decoder_tb.png)

**Row decoder simulation**
![row_decoder_sim](docs/figures/row_decoder_sim.png)

### 5.2 Decoder sub blocks

These are the decoder building blocks used in the hierarchy.

**2 to 4 decoder**
![2-4_decoder](docs/figures/2-4_decoder.png)

**2 to 4 decoder testbench**
![2_4_decoder_tb](docs/figures/2_4_decoder_tb.png)

**2 to 4 decoder simulation**
![2_4_decoder_sim](docs/figures/2_4_decoder_sim.png)

**3 to 8 decoder**
![3-8_decoder](docs/figures/3-8_decoder.png)

**3 to 8 decoder testbench**
![3_8_decoder_tb](docs/figures/3_8_decoder_tb.png)

**3 to 8 decoder simulation**
![3_8_decoder_sim](docs/figures/3_8_decoder_sim.png)

### 5.3 Column decoder and mux path

The column decoder creates one hot selection signals. The column mux connects one selected local bitline pair to the global BL/BLB pair used by the write driver and sense amp.

**Column decoder**
![column_decoder](docs/figures/column_decoder.png)

**Column mux**
![column_mux](docs/figures/column_mux.png)

**Column mux write simulation**
![CM_write_sim](docs/figures/CM_write_sim.png)

---

## 6. One bit SRAM slice

A single cell slice is used to validate correct bitcell behavior with the surrounding periphery and control timing before scaling to the full array.

**Design of 1 bit SRAM cell**
![Design_of_1-bit_SRAM_Cell](docs/figures/Design_of_1-bit_SRAM_Cell.png)

**Simulation of 1 bit SRAM cell**
![Sim_1-bit_SRAM_Cell](docs/figures/Sim_1-bit_SRAM_Cell.png)

---

## 7. Full SRAM operation

### 7.1 Full write operation

**Write waveform**
![sram_write](docs/figures/sram_write.png)

### 7.2 Full read operation

**Read waveform**
![sram_read](docs/figures/sram_read.png)

### 7.3 Integrated simulation views

**Integrated simulation (overview)**
![sim_1kb_sram](docs/figures/sim_1kb_sram.png)

**Integrated simulation waveforms**
![sim_wavef_1kb_sram](docs/figures/sim_wavef_1kb_sram.png)

---

## 8. Cadence library navigation

All design data is inside `sram/` (Cadence library root).

Typical entry points:
- Bitcell: `sram/SRAM_cell` and benches in `sram/SRAM_cell_sim`
- Precharge: `sram/precharge_ckt` plus benches in `sram/SRAM_PCH_sim`
- Write driver: `sram/write_driver` plus benches in `sram/SRAM_WD_sim`
- Sense amp: `sram/sense_amp*` plus benches in `sram/sense_amp_sim`
- Decoders: `sram/decoder`, `sram/c#2ddecoder*`
- Integration: `sram/full_SRAM`, `sram/SRAM_full`, and benches in `sram/full_SRAM_sim`

---


