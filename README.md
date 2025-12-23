# 1 kb 6T SRAM

Schematic level design and simulation of a 1 kb SRAM built from a standard 6T bitcell and full peripheral circuitry (precharge, decoders, column muxing, write driver, sense amplifier). Implemented in Cadence Virtuoso, with simulations validating each block and the integrated memory.

---

## At a glance

| Item | Value |
|---|---|
| Memory type | 6T SRAM |
| Capacity | 1 kb |
| Organization | 32 rows x 32 columns |
| I/O width | 1 bit (column selected through muxing) |
| Technology | 180 nm TSMC PDK |
| Design level | Schematic (transistor level blocks and hierarchical integration) |
| Tool | Cadence Virtuoso |
| Documentation assets | `docs/6T_SRAM.pdf`, `docs/figures/*.png` |

---

## Contents

- [Repository structure](#repository-structure)
- [Top level architecture](#top-level-architecture)
- [Peripheral circuits](#peripheral-circuits)
  - [Precharge and equalization](#precharge-and-equalization)
  - [Write driver](#write-driver)
  - [Sense amplifier](#sense-amplifier)
- [Decoders and column path](#decoders-and-column-path)
- [1 bit SRAM slice](#1-bit-sram-slice)
- [Full SRAM read and write](#full-sram-read-and-write)
- [Cadence library navigation](#cadence-library-navigation)
- [Rebuilding the presentation (optional)](#rebuilding-the-presentation-optional)

---

## Repository structure

This repo commits only the Cadence library root `sram/` plus documentation under `docs/`.

**Key files**
- `sram/` : Cadence Virtuoso library root (schematics, symbols, benches, simulation states)
- `docs/6T_SRAM.pdf` : presentation slides
- `docs/figures/` : exported schematics and waveform plots embedded below

---

## Top level architecture

### 1 kb array overview

- Capacity: **1 kb**
- Organization: **32 rows x 32 columns**
- Storage: **6T SRAM bitcell array**
- Shared global periphery: precharge, write driver, sense amp
- Address selection: row decoder selects one wordline, column decoder + mux selects one BL/BLB pair to connect to global bitlines

### Addressing and control

Top level is documented as a **1 bit I/O** datapath:

- `ADDR[9:0]`
  - lower bits select the row via row decoder and wordlines
  - upper bits select the column via column decoder and column mux path
- `DIN` data input (write)
- `DOUT` data output (read)
- Control signals (names may vary by symbol and benches):
  - `PR` precharge enable for BL/BLB
  - `WE` write enable for write driver
  - `SE` or `SA_EN` sense amplifier enable

### Top level figures

<p align="center">
  <img src="docs/figures/SRAM.png" alt="SRAM top" width="92%">
</p>

<p align="center">
  <img src="docs/figures/1KB_SRAM_Cell_Array.png" alt="1KB SRAM Cell Array" width="92%">
</p>

<p align="center">
  <img src="docs/figures/1kb_sram_circuitry.png" alt="1kb SRAM circuitry" width="92%">
</p>

---

## Peripheral circuits

## Precharge and equalization

Precharge sets a known initial condition before each access by pulling **BL** and **BLB** to VDD and equalizing them.

<details>
  <summary>Show precharge schematic, testbench, and simulation</summary>

  <p align="center">
    <img src="docs/figures/precharge.png" alt="Precharge schematic" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/precharge_tb.png" alt="Precharge testbench" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/precharge_sim.png" alt="Precharge simulation" width="95%">
  </p>

</details>

---

## Write driver

The write driver forces the global bitlines to complementary values during a write. During read and precharge, the driver is disabled so the bitlines can be precharged and conditionally discharged.

<details>
  <summary>Show write driver schematic, testbench, and simulation</summary>

  <p align="center">
    <img src="docs/figures/write_driver.png" alt="Write driver schematic" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/write_driver_tb.png" alt="Write driver testbench" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/write_driver_sim.png" alt="Write driver simulation" width="95%">
  </p>

</details>

---

## Sense amplifier

During read, the selected cell creates only a small BL versus BLB differential. The sense amplifier regenerates this small difference to a full rail logic output.

<details>
  <summary>Show sense amp schematic, testbench, and simulation</summary>

  <p align="center">
    <img src="docs/figures/senese_amp.png" alt="Sense amplifier schematic" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/sense_amp_test.png" alt="Sense amplifier testbench" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/sense_amp_test_sim.png" alt="Sense amplifier simulation" width="95%">
  </p>

</details>

---

## Decoders and column path

Row decoding selects exactly one wordline. Column decoding and muxing route one selected local BL/BLB pair to the shared global bitlines.

### Row decoder

<details>
  <summary>Show row decoder schematic, testbench, and simulation</summary>

  <p align="center">
    <img src="docs/figures/row_decoder.png" alt="Row decoder schematic" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/row_decoder_tb.png" alt="Row decoder testbench" width="85%">
  </p>

  <p align="center">
    <img src="docs/figures/row_decoder_sim.png" alt="Row decoder simulation" width="95%">
  </p>

</details>

### Decoder building blocks

<details>
  <summary>Show 2 to 4 and 3 to 8 decoder blocks with benches</summary>

  <p align="center">
    <img src="docs/figures/2-4_decoder.png" alt="2 to 4 decoder" width="75%">
  </p>

  <p align="center">
    <img src="docs/figures/2_4_decoder_tb.png" alt="2 to 4 decoder testbench" width="75%">
  </p>

  <p align="center">
    <img src="docs/figures/2_4_decoder_sim.png" alt="2 to 4 decoder simulation" width="95%">
  </p>

  <p align="center">
    <img src="docs/figures/3-8_decoder.png" alt="3 to 8 decoder" width="75%">
  </p>

  <p align="center">
    <img src="docs/figures/3_8_decoder_tb.png" alt="3 to 8 decoder testbench" width="75%">
  </p>

  <p align="center">
    <img src="docs/figures/3_8_decoder_sim.png" alt="3 to 8 decoder simulation" width="95%">
  </p>

</details>

### Column decoder and mux

<details>
  <summary>Show column decoder, column mux, and column mux write simulation</summary>

  <p align="center">
    <img src="docs/figures/column_decoder.png" alt="Column decoder" width="75%">
  </p>

  <p align="center">
    <img src="docs/figures/column_mux.png" alt="Column mux" width="75%">
  </p>

  <p align="center">
    <img src="docs/figures/CM_write_sim.png" alt="Column mux write simulation" width="95%">
  </p>

</details>

---

## 1 bit SRAM slice

A single cell slice is used to validate correct behavior with periphery and control sequencing before scaling to the full array.

<details>
  <summary>Show 1 bit SRAM design and simulation</summary>

  <p align="center">
    <img src="docs/figures/Design_of_1-bit_SRAM_Cell.png" alt="Design of 1-bit SRAM cell" width="92%">
  </p>

  <p align="center">
    <img src="docs/figures/Sim_1-bit_SRAM_Cell.png" alt="Simulation of 1-bit SRAM cell" width="95%">
  </p>

</details>

---

## Full SRAM read and write

### Write operation
<p align="center">
  <img src="docs/figures/sram_write.png" alt="SRAM write waveform" width="95%">
</p>

### Read operation
<p align="center">
  <img src="docs/figures/sram_read.png" alt="SRAM read waveform" width="95%">
</p>

<details>
  <summary>Show integrated simulation overview and waveforms</summary>

  <p align="center">
    <img src="docs/figures/sim_1kb_sram.png" alt="Integrated simulation overview" width="95%">
  </p>

  <p align="center">
    <img src="docs/figures/sim_wavef_1kb_sram.png" alt="Integrated simulation waveforms" width="98%">
  </p>

</details>

---

## Cadence library navigation

All design data is inside `sram/` (Cadence library root). Directory names follow Cadence export conventions and may include encoded characters such as `#2d` in place of `-`.

Typical entry points:
- Bitcell: `sram/SRAM_cell` and benches in `sram/SRAM_cell_sim`
- Precharge: `sram/precharge_ckt` plus benches in `sram/SRAM_PCH_sim`
- Write driver: `sram/write_driver` plus benches in `sram/SRAM_WD_sim`
- Sense amp: `sram/sense_amp*` plus benches in `sram/sense_amp_sim`
- Decoders: `sram/decoder`, `sram/c#2ddecoder*`
- Full integration: `sram/full_SRAM`, `sram/SRAM_full`, and benches in `sram/full_SRAM_sim`

---

