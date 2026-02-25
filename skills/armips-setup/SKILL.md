---
description: "armips assembler setup and CLI: installation, command-line options, symbol file formats (sym/sym2), temp output, equates, working directory. Use when setting up armips, running assembler commands, or configuring build scripts."
---

# armips Setup

## Installation

**From source (CMake):**
```bash
git clone --recursive https://github.com/Kingcom/armips.git
cd armips
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

Requires a C++17-compliant compiler (GCC, Clang, or MSVC 2015+).

**Prebuilt binaries:** Available on the [GitHub releases page](https://github.com/Kingcom/armips/releases) for Windows. macOS/Linux must build from source.

## Command-Line Usage

```
armips <FILE> [options]
```

| Option | Description |
|--------|-------------|
| `<FILE>` | Main assembly source file (required) |
| `-temp <file>` | Output temporary assembly data (addresses, origins) |
| `-sym <file>` | Output symbol table in sym format (NO$PSX / NO$GBA compatible) |
| `-sym2 <file>` | Output symbol table in sym2 format (PCSX2 / PPSSPP compatible) |
| `-equ <name> <value>` | Define text equate -- equivalent to `name equ value` in source |
| `-strequ <name> <value>` | Define string equate -- equivalent to `name equ "value"` in source |
| `-definelabel <name> <value>` | Define label -- equivalent to `.definelabel name, value` in source |
| `-root <directory>` | Set working directory for relative paths |
| `-erroronwarning` | Promote all warnings to errors |
| `-stat` | Display memory usage statistics per area |

**Examples:**
```bash
armips patch.asm
armips patch.asm -sym symbols.sym -temp output.txt
armips game.asm -equ VERSION us -erroronwarning -stat
armips overlay.asm -definelabel BASE_ADDR 0x80010000
```

## Symbol File Formats

**sym format** (`-sym`) -- NO$PSX / NO$GBA compatible:
- One symbol per line: `address label`
- Supported by: NO$PSX, NO$GBA, BizHawk, Mesen

**sym2 format** (`-sym2`) -- Extended format:
- Supports 64-bit addresses and additional metadata
- Supported by: PCSX2, PPSSPP

## Control Directives

These directives control symbol and warning behavior from within source files:

| Directive | Description |
|-----------|-------------|
| `.sym on/off` | Enable/disable symbol file writing for a section |
| `.erroronwarning on/off` | Toggle warning-to-error promotion |
| `.relativeinclude on/off` | Include paths relative to current file (default: off) |
| `.nocash on/off` | NO$GBA debug message semantics |
