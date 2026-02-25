---
description: "armips ARM instruction set reference: ARM7 (GBA), ARM9 (NDS), ARM11 (3DS), THUMB mode. Condition codes, data processing, load/store, branch, multiply, coprocessor, shift types, constant pools (.pool), register names. Use when writing ARM/THUMB assembly with armips, patching GBA/NDS/3DS ROMs."
---

# armips ARM Instructions

## Registers

**General purpose:** `r0`-`r15`
**Aliases:** `sp` (r13), `lr` (r14), `pc` (r15)
**Coprocessor registers:** `c0`-`c15`
**Coprocessor numbers:** `p0`-`p15`
**Status registers:** `cpsr`, `spsr` (field specifiers: `_ctl`, `_flg`, `f`, `s`, `x`, `c`)

## Mode Switching

| Directive | Description |
|-----------|-------------|
| `.arm` | Switch to 32-bit ARM instruction mode |
| `.thumb` | Switch to 16-bit THUMB instruction mode |
| `.pool` | Insert literal pool for `ldr rx,=imm` pseudo-ops |
| `.msg` | Insert No$gba debug message |

## Condition Codes

All ARM instructions (32-bit mode) can be conditionally executed by appending a suffix:

| Code | Meaning | Flags |
|------|---------|-------|
| `eq` | Equal | Z=1 |
| `ne` | Not equal | Z=0 |
| `cs`/`hs` | Carry set / unsigned >= | C=1 |
| `cc`/`lo` | Carry clear / unsigned < | C=0 |
| `mi` | Negative | N=1 |
| `pl` | Positive or zero | N=0 |
| `vs` | Overflow | V=1 |
| `vc` | No overflow | V=0 |
| `hi` | Unsigned > | C=1, Z=0 |
| `ls` | Unsigned <= | C=0 or Z=1 |
| `ge` | Signed >= | N=V |
| `lt` | Signed < | N!=V |
| `gt` | Signed > | Z=0, N=V |
| `le` | Signed <= | Z=1 or N!=V |
| (none) | Always (default) | -- |

## Shift Types

| Shift | Description |
|-------|-------------|
| `lsl` | Logical shift left |
| `lsr` | Logical shift right |
| `asr` | Arithmetic shift right |
| `ror` | Rotate right |
| `rrx` | Rotate right extended (single bit through carry) |

## ARM Mode Instructions (32-bit)

All instructions accept optional `{cond}` suffix. Instructions marked `{s}` optionally set flags.

### Data Processing

`and{cond}{s}`, `eor{cond}{s}`/`xor{cond}{s}`, `sub{cond}{s}`, `rsb{cond}{s}`, `add{cond}{s}`, `adc{cond}{s}`, `sbc{cond}{s}`, `rsc{cond}{s}`, `orr{cond}{s}`, `mov{cond}{s}`, `bic{cond}{s}`, `mvn{cond}{s}`

### Test/Compare

`tst{cond}`, `teq{cond}`, `cmp{cond}`, `cmn{cond}`

### Shift

`lsl{cond}{s}`, `lsr{cond}{s}`, `asr{cond}{s}`, `ror{cond}{s}`

### Branch

| Instruction | Description |
|-------------|-------------|
| `b{cond}` | Branch |
| `bl{cond}` | Branch with link |
| `bx{cond}` | Branch and exchange (ARM/THUMB switch) |
| `blx{cond}` | Branch with link and exchange |

### Multiply

`mul{cond}{s}`, `mla{cond}{s}`, `umull{cond}{s}`, `umlal{cond}{s}`, `smull{cond}{s}`, `smlal{cond}{s}`

**Halfword multiply:** `smlaXY{cond}`, `smlawY{cond}`, `smulwY{cond}`, `smlalXY{cond}`, `smulXY{cond}` (X/Y = `b` bottom or `t` top halfword)

### Load/Store Single

| Instruction | Description |
|-------------|-------------|
| `ldr{cond}` | Load word |
| `str{cond}` | Store word |
| `ldr{cond}b` | Load unsigned byte |
| `str{cond}b` | Store byte |
| `ldr{cond}h` | Load unsigned halfword |
| `str{cond}h` | Store halfword |
| `ldr{cond}sb` | Load signed byte |
| `ldr{cond}sh` | Load signed halfword |
| `ldr{cond}d` | Load doubleword |
| `str{cond}d` | Store doubleword |
| `ldr{cond}t` | Load word, user mode |
| `str{cond}t` | Store word, user mode |
| `ldr{cond}bt` | Load byte, user mode |
| `str{cond}bt` | Store byte, user mode |

### Load/Store Multiple

`ldm{cond}{mode}`, `stm{cond}{mode}`

**Addressing modes:** `ib` (increment before), `ia` (increment after), `db` (decrement before), `da` (decrement after), `fd` (full descending), `fa` (full ascending), `ed` (empty descending), `ea` (empty ascending)

**Stack shortcuts:** `push{cond}`, `pop{cond}`

### PSR Transfer

`mrs{cond}`, `msr{cond}`

### Swap

`swp{cond}`, `swp{cond}b`

### Coprocessor

`cdp{cond}`, `cdp2`, `mcr{cond}`, `mcr2`, `mrc{cond}`, `mrc2`, `mcrr{cond}`, `mrrc{cond}`

### Miscellaneous

| Instruction | Description |
|-------------|-------------|
| `nop` | No operation |
| `swi{cond}` | Software interrupt |
| `bkpt` | Breakpoint |
| `clz{cond}` | Count leading zeros |
| `pld` | Preload data |
| `adr{cond}{s}` | Calculate PC-relative address |
| `qadd{cond}` | Saturating add |
| `qsub{cond}` | Saturating subtract |
| `qdadd{cond}` | Saturating double and add |
| `qdsub{cond}` | Saturating double and subtract |

## ARM Pseudo-Operations

| Pseudo-op | Description |
|-----------|-------------|
| `ldr rx,=immediate` | Load any 32-bit constant via literal pool (requires `.pool`) |
| `add rx,=immediate` | PC-relative address calculation |

armips automatically optimizes: `mov`/`mvn` when immediate is invertible, `bic`/`and` conversion, `cmp`/`cmn` conversion.

## THUMB Mode Instructions (16-bit)

THUMB instructions are 16-bit and do NOT support condition codes (except conditional branches).

### Arithmetic/Logic

`add`, `sub`, `mov`, `cmp`, `neg`, `mul`, `and`, `eor`/`xor`, `orr`, `bic`, `mvn`, `tst`, `cmn`, `adc`, `sbc`

### Shift

`lsl`/`asl`, `lsr`, `asr`, `ror`

### Load/Store

`ldr`, `str`, `ldrb`, `strb`, `ldrh`, `strh`, `ldrsb`/`ldsb`, `ldrsh`/`ldsh`

### Stack

`push`, `pop`

### Multi-Load/Store

`stmia`, `ldmia`

### Branch

| Instruction | Description |
|-------------|-------------|
| `b` | Unconditional branch |
| `bl` | Branch with link (long) |
| `blh` | Branch with link high (first half of bl) |
| `bx` | Branch and exchange |
| `blx` | Branch with link and exchange |

### Conditional Branch

`beq`, `bne`, `bcs`/`bhs`, `bcc`/`blo`, `bmi`, `bpl`, `bvs`, `bvc`, `bhi`, `bls`, `bge`, `blt`, `bgt`, `ble`

### System

`swi`, `bkpt`, `nop`

## Constant Pools

When using `ldr rx,=value`, armips stores the constant in a literal pool. Place `.pool` after your code to emit the pool:

```
.thumb
  ldr  r0,=0x12345678
  ldr  r1,=0xDEADBEEF
  bx   lr
.pool
```

The pool must be reachable from the `ldr` instruction (within +-4KB for ARM, +-1KB for THUMB).
