---
description: "Recommended ASM project structure for armips: directory layout, naming conventions, patching patterns (strpad, relocation, shared macros, computed values), and architectural principles. Use when helping users organize armips assembly modifications (patches, mods, hacks, translations, romhacks)."
---

# armips Project Structure

> To generate this structure automatically, use the **armips-project-scaffold** skill.

## 1. Recommended Directory Tree

```
project/
├── main.asm                    # Entry point - orchestrates all modifications
├── common/
│   ├── macros.inc              # Core macros (strpad)
│   ├── shared_patches.inc      # Shared parametric macros (optional)
│   ├── addresses.inc           # All target binary addresses + filename constants
│   ├── values.inc              # Computed/derived values
│   ├── strings.inc             # All modified/translated strings
│   └── *.tbl                   # Character encoding tables (one or more)
├── <overlay_1>/                # One directory per target binary/overlay
│   ├── feature_a.asm
│   ├── feature_b.asm
│   └── *.inc / *.tbl           # Overlay-specific data and tables (optional)
├── <overlay_2>/
│   └── ...
└── <shared_binary>/            # Binary with free space for cross-binary relocation
    └── free_space.asm
```

## 2. Role of Each File

**main.asm** -- Entry point for the entire project. Declares the target architecture, loads `.tbl` files via `.loadtable`, includes common files in order (`macros.inc`, `strings.inc`, `addresses.inc`, `values.inc`, then `shared_patches.inc` if present), then opens and closes each binary with `.open`/`.close`, including per-overlay patch files inside each block. Filenames and base addresses in `.open` are referenced as `equ` constants defined in `addresses.inc`, not as string literals.

**addresses.inc** -- Single source of truth for all memory addresses and binary filename constants in the project. Naming convention: `OVERLAY_FUNCTION_PURPOSE_ADDR` for addresses, `OVERLAY_FILENAME` for filenames. Special suffixes indicate the kind of address: `_LUI_ADDR` for pointer load sites (lui/addiu pairs), `_WIDTH_ADDR` for UI width values, `_COLUMN_ADDR` for column/position offsets.

**strings.inc** -- All strings defined as `equ` constants. Naming convention: `CATEGORY_NAME_STR`. A string used in multiple overlays is defined once here and referenced everywhere, avoiding duplication.

**values.inc** -- Values automatically derived from strings via `strlen()` and arithmetic. Example: a highlight width computed as `strlen(STR) * CHAR_WIDTH_PX`. No manual magic numbers -- all values trace back to string definitions.

**macros.inc** -- Core macros like `strpad` (write string + zero-fill + overflow guard via `.area`). Keep this file small -- only fundamental macros that other files depend on.

**shared_patches.inc** -- (Optional) Shared parametric macros for repetitive modifications that apply the same logic across multiple overlays with different addresses. Separated from `macros.inc` because these macros reference project-specific strings and addresses, while `macros.inc` contains only general-purpose utilities.

**Overlay directories** -- One `.asm` file per logical feature, not one monolith per binary. Each file contains the patches for a single feature or subsystem within that overlay.

**.tbl files** -- One or more character encoding tables. Multiple tables are supported for games with different encoding contexts (e.g., menu text vs. dialogue vs. battle messages). The main table is typically loaded via `.loadtable` in `main.asm`. Additional tables used only by a specific overlay can live inside the overlay directory and be loaded from there.

## 3. Fundamental Patterns

### Pattern 1: strpad macro

Define and use the `strpad` macro for safe string writing with overflow detection. The second argument to `.area` tells armips to zero-fill all remaining bytes automatically. The `.area` directive triggers a build error if the encoded string exceeds the slot.

```
.macro strpad, str, slot_size
.area slot_size, 0x00
    .string str
.endarea
.endmacro
```

### Pattern 2: In-place string replacement

Direct replacement at the original address when the translated string fits within the original slot.

```
.org OVERLAY_STRING_ADDR
    strpad TRANSLATED_STR, 12
```

### Pattern 3: EOF relocation

For strings too long for the original slot, append past end of file and redirect the pointer to the new location.

```
.org OVERLAY_EOF_ADDR
new_string_label:
    .string LONG_STRING_STR

.org OVERLAY_POINTER_LUI_ADDR
    li a1, new_string_label
```

### Pattern 4: Cross-binary relocation

Strings that don't fit in any overlay slot, placed in free space of the main executable and referenced from multiple overlays.

```
; In main executable section
.org MAIN_EXE_FREE_SPACE_ADDR
shared_str_label:
    .string SHARED_LONG_STR

; In overlay section - redirect pointer
.org OVERLAY_POINTER_LUI_ADDR
    li a1, shared_str_label
```

### Pattern 5: Freed slot reuse

Slots freed by relocated strings can be reused for other strings. Track with comments documenting what was relocated and the available size.

```
; Slot at 0x1000 (16B): original string relocated → reuse for new string
.org OVERLAY_FREED_SLOT_ADDR
reused_label:
    strpad NEW_SHORT_STR, 16
```

### Pattern 6: Shared parametric macros

For repetitive modifications across overlays with the same logic but different addresses. Define once, call once per overlay.

```
.macro patch_labels, addr_a, addr_b, addr_c
    .org addr_a
        strpad LABEL_A_STR, 8
    .org addr_b
        strpad LABEL_B_STR, 8
    .org addr_c
        strpad LABEL_C_STR, 12
.endmacro

; Called once per overlay with overlay-specific addresses
patch_labels OVERLAY1_A_ADDR, OVERLAY1_B_ADDR, OVERLAY1_C_ADDR
```

### Pattern 7: Table rewriting

Rewrite sequential string tables with prefix byte, variable-length entries, and zero-fill to pad the remainder of the table area.

```
.org OVERLAY_TABLE_ADDR
    .db 0x03 :: .string ENTRY_1_STR
    .db 0x03 :: .string ENTRY_2_STR
    .db 0x03 :: .string ENTRY_3_STR
    .fill (OVERLAY_TABLE_ADDR + TABLE_SIZE - .), 0
```

### Pattern 8: Code patching

Modify load instructions for pointer redirects and UI positioning. The `li` pseudo-instruction expands to the appropriate lui/addiu pair.

```
; Pointer redirect (lui/addiu pair)
.org OVERLAY_POINTER_LUI_ADDR
    li a1, new_string_label

; Immediate value patch for UI positioning
.org OVERLAY_COLUMN_OFFSET_ADDR
    li a2, NEW_COLUMN_OFFSET
```

## 4. Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Addresses | `OVERLAY_FUNCTION_PURPOSE_ADDR` | `TITLE_EXE_OPTIONS_TITLE_ADDR` |
| Strings | `CATEGORY_NAME_STR` | `OPTIONS_TITLE_STR` |
| Local labels | `overlay_str_purpose` | `field_str_vib_off` |
| Static labels | `@purpose` | `@compute_width` |
| Nested labels | `@@sub_purpose` | `@@is_spacing` |
| Computed values | `DESCRIPTION_NAME` | `HIGHLIGHT_WIDTH_CAM_L` |
| Macros | `verb_object` | `patch_ui_attr_labels` |

## 5. Architectural Principles

- **Single source of truth** -- strings, addresses, and values are never duplicated. Define once in `common/`, reference everywhere.
- **Build-time overflow detection** -- use `.area` (via `strpad` or directly) so the assembler errors if a patch exceeds its allocated space.
- **Derived values via `strlen()` and arithmetic** -- no manual magic numbers. Widths, offsets, and sizes are computed from string definitions.
- **One `.open`/`.close` per binary in main.asm** -- total isolation between binaries. Each binary block includes only its own overlay patches.
- **Progressive relocation strategy** -- try in-place first, then freed slot reuse, then EOF relocation, then cross-binary relocation to the main executable.
