---
description: "armips MIPS instruction set reference: R3000 (PSX), R4000 (N64), Allegrex/VFPU (PSP), RSP vectors (N64), EmotionEngine (PS2). Registers, branch/ALU/load/store/coprocessor instructions, FPU, built-in macros (li/la/blt/bgt), load-delay detection (.fixloaddelay). Use when writing MIPS assembly with armips, patching PSX/PSP/N64/PS2 ROMs, or debugging MIPS instruction syntax."
---

# armips MIPS Instructions

## Registers

### General Purpose (r0-r31)

```
r0/zero    r1/at     r2/v0    r3/v1
r4/a0      r5/a1     r6/a2    r7/a3
r8/t0      r9/t1     r10/t2   r11/t3
r12/t4     r13/t5    r14/t6   r15/t7
r16/s0     r17/s1    r18/s2   r19/s3
r20/s4     r21/s5    r22/s6   r23/s7
r24/t8     r25/t9    r26/k0   r27/k1
r28/gp     r29/sp    r30/fp   r31/ra
```

**FPU Registers:** `f0`-`f31`
**FPU Control:** `fcr0`/`fir`, `fcr31`/`fcsr`

### COP0 Registers

```
index, random, entrylo, entrylo0, entrylo1, context, pagemask,
wired, badvaddr, count, entryhi, compare, status/sr, cause, epc,
prid, config, lladdr, watchlo, watchhi, xcontext, badpaddr,
ecc, perr, cacheerr, taglo, taghi, errorepc
```

### PSX COP2 (GTE) Data Registers

```
vxy0, vz0, vxy1, vz1, vxy2, vz2, rgbc, otz,
ir0, ir1, ir2, ir3, sxy0, sxy1, sxy2, sxyp,
sz0, sz1, sz2, sz3, rgb0, rgb1, rgb2, res1,
mac0, mac1, mac2, mac3, irgb, orgb, lzcs, lzcr
```

### PSX COP2 (GTE) Control Registers

```
rt0-rt4, trx, try, trz, llm0-llm4, rbk, gbk, bbk,
lcm0-lcm4, rfc, gfc, bfc, ofx, ofy, h, dqa, dqb, zsf3, zsf4, flag
```

### RSP Registers

**Vector:** `v0`-`v31`
**Vector control:** `vco`, `vcc`, `vce`
**COP0:** `sp_mem_addr`, `sp_dram_addr`, `sp_rd_len`, `sp_wr_len`, `sp_status`, `sp_dma_full`, `sp_dma_busy`, `sp_semaphore`, `dpc_start`, `dpc_end`, `dpc_current`, `dpc_status`, `dpc_clock`, `dpc_bufbusy`, `dpc_pipebusy`, `dpc_tmem`

### PS2 VU Registers

`vf0`-`vf31`

## Core Instructions

### Branch

`j`, `jal`, `jr`, `jalr`, `beq`, `bne`, `beqz`, `bnez`, `blez`, `bgtz`, `bltz`, `bgez`, `bltzal`, `bgezal`, `b`, `bal`

**Branch-likely:** `beql`, `bnel`, `beqzl`, `bnezl`, `blezl`, `bgtzl`, `bltzl`, `bgezl`, `bltzall`, `bgezall`

### Arithmetic

| Category | Instructions |
|----------|-------------|
| Immediate | `addi`, `addiu`, `subi`, `subiu`, `slti`, `sltiu`, `andi`, `ori`, `xori`, `lui` |
| Register | `add`, `addu`, `sub`, `subu`, `slt`, `sltu`, `and`, `or`, `xor`/`eor`, `nor` |
| Pseudo | `move`, `clear`, `neg`, `negu`, `not`, `sgt`, `sgtu` |
| Multiply | `mult`, `multu`, `div`, `divu`, `mfhi`, `mthi`, `mflo`, `mtlo` |

**64-bit (R4000+):** `daddi`, `daddiu`, `dsubi`, `dsubiu`, `dadd`, `daddu`, `dsub`, `dsubu`, `dmult`, `dmultu`, `ddiv`, `ddivu`, `dneg`, `dnegu`

**PS2 extensions:** `madd`, `maddu`, `msub`, `msubu`, `mfsa`, `mtsa`, `clz`, `clo`, `max`, `min`

### Shift

`sll`, `srl`, `sra`, `sllv`, `srlv`, `srav`, `rotr`, `rotrv`

**64-bit:** `dsll`, `dsrl`, `dsra`, `dsll32`, `dsrl32`, `dsra32`

### Load/Store

| Category | Instructions |
|----------|-------------|
| Load | `lb`, `lbu`, `lh`, `lhu`, `lw`, `lwu`, `ld`, `lq`, `ll`, `lld` |
| Store | `sb`, `sh`, `sw`, `sd`, `sq`, `sc`, `scd` |
| Unaligned | `lwl`, `lwr`, `ldl`, `ldr`, `swl`, `swr`, `sdl`, `sdr` |
| FPU | `lwc1`/`l.s`, `swc1`/`s.s`, `ldc1`/`l.d`, `sdc1`/`s.d` |
| COP2 | `lwc2`/`lv.s`, `swc2`/`sv.s`, `lqc2`, `sqc2` |
| Cache | `cache` |

### System

`syscall`, `break`, `sync`, `nop`

### Trap (R4000+)

`tge`, `tgeu`, `tlt`, `tltu`, `teq`, `tne`, `tgei`, `tgeiu`, `tlti`, `tltiu`, `teqi`, `tnei`

### COP0

`mfc0`, `mtc0`, `dmfc0`, `dmtc0`, `tlbr`, `tlbwi`, `tlbwr`, `tlbp`, `rfe`, `eret`

### COP2 Transfer

`mfc2`, `mtc2`, `cfc2`, `ctc2`, `cop2`

**PSP VFPU transfer:** `mfv`, `mtv`, `mfvc`, `mtvc`

## FPU Instructions

### Single-Precision

`add.s`, `sub.s`, `mul.s`, `div.s`, `sqrt.s`, `abs.s`, `mov.s`, `neg.s`

**Rounding:** `round.w.s`, `trunc.w.s`, `ceil.w.s`, `floor.w.s`, `round.l.s`, `trunc.l.s`, `ceil.l.s`, `floor.l.s`

**Conversion:** `cvt.d.s`, `cvt.w.s`, `cvt.l.s`

**PS2 extensions:** `rsqrt.w.s`, `adda.s`, `suba.s`, `mula.s`, `madd.s`, `msub.s`, `madda.s`, `msuba.s`, `max.s`, `min.s`

### Double-Precision

`add.d`, `sub.d`, `mul.d`, `div.d`, `sqrt.d`, `abs.d`, `mov.d`, `neg.d`

**Rounding:** `round.w.d`, `trunc.w.d`, `ceil.w.d`, `floor.w.d`, `round.l.d`, `trunc.l.d`, `ceil.l.d`, `floor.l.d`

**Conversion:** `cvt.s.d`, `cvt.w.d`, `cvt.l.d`

### Integer Conversion

`cvt.s.w`, `cvt.d.w`, `cvt.s.l`, `cvt.d.l`

### FPU Comparisons

**Single:** `c.f.s`, `c.un.s`, `c.eq.s`, `c.ueq.s`, `c.olt.s`/`c.lt.s`, `c.ult.s`, `c.ole.s`/`c.le.s`, `c.ule.s`, `c.sf.s`, `c.ngle.s`, `c.seq.s`, `c.ngl.s`, `c.nge.s`, `c.ngt.s`

**Double:** `c.f.d`, `c.un.d`, `c.eq.d`, `c.ueq.d`, `c.olt.d`, `c.ult.d`, `c.ole.d`, `c.ule.d`, `c.sf.d`, `c.ngle.d`, `c.seq.d`, `c.ngl.d`, `c.lt.d`, `c.nge.d`, `c.le.d`, `c.ngt.d`

**Branch on FPU:** `bc1f`, `bc1t`, `bc1fl`, `bc1tl`

### FPU Transfer

`mfc1`, `mtc1`, `dmfc1`, `dmtc1`, `cfc1`, `ctc1`

## PSP VFPU Instructions

Size suffix `.S` = `.s` (single), `.p` (pair), `.t` (triple), `.q` (quad).

### Arithmetic

`vadd.S`, `vsub.S`, `vmul.S`, `vdiv.S`, `vsbn.S`, `vdot.S`, `vscl.S`, `vhdp.S`, `vdet.S`, `vcrs.S`

### Compare/Min/Max

`vcmp.S`, `vmin.S`, `vmax.S`, `vscmp.S`, `vsge.S`, `vslt.S`

### Unary/Constants

`vmov.S`, `vabs.S`, `vneg.S`, `vzero.S`, `vone.S`, `vidt.S`, `vsat0.S`, `vsat1.S`, `vrcp.S`, `vrsq.S`, `vsin.S`, `vcos.S`, `vexp2.S`, `vlog2.S`, `vsqrt.S`, `vasin.S`, `vnrcp.S`, `vnsin.S`, `vrexp2.S`, `vcst.S`

### Type Conversion

`vf2in.S`, `vf2iz.S`, `vf2iu.S`, `vf2id.S`, `vi2f.S`, `vf2h.S`, `vh2f.S`, `vuc2i.S`, `vc2i.S`, `vus2i.S`, `vs2i.S`, `vi2uc.S`, `vi2c.S`, `vi2us.S`, `vi2s.S`

### Conditional Move

`vcmovt.S`, `vcmovf.S`

### Sort/Butterfly

`vsrt1.S`, `vsrt2.S`, `vsrt3.S`, `vsrt4.S`, `vbfy1.S`, `vbfy2.S`

### Reduction

`vfad.S`, `vavg.S`, `vocp.S`, `vsocp.S`, `vsgn.S`

### Matrix

`vmmul.S`, `vmscl.S`, `vmmov.S`, `vmidt.S`, `vmzero.S`, `vmone.S`, `vrot.S`

### Transform

`vtfm2.p`, `vhtfm2.p`, `vtfm3.t`, `vhtfm3.t`, `vtfm4.q`, `vcrsp.t`, `vqmul.q`

### Prefix/Immediate

`vpfxs`, `vpfxt`, `vpfxd`, `viim.s`, `vfim.s`

### Transfer/Color

`vmfv.S`, `vmtv.S`, `vmfvc.S`, `vmtvc.S`, `vt4444.S`, `vt5551.S`, `vt5650.S`

### Random

`vrnds.S`, `vrndi.S`, `vrndf1.S`, `vrndf2.S`

### Misc

`vlgb.S`, `vwbn.S`

### VFPU Load/Store

`lv.s`, `sv.s`, `lv.q`, `sv.q`, `ulv.q`, `usv.q`, `lvl.q`, `lvr.q`, `svl.q`, `svr.q`

## RSP Vector Instructions

### Arithmetic

`vmulf`, `vmulu`, `vrndp`, `vmulq`, `vmudl`, `vmudm`, `vmudn`, `vmudh`, `vmacf`, `vmacu`, `vrndn`, `vmacq`, `vmadl`, `vmadm`, `vmadn`, `vmadh`, `vadd`, `vsub`, `vsut`, `vabs`, `vaddc`, `vsubc`, `vaddb`, `vsubb`, `vaccb`, `vsucb`, `vsad`, `vsac`, `vsum`, `vsar`, `vacc`, `vsuc`

### Compare/Select

`vlt`, `veq`, `vne`, `vge`, `vcl`, `vch`, `vcr`, `vmrg`

### Logical

`vand`, `vnand`, `vor`, `vnor`, `vxor`, `vnxor`

### Reciprocal/Sqrt

`vrcp`, `vrcpl`, `vrcph`, `vmov`, `vrsq`, `vrsql`, `vrsqh`

### Special

`vnop`, `vextt`, `vextq`, `vextn`, `vinst`, `vinsq`, `vinsn`, `vnull`

### RSP Vector Load/Store

**Load:** `lbv`, `lsv`, `llv`, `ldv`, `lqv`, `lrv`, `lpv`, `luv`, `lhv`, `lfv`, `ltv`
**Store:** `sbv`, `ssv`, `slv`, `sdv`, `sqv`, `srv`, `spv`, `suv`, `shv`, `sfv`, `swv`, `stv`

## PS2 Multimedia

`mand`, `mnand`, `mvor`, `mnor`, `mvxor`, `mvnxor`

**PS2 COP2 (VU) branch:** `bvf`, `bvt`, `bvfl`, `bvtl`

## Built-in Macros

### Immediate Loading

| Macro | Description |
|-------|-------------|
| `li reg,imm` | Load 32-bit immediate (uses `lui`+`ori` or `addiu` as needed) |
| `la reg,imm` | Load address |
| `li.s freg,imm` | Load single-precision float immediate |

### Extended Load/Store

These macros accept `imm`, `imm(reg)`, or `(reg)` addressing and auto-expand to multi-instruction sequences when the offset exceeds 16 bits:

**Load:** `lb`, `lbu`, `lh`, `lhu`, `lw`, `lwu`, `ld`, `lwc1`, `lwc2`, `ldc1`, `ldc2`
**Store:** `sb`, `sh`, `sw`, `sd`, `swc1`, `swc2`, `sdc1`, `sdc2`

### Unaligned Load/Store

| Load | Store |
|------|-------|
| `ulh reg,addr` | `ush reg,addr` |
| `ulhu reg,addr` | `usw reg,addr` |
| `ulw reg,addr` | `usd reg,addr` |
| `uld reg,addr` | |

### Branch Macros

Register-register and register-immediate forms. All expand to `slt`+branch sequences:

`blt`, `bltu`, `bgt`, `bgtu`, `bge`, `bgeu`, `ble`, `bleu`

**With immediate:** `blt reg,imm,dest` / `blt reg1,reg2,dest`

**Branch-likely variants:** `bltl`, `bltul`, `bgtl`, `bgtul`, `bgel`, `bgeul`, `blel`, `bleul`, `bnel`, `beql` (immediate forms)

### Set Macros

Register-immediate and register-register forms:

`sge`, `sgeu`, `sle`, `sleu`, `sne`, `seq`

**With immediate:** `slt reg1,reg2,imm` / `sltu reg1,reg2,imm` / `sgt reg1,reg2,imm` / `sgtu reg1,reg2,imm`

### Rotate Macros

`rol reg1,reg2,reg3` / `rol reg1,reg2,imm`
`ror reg1,reg2,reg3` / `ror reg1,reg2,imm`

### Absolute Value

`abs reg1,reg2` (32-bit), `dabs reg1,reg2` (64-bit)

### Upper/Lower Suffixes

`.u` and `.l` suffixes on `li`, `la`, load, and store macros emit only the upper or lower instruction of a two-instruction expansion.

## PSP Allegrex Extensions

`wsbh`, `wsbw`, `seb`, `seh`, `bitrev`, `ext`, `ins`

## Load Delay Detection

MIPS R3000 has a one-cycle load delay -- the loaded value is not available in the next instruction.

| Directive | Description |
|-----------|-------------|
| `.fixloaddelay` | Auto-insert `nop` to resolve load delay hazards |
| `.resetdelay` | Clear load delay tracking state |

armips warns when a load result register is used by the immediately following instruction. Use `.fixloaddelay` to have armips automatically insert nops.

## ELF Import (PSP)

| Directive | Description |
|-----------|-------------|
| `.importobj "file.o"` | Import ELF object file |
| `.importlib "file.a",ctor,dtor` | Import static library with constructor/destructor labels |
