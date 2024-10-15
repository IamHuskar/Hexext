HEXEXT

Supports IDA 7.0 only currently. (32 and 64)

This glorious plugin uses a variety of trickz to manipulate the internal IR of the Hexrays decompiler with the aim
of improving code generation.

Compiling this requires a compiler with at least C++17 support

right now documentation is very sparse I will be writing some up eventually

https://forum.reverse4you.org/t/hexext-source-release-the-return-of-the-hex/10675

It’s still super messy, I’ve just kinda given up on making it less messy for now. Documentation isn’t that great either.

There’s a lot of new stuff in new binaries, but not all of it is well tested and some things may be unstable.

This release removes the if-inversion from the early preoptimization pass and moves it to instead operate using the ctree API. Auto-if inversion in scenarios where the if immediately falls through to a return at the end of the block. It looks much better in my opinion.

This release also adds new rules for transforming bitwise operations on floats and doubles.

A series of massive changes were made to the way the plugin operates internally. A way was found to map out all of the micro registers automatically (see mvm/mvm_base.cpp for that), and I discovered how to tag intrinsics with unique roles (exrole/).

A quick map for anyone interested in looking through the code:

combine_rules/ mostly consists of optimization rules for the microgen, which recognize patterns and transform the code to be noicer.

internal/micro_filter_api.hpp has the implementation of hexexts initial codegen hook. The plugin hijacks control of the microgen through a microcode_filter_t which runs all of the asm filters on the current instruction before testing for another filters match.

mvm/ contains everything related to the microcodes registers. In some cases, the locations of temporary registers need to be deduced from the locations of registers that are dynamically calculated. in other cases, the temporary register locations are hardcoded.

microgen_additions/ has everything related to the initial microgen stage, where the decompiler translates instructions from their architecture-dependent format to its architecture independent microcode. If you want to implement codegen for new instructions, look at bmi1_codegen.cpp or fixup_bittest_codegen.cpp. It also features several “asm rewriters”, which are architecture-specific for the most part and perform small but helpful transformations on the current input insn_t in codegen_t or on instructions decoded (by other asm rewriters or microgen filters) using microgen_decode_prev_insn or microgen_decode_insn.

The root file micro_on_70.hpp is a massive, disgusting beast like most of my headers. It contains helper functions, structure definitions, etc. It also contains the codegen_ex_t class, which exposes the (undocumented on 7.0) emit function for emitting microinstructions.

hexutils/ contains more ungodly monstrosities, a large number of helper functions, dataflow analysis functions, traverse_micro, and code for the computation of the potentially non-zero bits of an operand.

micro_hooks.cpp has the implementation of the main hexrays callback and the dispatching of events across the plugin, etc. It also has an ungodly function that fixes up various mistakes I made during the initial release but which needs to die asap because it’ll probably fuck things up in the future.

exrole/ is responsible for mapping architecture-dependent intrinsics to independent “extended roles”. These extended roles can be extracted at any point after the preoptimization phase to implement optimization rules operating on intrinsics (see combine_rules/combine_core.cpp - detect_xdu_in_xor128_t::run_combine/simd_ld_shrtrim_t::run_combine for an example of this).

There’s a massive amount of unused garbage bundled in as well, but there’s also some pretty solid, neat stuff in here. Hope everyone enjoys.
