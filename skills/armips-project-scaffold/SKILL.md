---
description: "Interactive scaffold generator for armips ASM projects: creates recommended directory structure, main.asm entry point, common includes (addresses, strings, macros, values), character encoding tables, and per-overlay template files. Use when a user wants to start a new armips project for patching, modding, hacking, or translating ROMs, or when you detect the user is writing assembly modifications without an organized structure."
---

# armips Project Scaffold Generator

> This workflow generates the project structure described in **armips-project-structure**.

## 1. When to Activate

**Proactive activation** -- Claude detects that the user wants to start writing assembly modifications with armips and no `main.asm` exists in the working directory. Trigger terms: patch, mod, hack, modifica, traduzione, translation hack, romhack, rom hack, or any intent to modify binaries/ROMs with armips. Claude proposes:

> *"Do you want me to create the recommended project structure to organize your ASM modifications?"*

**Explicit activation** -- The user directly asks to create the project structure (e.g. "create the ASM project structure", "scaffold the patch project", "set up the armips project").

## 2. Information to Gather

Ask the user **one question at a time**, in this order:

1. **Architecture target?** (`.psx`, `.gba`, `.nds`, `.n64`, `.psp`, `.ps2`, etc.)
2. **How many and which binaries/overlays to modify?** (file names)
3. **Character encoding tables (`.tbl`) needed? How many?**
4. **Are there repetitive modifications that apply to multiple overlays with the same logic but different addresses?** (if yes, a separate `common/shared_patches.inc` file will be created)
5. **Is there a main binary with free space usable for cross-binary string relocation?**

## 3. Generated Structure

After gathering answers, generate the following directory tree. Use the user's actual values in place of `<placeholders>`:

```
<project>/
├── main.asm
├── common/
│   ├── macros.inc
│   ├── shared_patches.inc   (if shared macros needed)
│   ├── addresses.inc
│   ├── values.inc
│   ├── strings.inc
│   └── <name>.tbl           (one per encoding table)
├── <overlay_1>/
│   └── feature.asm
├── <overlay_2>/
│   └── feature.asm
└── <shared_binary>/          (if cross-binary relocation needed)
    └── free_space.asm
```

## 4. File Templates

These are the exact contents to write when generating the scaffold. Replace `<placeholders>` with user-provided values.

### main.asm

```
;; =============================================================
;; <project_name> - ASM Modifications
;; Entry point - orchestrates all modifications
;; =============================================================

.<arch>

;; --- Character encoding tables ---
.loadtable "common/<name>.tbl"

;; --- Common includes ---
.include "common/macros.inc"
.include "common/strings.inc"
.include "common/addresses.inc"
.include "common/values.inc"
;; .include "common/shared_patches.inc"  ; uncomment if shared macros exist

;; --- <Overlay 1> ---
;; Tip: use .open FILENAME,"<file>_patched",ADDR to preserve the original
.open <OVERLAY1>_FILENAME, <OVERLAY1>_ADDR
.include "<overlay1>/feature.asm"
.close

;; --- <Overlay 2> ---
.open <OVERLAY2>_FILENAME, <OVERLAY2>_ADDR
.include "<overlay2>/feature.asm"
.close
```

If multiple `.tbl` files, add one `.loadtable` line per table. One `.open`/`.close` block per binary.

### common/macros.inc

```
;; =============================================================
;; Reusable macros
;; =============================================================

;; Write null-terminated string with zero-fill padding.
;; Triggers build error if string overflows slot_size.
;; The second argument to .area zero-fills remaining bytes automatically.
.macro strpad, str, slot_size
.area slot_size, 0x00
    .string str
.endarea
.endmacro
```

### common/shared_patches.inc

Generate this file only if the user said there are repetitive modifications across overlays:

```
;; =============================================================
;; Shared parametric macros for repetitive modifications
;; Call once per overlay with overlay-specific addresses
;; =============================================================

.macro patch_shared_labels, addr_a, addr_b
    .org addr_a
        strpad LABEL_A_STR, 8
    .org addr_b
        strpad LABEL_B_STR, 8
.endmacro
```

### common/addresses.inc

```
;; =============================================================
;; Address definitions and filename constants - single source of truth
;; Naming: OVERLAY_FUNCTION_PURPOSE_ADDR for addresses
;;         OVERLAY_FILENAME for binary filenames
;; Suffixes: _LUI_ADDR (pointer load), _WIDTH_ADDR, _COLUMN_ADDR
;; =============================================================

;; --- <Overlay 1> ---
<OVERLAY1>_FILENAME equ "<file1>"
<OVERLAY1>_ADDR equ 0x00000000       ; RAM load address

;; --- <Overlay 2> ---
<OVERLAY2>_FILENAME equ "<file2>"
<OVERLAY2>_ADDR equ 0x00000000       ; RAM load address
```

### common/strings.inc

```
;; =============================================================
;; All modified/translated strings - single source of truth
;; Naming convention: CATEGORY_NAME_STR
;; Define each string once, reference everywhere
;; =============================================================

;; --- <Overlay 1> strings ---

;; --- <Overlay 2> strings ---

;; --- Shared strings (used across multiple overlays) ---
```

### common/values.inc

```
;; =============================================================
;; Computed/derived values
;; Use strlen() and arithmetic - no manual magic numbers
;; =============================================================

; CHAR_WIDTH_PX equ 8
; EXAMPLE_WIDTH equ (strlen(EXAMPLE_STR) * CHAR_WIDTH_PX)
```

### common/\<name\>.tbl

```
00=<NUL>
30=0
31=1
32=2
33=3
34=4
35=5
36=6
37=7
38=8
39=9
41=A
42=B
...
5A=Z
61=a
62=b
...
7A=z
20=
2E=.
21=!
3F=?
/00
```

This is a starting template. The user will customize it based on the game's actual character encoding.

### \<overlay\>/feature.asm

```
;; =============================================================
;; <Overlay> - <Feature description>
;; =============================================================

;; --- In-place string replacement ---
; .org <OVERLAY>_STRING_ADDR
;     strpad EXAMPLE_STR, 12

;; --- Relocated string (overflow: append past EOF) ---
; .org <OVERLAY>_EOF_ADDR
; new_string_label:
;     .string LONG_EXAMPLE_STR
;
; .org <OVERLAY>_POINTER_LUI_ADDR
;     li a1, new_string_label
```

## 5. After Generation

After creating the scaffold, explain the generated structure briefly, then suggest next steps:

1. **Identify target addresses** -- use a disassembler (Ghidra) or emulator debugger to find the addresses of strings, pointers, and code to modify.
2. **Add addresses** to `common/addresses.inc` following the naming convention.
3. **Define strings** in `common/strings.inc` -- one definition per string, referenced everywhere.
4. **Write the first modification** in the appropriate `<overlay>/feature.asm` file.
5. **Build** with `armips main.asm` and test in an emulator.
