# WASM Analysis

This document provides notes on the WASM for a simple function named 'add' that
returns the sum of two arguments, all values are of type float. 

Tools used to generate .wasm and .wat are from the [WABT] project. The
[repository](https://github.com/cullenr/wasm-analysis) for this document
contains compiled Linux binaries for convenience.

The WASM specification can be found here
https://webassembly.github.io/spec/core/bikeshed/index.html

## WAT

```
(module
 (func $add (param f32) (param f32) (result f32)
   get_local 0
   get_local 1
   f32.add)
 (export "add" (func 0)))
```
## WASM

wat2wasm output:

```
00000000: 0061 736d 0100 0000 0107 0160 027d 7d01  .asm.......`.}}.
00000010: 7d03 0201 0007 0701 0361 6464 0000 0a09  }........add....
00000020: 0107 0020 0020 0192 0b0a                 ... . ....
```
## WAT from WASM

Note the type section which is not present in the original WAT. wat2wasm output:

```
(module
  (type (;0;) (func (param f32 f32) (result f32)))
  (func (;0;) (type 0) (param f32 f32) (result f32)
    local.get 0
    local.get 1
    f32.add)
  (export "add" (func 0)))
```

## Breakdown

The wasm file starts of with a module definition.  According to the spec a
module is formatted like so `magic, version, sections`.

[link](https://webassembly.github.io/spec/core/binary/modules.html#binary-module)

## Preamble

`0061 736d` : magic number
`0100 0000` : wasm version

## [Sections](https://webassembly.github.io/spec/core/binary/modules.html#sections)

The sections portion of the module starts with a type-section (typesec) and is
followed by a function-section

`0107 0160 027d 7d01 7d` : [typesec]

`01` : section id for type-section
`07` : length of section in bytes (note the 7 bytes below)
`01` : the length of the functype vector that the type-section expects
`60` : functype opcodea (two vectors follow for params and results respectively)
`02` : the length of the valtype vector of params
`7d` : the opcode for 32 bit float (1st param)
`7d` : the opcode for 32 bit float (2nd param)
`01` : the length of the valtype vector of results
`7d` : the opcode for 32 bit float (return value)

`03 0201 00` [funcsec] 

`03` : section id for function-section
`02` : length of this section in bytes
`01` : length of the functype vector that the function-section expects
`00` : the index of the code body in the code-section below.

`07 0701 0361 6464 0000` [exportsec]

`07` : section id for export-section
`07` : length of this section in bytes
`01` : length of the export vector that the export-section expects
`03` : length of the byte arr for this exports _name_ "add"
`61` : byte 'a'
`64` : byte 'd'
`64` : byte 'd'
`00` : declare this export is a function
`00` : the id of the function to export, in this case 00

`0a09 0107 0020 0020 0192 0b` [codesec]

`0a` : section id for code-section
`10` : length of this section in bytes
`01` : length of the [code] vector that the code-section expects 
`07` : length of the [func] in bytes
`00` : length of the locals array - 0 so no locals are present. 
`20` : start of the expression section - 0x20 is the local.get instruction
`00` : id for locla.get for the first param
`20` : second call to local.get to get param 2
`01` : id for local.get for the second param
`92` : the opcode for f32.add
`0b` : end of the expression dentoed by the end opcode 0x0b

`0a` : this is marked as reserved but undocumented in the [spec](https://webassembly.github.io/spec/core/bikeshed/index.html#a7-index-of-instructions)

[WABT]: https://github.com/WebAssembly/wabt

[typesec]:https://webassembly.github.io/spec/core/binary/modules.html#type-section
[funcsec]:https://webassembly.github.io/spec/core/binary/modules.html#function-section 
[exportsec]:https://webassembly.github.io/spec/core/binary/modules.html#export-section

[index]:https://webassembly.github.io/spec/core/binary/modules.html#indices
[functype]:https://webassembly.github.io/spec/core/binary/types.html#binary-functype
[codesec]:https://webassembly.github.io/spec/core/binary/modules.html#code-section 
[code]:https://webassembly.github.io/spec/core/binary/modules.html#code-section 
[func]:https://webassembly.github.io/spec/core/binary/modules.html#code-section 

**TODO:** add a new function to the module so we can see how this affects the
original and also make the new function contain a local - a ++ function maybe.
