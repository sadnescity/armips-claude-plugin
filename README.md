# armips Claude Code Plugin

Reference plugin for the [armips assembler](https://github.com/Kingcom/armips) — a cross-platform assembler for MIPS and ARM architectures, widely used in ROM hacking.

## Skills

| Skill | Description |
|-------|-------------|
| armips-setup | Installation, CLI usage, symbol file formats |
| armips-syntax | Directives, expressions, labels, macros, tables |
| armips-mips | MIPS instruction sets: R3000, R4000, Allegrex, RSP, EmotionEngine |
| armips-arm | ARM instruction sets: ARM7/ARM9 + THUMB mode |
| armips-workflows | Practical ROM hacking workflows: patching, text hacking, symbol integration |
| armips-project-structure | Recommended ASM project layout, naming conventions, patching patterns |
| armips-project-scaffold | Interactive project scaffold generator for new armips projects |

## Installation

```
/plugin marketplace add brisma/claude-plugins
/plugin install armips@brisma-plugins
```

## Supported Architectures

- **MIPS:** R3000 (PSX), R4000 (N64), Allegrex (PSP), RSP (N64), EmotionEngine (PS2)
- **ARM:** ARM7 (GBA), ARM9 (NDS), ARM11 (3DS), THUMB mode
- **SuperH:** Saturn, 32X

## License

MIT — same as armips.
