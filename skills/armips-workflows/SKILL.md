---
description: "armips ROM hacking workflows: PSX binary patching, text hacking with custom tables (.loadtable/.string), multi-file overlays, GBA/NDS ARM/THUMB patching, Ghidra symbol integration, symbol file generation for emulator debugging. Use when planning or executing ROM hacking tasks with armips."
---

# armips ROM Hacking Workflows

## Workflow A: Patch a PSX Binary

Modify code at a known address in a PSX executable.

1. **Identify the target** -- use Ghidra or an emulator debugger to find the function address and the file offset. The PSX typically loads executables at `0x80010000` with an 0x800-byte header.

2. **Write the patch:**
```
.psx
.open "SLUS_123.45","SLUS_123.45_patched",0x80010000-0x800

.org 0x8003A000            ; target address in RAM
  li    a0,0x01            ; new argument value
  jal   TargetFunction     ; call the function
  nop                      ; branch delay slot

.close
```

3. **Assemble:**
```bash
armips patch.asm -sym symbols.sym
```

4. **Verify** -- load the patched file in an emulator, check the symbol file for correct addresses.

**Notes:**
- `.open` with two filenames preserves the original -- always recommended
- `HeaderSize` = RAM load address minus file header size
- Use `.area` to prevent overwriting adjacent code:
  ```
  .org 0x8003A000
  .area 0x20               ; max 32 bytes for this patch
    li    a0,0x01
    jal   TargetFunction
    nop
  .endarea
  ```

## Workflow B: Text Hacking with Custom Tables

Insert translated text using a custom character encoding table.

1. **Create the table file** (`table.tbl`):
```
02=a
03=b
04=c
...
1A=z
1B=
1C=.
1D=!
1E=?
/FF
```

The `/FF` line sets the string terminator byte.

2. **Write the insertion script:**
```
.psx
.open "SLUS_123.45","SLUS_123.45_patched",0x80010000-0x800

.loadtable "table.tbl"

.org 0x80050000            ; text data address
.string "Hello world!"     ; encoded using table, terminated with 0xFF

.org 0x80050100
.stringn "No terminator"   ; encoded but no terminator appended

.close
```

3. **Multi-line text blocks:**
```
.org 0x80050000
.string "Line one"
.string "Line two"
.string "Line three"
```

**Tips:**
- Use `.stringn` when the game uses fixed-length text fields
- Multiple `.loadtable` calls can switch between encodings mid-file
- Shift-JIS text uses `.sjis` / `.sjisn` directly, no table needed

## Workflow C: Multi-File Overlay Patching

Patch multiple files in a single assembly run, with cross-file label references.

```
.psx

; Main executable
.open "SLUS_123.45","SLUS_123.45_patched",0x80010000-0x800
.org 0x8003A000
  jal   NewFunction        ; references label in overlay
  nop
.close

; Overlay file
.open "OVL_01.BIN","OVL_01.BIN_patched",0x80100000
.org 0x80100000
NewFunction:
  li    v0,1
  jr    ra
  nop
.close
```

Labels defined in one file are visible in all subsequently opened files within the same assembly run.

## Workflow D: Patch a GBA/NDS ROM

ARM/THUMB patching for Nintendo platforms.

### GBA (ARM7)

```
.gba
.open "game.gba","game_patched.gba",0x08000000

; ARM mode patch
.org 0x08001000
.arm
  mov   r0,#0x10
  bl    NewRoutine
  bx    lr

; THUMB mode patch (for size-constrained areas)
.org 0x08002000
.thumb
NewRoutine:
  push  {lr}
  mov   r0,#1
  pop   {pc}

.pool                      ; emit literal pool
.close
```

### NDS (ARM9)

```
.nds
.open "arm9.bin","arm9_patched.bin",0x02000000

.org 0x02001000
.arm
  ldr   r0,=0x12345678     ; uses literal pool
  bl    CustomFunc
  bx    lr

.org 0x0200F000
CustomFunc:
  mov   r0,#0
  bx    lr

.pool
.close
```

**Notes:**
- GBA header starts at `0x08000000`
- NDS ARM9 binary typically loads at `0x02000000`
- Always place `.pool` after code sections that use `ldr rx,=imm`
- Use `.thumb` in GBA for space efficiency (16-bit instructions)

## Workflow E: Ghidra Symbol Integration

Use Ghidra analysis results as labels in armips patches.

1. **Export symbols from Ghidra** -- in GhidraMCP, list functions and note addresses:
```
list_functions(offset=0, limit=500)
```

2. **Define labels in armips** matching Ghidra's analysis:
```
.psx
.definelabel ParseInput,       0x80023A40
.definelabel RenderFrame,      0x80024B00
.definelabel LoadTexture,      0x80031200
.definelabel StringTable,      0x80050000

.open "SLUS_123.45","SLUS_123.45_patched",0x80010000-0x800

.org 0x80023A40
  ; patch ParseInput using other known labels
  jal   LoadTexture
  nop

.close
```

3. **Automate with `-definelabel`** on the command line:
```bash
armips patch.asm \
  -definelabel ParseInput 0x80023A40 \
  -definelabel RenderFrame 0x80024B00
```

**Tip:** For large symbol sets, create an `include` file with `.definelabel` directives and use `.include "symbols.inc"`.

## Workflow F: Symbol Files for Emulator Debugging

Generate symbol files that emulator debuggers can load.

1. **Generate sym format** (NO$PSX / NO$GBA / BizHawk / Mesen):
```bash
armips patch.asm -sym symbols.sym
```

2. **Generate sym2 format** (PCSX2 / PPSSPP):
```bash
armips patch.asm -sym2 symbols.sym2
```

3. **Use both simultaneously:**
```bash
armips patch.asm -sym debug.sym -sym2 debug.sym2
```

4. **Control symbol output** in source -- exclude internal/temporary labels:
```
.sym off
@@temp_loop:
  nop
  bnez  a0,@@temp_loop
  nop
.sym on

ExportedLabel:             ; this appears in symbol file
  nop
```

**Tips:**
- Function labels (`.func`/`.endfunc`) produce cleaner symbol files
- Use meaningful label names -- they appear directly in the debugger
- `.definelabel` symbols are also exported to symbol files
