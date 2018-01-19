# Restore disabled script op codes

Version 0.1, 2018-01-19 - DRAFT FOR DISCUSSION




## Introduction

This document describes proposed requirements for reactivating several script op codes.  In 2011 the discovery of
two serious bugs in `OP_LSHIFT` and `OP_RETURN` prompted the deactivation of these and 13 additional op codes.
The following list of disabled op codes is broken down by category.

##### Splice operations

|Word       |OpCode |Hex |Input         |Output  | Description                                                      |
|-----------|-------|----|--------------|--------|------------------------------------------------------------------|
|OP_CAT     |126    |0x7e|x1 x2         |out     |Concatenates two byte strings                                     |
|OP_SUBSTR  |127    |0x7f|in begin size |out     |Returns a section of a string                                     |
|OP_LEFT    |128    |0x80|in size       |out     |Keeps only characters left of the specified point in the string   |
|OP_RIGHT   |129    |0x81|in size       |out     |Keeps only characters right of the specified point in the string  |


##### Bitwise logic

|Word       |OpCode |Hex |Input         |Output  | Description                                                      |
|-----------|-------|----|--------------|--------|------------------------------------------------------------------|
|OP_INVERT  |131    |0x83|in            |out     |Flips all bits of the input                                       |
|OP_AND     |132    |0x84|x1 x2         |out     |Boolean *AND* beteween each bit of the inputs                     |
|OP_OR      |133    |0x85|x1 x2         |out     |Boolean *OR* beteween each bit of the inputs                      |
|OP_XOR     |134    |0x86|x1 x2         |out     |Boolean *EXCLUSIVE OR* beteween each bit of the inputs            |


##### Arithmetic

|Word       |OpCode |Hex |Input         |Output  | Description                                                      |
|-----------|-------|----|--------------|--------|------------------------------------------------------------------|
|OP_2MUL    |141    |0x8d|in            |out     |The input is multiplied by 2                                      |
|OP_2DIV    |142    |0x8e|in            |out     |The input is divided by 2                                         |
|OP_MUL     |149    |0x95|a b           |out     |*a* is multiplied by *b*                                          |
|OP_DIV     |150    |0x96|a b           |out     |*a* is divided by *b*                                             |
|OP_MOD     |151    |0x97|a b           |out     |return the remainder after *a* is divided by *b*                  |
|OP_LSHIFT  |152    |0x98|a b           |out     |shifts *a* left by *b* bits, preserving sign                      |
|OP_RSHIFT  |152    |0x99|a b           |out     |shifts *a* right by *b* bits, preserving sign                     |


It is proposed to reintroduce these op codes (or equivalent functinality) in a staged process.  The first stage being
t enable a limited subset in the May hard fork. 

Splice operations: `OP_CAT`, `OP_SPLIT`**

Bitwise logic: `OP_AND`, `OP_OR`, `OP_XOR`

Arithmetic: `OP_MOD`

New: optionally*** , either of: 
* `<n> OP_ZEROS` - returns a byte vector of length *n* containing all zeros
* `<string> <n> OP_REPEAT` - returns a byte vector of <string> repeated *n* times
* `<string> <n> OP_PADLEFT` - pads the left of <string> until it is of length *n*

** A new operation, `OP_SPLIT`, is proposed as a replacement for `OP_SUBSTR`, `OP_LEFT`and `OP_RIGHT`. All three operations can be
simulated with varying combinations of `OP_SPLIT`, `OP_ROT` and `OP_DROP`.

*** futher discussion of the purpose of these new operation under bitwise operations.

## Risks and philosophical approach

In general the approach taken is a minimalist one in order limit edge cases as much as possible.  Where it is possible
for a single more limited op code to be combined with other op codes to achieve a functionality that is preferred over
a more complex op code.  Input conditions that create ambiguous or undefined behaviour should fail fast.

Each op code should be examined for the following risk conditions and mitigating behaviour defined explcitly:
* Operand byte length mismatch.  Where it would be normally expected that two operands would be of matching byte lengths
the resultant behaviour should be defined.
* TODO signed integer issues
* TODO stack size issues - limit operand and output size
* TODO resource (CPU) utilization - exponential cycle cost attacks
* TODO overflow issue, e.g. OP_MUL overflowing int32 or int64 or uint256
* TODO endian issues ???
* TODO empty byte vector operands

## Definitions

## Specification

## Test plan

## References

<a name="op_codes">[1]</a> https://en.bitcoin.it/wiki/Script#Opcodes