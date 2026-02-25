---
description: "armips assembler directives and syntax: file operations (.open/.create/.close), addressing (.org/.area/.region), data (.byte/.word/.float), text (.string/.loadtable/.sjis), labels (global/static/local), macros, equates, conditional assembly (.if/.ifdef), expressions (40+ built-in functions), includes (.include/.incbin). Use when writing armips assembly, understanding directives, or debugging assembler errors."
---

# armips Syntax Reference

## Comments and Separators

```
; single-line comment
// single-line comment
/* block comment */
```

- `::` separates multiple statements on one line
- `\` at end of line continues to next line

## Architecture Selection

| Directive | Architecture | CPU |
|-----------|-------------|-----|
| `.psx` | PlayStation 1 | MIPS R3000 |
| `.ps2` | PlayStation 2 | EmotionEngine |
| `.psp` | PlayStation Portable | Allegrex |
| `.n64` | Nintendo 64 | MIPS R4000 |
| `.rsp` | N64 RSP | Reality Signal Processor |
| `.gba` | Game Boy Advance | ARM7 |
| `.nds` | Nintendo DS | ARM9 |
| `.3ds` | Nintendo 3DS | ARM11 |
| `.saturn` | Sega Saturn | SuperH |
| `.32x` | Sega 32X | SuperH |
| `.arm.big` | ARM big-endian | generic ARM |
| `.arm.little` | ARM little-endian | generic ARM |

## File Operations

| Directive | Description |
|-----------|-------------|
| `.open FileName,HeaderSize` | Open existing file for modification |
| `.open OldFile,NewFile,HeaderSize` | Copy OldFile to NewFile, open copy |
| `.create FileName,HeaderSize` | Create new output file |
| `.close` | Close current output file |
| `.include FileName[,encoding]` | Include another assembly source file |
| `.incbin FileName[,start[,size]]` | Include raw binary data from file |
| `.loadelf name[,outputname]` | Open ELF file for output (PSP only) |

`HeaderSize` is the offset between memory addresses and file positions. For example, a PSX executable loaded at `0x80010000` with a 2048-byte header uses `HeaderSize = 0x80010000 - 0x800 = 0x8000F800` (or equivalently, set `.headersize` after `.open`).

**Aliases:** `.openfile`, `.createfile`, `.closefile`, `.import` (for `.incbin`)

## Position Control

| Directive | Description |
|-----------|-------------|
| `.org address` | Set current memory address |
| `.orga address` | Set current absolute file address |
| `.headersize value` | Set memory-to-file offset |
| `.align [num[,fill]]` | Align memory address to multiple of `num` (default 4) |
| `.aligna [num[,fill]]` | Align absolute file address to multiple of `num` |
| `.skip length` | Skip bytes without writing |

**Note:** `.org` and `org` (without dot) are both valid. Same for `.orga`/`orga`.

## Data Directives

**8-bit (byte):**

| Directive | Aliases |
|-----------|---------|
| `.byte` | `.db`, `.dcb`, `.d8`, `.ascii`, `db`, `dcb` |
| `.asciiz` | Like `.ascii` but appends null terminator |

**16-bit (halfword):**

| Directive | Aliases |
|-----------|---------|
| `.halfword` | `.hword`, `.dh`, `.dcw`, `.d16`, `dh`, `dcw` |

**32-bit (word):**

| Directive | Aliases |
|-----------|---------|
| `.word` | `.dw`, `.dcd`, `.d32`, `dw`, `dcd` |

**64-bit (doubleword):**

| Directive | Aliases |
|-----------|---------|
| `.doubleword` | `.dword`, `.dd`, `.dcq`, `.d64`, `dd`, `dcq` |

**Floating-point:**

| Directive | Size |
|-----------|------|
| `.float` | 32-bit IEEE 754 |
| `.double` | 64-bit IEEE 754 |

**Fill:**

| Directive | Description |
|-----------|-------------|
| `.fill length[,value]` | Fill `length` bytes with `value` (default 0) |

All data directives accept comma-separated lists: `.word 1,2,3,4`

## Text and Encoding

| Directive | Description |
|-----------|-------------|
| `.loadtable file[,encoding]` | Load custom character encoding table |
| `.string "text"[,...]` | Output text using loaded table encoding, with terminator |
| `.stringn "text"[,...]` | Same as `.string` but without terminator |
| `.sjis "text"[,...]` | Output Shift-JIS encoded text with null terminator |
| `.sjisn "text"[,...]` | Shift-JIS without terminator |

**Aliases:** `.str` (`.string`), `.strn` (`.stringn`), `.table` (`.loadtable`)

**Table file format** -- one mapping per line:
```
02=a
03=b
1D=the
2F=you
/FF
```

The `/XX` line sets the string terminator byte (default `00`).

## Labels

**Three scoping levels:**

| Prefix | Scope | Example |
|--------|-------|---------|
| (none) | Global -- valid everywhere | `MyLabel:` |
| `@` | Static -- valid in defining file only | `@FileLocal:` |
| `@@` | Local -- valid between surrounding global/static labels | `@@loop:` |

- `.` (dot) references the current memory address
- Names: A-Z, 0-9, underscores. Case-insensitive. Cannot start with digit.

**Label definition directives:**

| Directive | Description |
|-----------|-------------|
| `.definelabel name,value` | Define global label with numeric value |
| `.definedatalabel name,value` | Define data label (ARM: no Thumb bit) |
| `.definearmlabel name,value` | Define ARM mode label |
| `.definethumblabel name,value` | Define THUMB mode label |

**Function scoping:**
```
.func FunctionName
  @@local_label:         ; scoped to this function only
  nop
.endfunc
```
Aliases: `.function`/`.endfunction`

## Equates

Text replacement before parsing -- NOT numeric labels:
```
MyConstant  equ  0x100
@FileConst  equ  gp
@@LocalEq   equ  4(sp)
```

Must have whitespace before and after `equ`.

## Macros

```
.macro PushRegisters,reg1,reg2
  addiu  sp,sp,-8
  sw     \reg1,0(sp)
  sw     \reg2,4(sp)
.endmacro
```

- Parameters referenced with `\param` in macro body
- Up to 128 nesting levels
- Local labels (`@@`) auto-renamed for uniqueness per expansion
- Invocation: `PushRegisters a0,a1`

## User-Defined Expression Functions

```
.expfunc double(x), x*2
.expfunc fib(n), n <= 2 ? 1 : fib(n-1)+fib(n-2)
```

Supports recursion. Function name must be unique.

## Conditional Assembly

| Directive | Description |
|-----------|-------------|
| `.if expression` | Assemble if expression is nonzero |
| `.ifdef symbol` | Assemble if symbol is defined |
| `.ifndef symbol` | Assemble if symbol is NOT defined |
| `.else` | Alternative block |
| `.elseif expression` | Alternative with condition |
| `.elseifdef symbol` | Alternative if symbol defined |
| `.elseifndef symbol` | Alternative if symbol not defined |
| `.endif` | End conditional block |

## Areas, Regions, and Autoregions

**Area** -- bounded allocation with overflow checking:
```
.area 0x100              ; max 256 bytes
  ; code here -- error if exceeds 0x100 bytes
.endarea

.area 0x100,0x00         ; fill remaining with 0x00
  nop
.endarea
```

**Region/Autoregion** -- automatic placement:
```
.region 0x200            ; define 512 bytes of available space
.endregion

.autoregion              ; placed in first region with enough space
  ; code here
.endautoregion
```

## Expressions

**Number formats:**

| Format | Example |
|--------|---------|
| Decimal | `1234` |
| Hexadecimal | `0xFF`, `0FFh` |
| Octal | `0o12`, `12o` |
| Binary | `0b1010`, `1010b` |
| Character | `'a'` (yields 97) |
| Float | `3.14`, `314e-2` |

**Operators:** All C operators supported with standard precedence: `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `~`, `<<`, `>>`, `&&`, `||`, `!`, `==`, `!=`, `<`, `>`, `<=`, `>=`, `? :`. String `+` concatenates.

## Built-in Functions

### General

| Function | Return | Description |
|----------|--------|-------------|
| `version()` | int | armips version as integer |
| `endianness()` | string | `"big"` or `"little"` |
| `outputname()` | string | Current output filename |
| `org()` | int | Current memory address |
| `orga()` | int | Current absolute file address |
| `headersize()` | int | Current header size |
| `defined(symbol)` | int | 1 if symbol defined, 0 otherwise |
| `fileexists(file)` | int | 1 if file exists |
| `filesize(file)` | int | File size in bytes |

### Math and Conversion

| Function | Description |
|----------|-------------|
| `abs(val)` | Absolute value |
| `min(a,b,...)` | Minimum of values |
| `max(a,b,...)` | Maximum of values |
| `round(val)` | Round float to nearest integer |
| `int(val)` | Truncate float to integer |
| `float(val)` | Cast integer to float |
| `frac(val)` | Fractional part of float |
| `tostring(val)` | Convert to string |
| `tohex(val[,digits])` | Convert to hex string (default 8 digits) |

### String

| Function | Description |
|----------|-------------|
| `strlen(str)` | String length |
| `substr(str,start,count)` | Extract substring |
| `find(str,substr[,start])` | First index of substring, -1 if not found |
| `rfind(str,substr[,start])` | Last index of substring, -1 if not found |
| `regex_match(str,regex)` | 1 if regex matches entire string |
| `regex_search(str,regex)` | 1 if regex matches any part |
| `regex_extract(str,regex[,idx])` | Extract matched substring |

### File Reading

| Function | Description |
|----------|-------------|
| `readbyte(file[,pos])` | Read unsigned 8-bit from file |
| `readu8(file[,pos])` | Read unsigned 8-bit |
| `readu16(file[,pos])` | Read unsigned 16-bit |
| `readu32(file[,pos])` | Read unsigned 32-bit |
| `readu64(file[,pos])` | Read unsigned 64-bit |
| `reads8(file[,pos])` | Read signed 8-bit |
| `reads16(file[,pos])` | Read signed 16-bit |
| `reads32(file[,pos])` | Read signed 32-bit |
| `reads64(file[,pos])` | Read signed 64-bit |
| `readascii(file[,start[,len]])` | Read ASCII string |

### Architecture-Specific

| Function | Arch | Description |
|----------|------|-------------|
| `hi(val)` | MIPS | High half of 32-bit value, adjusted for sign extension |
| `lo(val)` | MIPS | Sign-extended low half of 32-bit value |
| `isarm()` | ARM | 1 if currently in ARM mode |
| `isthumb()` | ARM | 1 if currently in THUMB mode |

## Messages

| Directive | Description |
|-----------|-------------|
| `.warning "msg"` | Issue warning |
| `.error "msg"` | Issue error and halt |
| `.notice "msg"` | Print informational message |
