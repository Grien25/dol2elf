# DOL 2 ELF

Create an ELF executable file from a [DOL](http://wiibrew.org/wiki/DOL) file.

## Usage

~~~sh
dol2elf foo.dol foo.elf
~~~

## What this tool does

This tool takes a GameCube/Wii DOL binary and produces a 32-bit PowerPC ELF file that can be read by standard tools (such as `objdump`, `gdb`, `radare2`) that do not natively support the DOL format. The generated ELF is not meant to be executed, but is useful for analysis and inspection.

### Conversion Details

- **Segments and Sections:**
  - Each non-empty DOL text segment becomes both an ELF program segment (PT_LOAD) and a section (named `.text.N`).
  - Each non-empty DOL data segment becomes both an ELF program segment (PT_LOAD) and a section (named `.data.N`).
  - If the DOL has a BSS segment, it is mapped to both an ELF PT_LOAD segment and a `.bss` section (with SHT_NOBITS type).
- **Section Names:**
  - Section names are generated as `.text.0`, `.text.1`, ..., `.data.0`, `.data.1`, ..., and `.bss` as appropriate.
  - A `.shstrtab` section is included for section names.
  - A `.dolhdr` section contains a copy of the DOL header.
- **String Table:**
  - The ELF includes a `.shstrtab` section containing all section names.
- **DOL Header:**
  - The DOL file header is copied into a `.dolhdr` section for reference.
- **File Layout:**
  - The ELF header, program headers, and section headers are written first.
  - The section name string table follows.
  - The entire DOL file is then copied verbatim at the end of the ELF file (after the ELF headers and string table).

### Limitations

- The resulting ELF is a static, non-relocatable, non-executable file intended for analysis only.
- No symbol or relocation information is generated.
- The ELF is always big-endian, 32-bit, and marked as PowerPC architecture.

## Example

~~~sh
./dol2elf foo.dol foo.elf
objdump -h foo.elf
~~~

This will show ELF sections corresponding to the DOL segments, plus `.shstrtab` and `.dolhdr`.
