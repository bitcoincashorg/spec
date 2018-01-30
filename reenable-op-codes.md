# Restore disabled script op codes

Version 0.1, 2018-01-19 - DRAFT FOR DISCUSSION

##Draft discussion notes

For the purposes of discussion of this draft additional notes are contained throughout with the heading *DRAFT DISCCUSION*.  These are
intended to be removed from the finalalized version of this document.

Optional rules are denoted by *RULE OPTION* where it is intended one the presented options will be adopted following consensus within
the workgroup.


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


It is proposed to reintroduce these op codes (or equivalent functionality) in a staged process.  The first stage being
to enable a limited subset in the May hard fork. 

Splice operations: `OP_CAT`, `OP_SPLIT`**

Bitwise logic: `OP_AND`, `OP_OR`, `OP_XOR`

Arithmetic: `OP_MOD`

New: optionally*** , either of: 
* `<n> OP_ZEROS` - returns a byte vector of length `n` containing all zeros
* `<string> <n> OP_REPEAT` - returns a byte vector of <string> repeated `n` times
* `<string> <n> OP_PADLEFT` - pads the left of <string> until it is of length `n`
* `<string> <string> OP_MATCHLEN` - pads the left of the shorter string to match the length of the longer

** A new operation, `OP_SPLIT`, is proposed as a replacement for `OP_SUBSTR`, `OP_LEFT`and `OP_RIGHT`. All three operations can be
simulated with varying combinations of `OP_SPLIT`, `OP_SWAP` and `OP_DROP`.

*** futher discussion of the purpose of these new operation under bitwise operations.

## Risks and philosophical approach

In general the approach taken is a minimalist one in order limit edge cases as much as possible.  Where it is possible
for a single more primitive op code to be combined with other op codes to achieve a functionality that is preferred over
a more complex op code.  Input conditions that create ambiguous or undefined behaviour should fail fast.  

Each op code should be examined for the following risk conditions and mitigating behaviour defined explcitly:
* Operand byte length mismatch.  Where it would be normally expected that two operands would be of matching byte lengths
the resultant behaviour should be defined.
* Signed integer.  Whether signed integers are permitted operands and whether any special handling is required.
* Stack size impact.  Both number of elements and total size of elements. 
* Overflows.  Defined behaviour in the instance that result of the operation exceeds MAX_SCRIPT_ELEMENT_SIZE
* Empty byte vector operands.  Whether empty byte vectors should be allowed as a representation of zero.

## Definitions

* *Stack memory use* - sum of the size of the elements on the stack - gives an indication of impact on memory use
* *Operand order* - in keeping with convention where multiple operands are specified the top most stack item is the 
last operand.  e.g. `x1 x2 OP_CAT` --> x2 is the top stack item and x1 is the next from the top
* *Rule options* - For the purposes of this draft and for further discussion some rules are presented with options.
These are denoted by the heading *RULE OPTION* followed by an ordered list.  Generally the options will be a choice
between a restrictive and a more liberal rule.

## Specification

Global failure conditions apply to all operations. These failure conditions must be checked by the implementation when 
it is possible that they will occur:
* for all e : elements on the stack, `0 <= len(e) <= MAX_SCRIPT_ELEMENT_SIZE`
* for each operator, the required number of operands are present on the stack when the operand is executed

These unit tests should be included for every operation:
1. executing the operation with an input element of length greater than `MAX_SCRIPT_ELEMENT_SIZE` will fail
2. executing the operation with an incorrect number of operands causes a failure


Operand consumption:

In all cases where not explicitly stated otherwise the operand stack elements are consumed by the operation and replaced with the output.

## Splice perations

### OP_CAT
Concatenates two operands.

    x1 x2 OP_CAT → out
    
The operator must fail if:
* `0 <= len(out) <= MAX_SCRIPT_ELEMENT_SIZE` - the operation cannot output elements that violate the constraint on the element size
    * Draft discussion: OP_CAT is the only op code (?) that can output a byte vector of greater length than it's inputs.  Previously there
    has been no other way introduce a data element  longer than MAX_SCRIPT_ELEMENT_SIZE to the stack.  Also note that
    that op code outputs are not constrained by MAX_SCRIPT_ELEMENT_SIZE. As such a series of OP_CAT OP_DUP OP_CAT OP_DUP etc... creates
     exponential growth in the output vector length.  Given that a side effect of enabling OP_CAT is to introduce a new mechanism for creating
     script stack elements it is consistent to apply the same size constraint that is effectively in place for other mechanisms.  Consequently
     any future decision to relax that constraint will consistently apply to OP_CAT outputs as well.
* note that the concatentation of a zero length operand is valid

Impact of successful execution:
* stack memory use is constant
* number of elements on stack is reduced by one

The limit on the length of the output prevents the memory exhaustion attack and results in the operation having less 
impact on stack size than existing OP_DUP operators.

Unit tests:
1. `maxlen_x y OP_CAT → failure` – concatenating any operand except an empty vector, including a single byte value (e.g. `OP_1`), onto a maximum sized array causes failure
3. `large_x large_y OP_CAT → failure` – concatenating two operands, where the total length is greater than `MAX_SCRIPT_ELEMENT_SIZE`, causes failure
4. `OP_0 OP_0 OP_CAT → OP_0` – concatenating two empty arrays results in an empty array
5. `x OP_0 OP_CAT → x` – concatenating an empty array onto any operand results in the operand, including when `len(x) = MAX_SCRIPT_ELEMENT_SIZE`
6. `OP_0 x OP_CAT → x` – concatenating any operand onto an empty array results in the operand, including when `len(x) = MAX_SCRIPT_ELEMENT_SIZE`
7. `x y OP_CAT → concat(x,y)` – concatenating two operands generates the correct result

### OP_SPLIT

####DRAFT DISCUSSION

It will be noted that the new operation, `OP_SPLIT`, is proposed as a replacement for `OP_SUBSTR`, `OP_LEFT`and `OP_RIGHT`. All three operations can be
simulated with varying combinations of `OP_SPLIT`, `OP_SWAP` and `OP_DROP`.  This is in keeping with the minimalist philosophy where a single
primitive can be used to simulate multiple more complex operations.

####END DRAFT DISCUSSION

Split the operand at the given position.  This operation is the exact inverse of OP_CAT

    x n OP_SPLIT -> x1 x2

Notes:
* `x` is split at position `n`, where `n` is the number of bytes from the beginning
* `x1` will be the first `n` bytes of `x` and `x2` will be the remaining bytes 
* if `n == 0`, then `x1` is the empty array and `x2 == x`
* *RULE OPTIONS*
    1. Liberal: if `n >= len(x)`, then `x1 == x` and `x2` is the empty array. OR
    2. Restrictive: if `n > len(x)`, then the operator must fail.
        * note: under this option if `n > len(x)` then `x1 == x` and `x2` is the empty array.
    * Discussion: Arguably allowing n > len(x) opens the possibility of a script continuing to run under unexpected
        conditions.  The restrictive option eliminates out-of-bounds errors.  Whilst potentially placing the burden
        on the script author to do an additional bounds check.
* `x n OP_SPLIT OP_CAT` -> `x` - for all `x` and for all `n >= 0`
    
The operator must fail if:
* `!isnum(n)` - `n` is not a number
* `n < 0` - `n` is negative
* RULE OPTION (i): `n > len(x)` - this rule must apply is the restrictive option is accepted.

Impact of successful execution:
* stack memory use is constant (slight reduction by `len(n)`)
* number of elements on stack is constant

Unit tests:
* `OP_0 n OP_SPLIT -> OP_0 OP_0`, for all positive numbers n - execution of OP_SPLIT on empty array results in two empty arrays
* `x 0 OP_SPLIT -> OP_0 x`
* `x len(x) OP_SPLIT -> x OP_0`
* *RULE OPTION (i)*: `x len(x)+1 OP_SPLIT -> FAIL`

## Bitwise logic

####DRAFT DISCUSSION

For all bitwise operations that take two operands the case must be considered where `len(x1) != len(x2)`.  There are two proposed approaches
to this:

*RULE OPTION*

    1. Restrictive: Enforce equal lengths i.e. where `len(x1) != len(x2)` the operator will fail
        * This option is a fail early approach that requires the script author to be aware of operand length.  In order to facilitate
        use cases where lengths differ it is necessary to provide an additional opcode to easily pad an operand to the required length.
        These are specifed under the section "Optional new operations"
    2. Liberal: Pad the shorter operand on the left with zero bytes such that both are of equal length.
        * This is the approach taken in the original Satoshi code.
        
####END DRAFT DISCUSSION


### OP_AND

Boolean *and* between each bit in the operands.

	x1 x2 OP_AND → out

Notes:
* where `len(x1) == 0 == len(x2)` the output will be an empty array.

The operator must fail if:
1. *RULE OPTION (1)*: `len(x1) != len(x2)` - the length, in bytes, of the two values is not equal

Impact of successful execution:
* stack memory use reduced by `len(x1)`
* number of elements on stack is reduced by one

Unit tests:

1. *RULE OPTION (1)*: `x1 x2 OP_AND -> failure`, where `len(x1) != len(x2)` - operation fails when length of operands not equal
2. `x1 x2 OP_AND -> x1 & x2` - check valid results

TODO: minimal encoding of numbers – would this cause numbers (byte arrays where len <= 4) to be automatically left truncated? Is it possible to AND the values 0x0005 and 0x0100?

### OP_OR

Boolean *or* between each bit in the operands.

	x1 x2 OP_OR → out
	
The operator must fail if:
1. *RULE OPTION (1)*:`len(x1) != len(x2)` - the length, in bytes, of the two values is not equal

Impact of successful execution:
* stack memory use reduced by `min(len(x1), len(x2))`
* number of elements on stack is reduced by one

Unit tests:
1. *RULE OPTION (1)*:`x1 x2 OP_OR -> failure`, where `len(x1) != len(x2)` - operation fails when length of operands not equal
2. `x1 x2 OP_OR -> x1 | x2` - check valid results

### OP_XOR
Boolean *xor* between each bit in the operands.

	x1 x2 OP_XOR → out
	
The operator must fail if:
1. *RULE OPTION (1)*: `len(x1) != len(x2)` - the length, in bytes, of the two operands is not equal

Impact of successful execution:
* stack memory use reduced by `len(x1)`
* number of elements on stack is reduced by one

Unit tests:
1. *RULE OPTION (1)*: `x1 x2 OP_XOR -> failure`, where `len(x1) != len(x2)` - operation fails when length of operands not equal
2. `x1 x2 OP_XOR -> x1 xor x2` - check valid results
    
## Arithmetic
    
### OP_MOD

Returns the remainder after dividing a by b.

	a b OP_MOD → out
	
The operator must fail if:
1. `!isnum(a) || !isnum(b)` - either operand is not a valid number
1. `a < 0` - `a` is a negative number
1. `b <= 0` - `b` is a negative number or equal to any type of zero

Impact of successful execution:
* stack memory use reduced (one element removed)
* number of elements on stack is reduced by one

Unit tests:
1. `a b OP_MOD -> failure` where `!isnum(a)` or `!isnum(b)` - both operands must be valid numbers
2. `a 0 OP_MOD -> failure` - division by positive zero (all sizes), negative zero (all sizes), `OP_0` 
3. `a b OP_MOD -> failure` where `a < 0`, `b < 0` - both operands must be positive
4. check valid results for operands of different lengths `1..4`

## Optional new operations

####DRAFT DISCUSSION

In order to facilitate the "operands must be equal length" rule for bitwise logic.  An additional operator is required to give script
authors a reasonable way of padding operands when required.  Four options are presented.  Only one is required to fullfill the stated purpose.

TODO: assign op code bytes to each operator.
        
####END DRAFT DISCUSSION


### OP_ZEROES
Produces byte vector of length `n` containing all zero bytes

	n OP_ZEROES → out
	
The operator must fail if:
1. `!isnum(n)` - `n` is not a number
2. `n < 0` - the length must be positive
2. `n > MAX_SCRIPT_ELEMENT_SIZE` - the length of the result would be too large

Notes:
* `n = 0` is valid, a zero length byte string is produced.  This is the functional equilent of OP_0

Impact of successful execution:
* stack memory use increased by `n - len(n)`, maximum `MAX_SCRIPT_ELEMENT_SIZE`
* number of elements on stack is constant 

Unit tests:
1. `0 OP_ZEROES -> OP_0` for all values of 0 (positive zero, negative zero, `OP_0`)
2. `a OP_ZEROES -> failure` where `!isnum(a)`
3. `a OP_ZEROES -> failure` where `a < 0`
4. `a OP_ZEROES -> failure` where `a > MAX_SCRIPT_ELEMENT_SIZE`
5. valid samples
 
Still to investigate: minimal encoding of numbers – could this be used to produce an invalid number which would cause a failure?

### OP_REPEAT
Produce array of repeated bytes.

	x n OP_REPEAT → out
	
Examples: 
* `0x00 2 OP_REPEAT -> 0x0000`
* `0x12FE 3 OP_REPEAT -> 0x12FE12FE12FE`
	
The operator must fail if:
1. `!isnum(n)` - `n` is not a number
2. `n < 0` - `n` is negative 
2. `len(x)*n > MAX_SCRIPT_ELEMENT_SIZE` - the length of the result would be too large

Notes:
* repeating an array zero times is a valid operation and results in `OP_0`
* repeating an empty array (`x = OP_0`) is a valid operation and produces an empty array (`OP_0`)

Impact of successful execution:
* stack memory use increased by `(len(x) * (n - 1)) - len(n)`, maximum `MAX_SCRIPT_ELEMENT_SIZE`
* number of elements on stack is reduced by one

Unit tests: 
1. `x n OP_REPEAT -> failure` where `!isnum(n)` - fails if `n` not a number
2. `x -1 OP_REPEAT -> failure` - fails if `n < 0`
3. `x 0 OP_REPEAT -> OP_0` - repeating any array zero times results in `OP_0`, for all types of zero
4. `OP_0 n OP_REPEAT → OP_0` – repeating an empty array an arbitrary number of times produces an empty array
5. `OP_0 (MAX_SCRIPT_ELEMENT_SIZE + 1) OP_REPEAT -> OP_0` - should not fail because output will still be `OP_0`
5. `OP_1 (MAX_SCRIPT_ELEMENT_SIZE + 1) OP_REPEAT -> failure` - result is too large   
6. valid samples

Still to investigate: same as OP_ZEROES re minimal encoding

### OP_PADLEFT
Pad the left of the byte array with zeroes until `len(out) = n`

	x n OP_PADLEFT → out
	
Examples:
* `0x1122 4 OP_PADLEFT -> 0x00001122`
	
The operator must fail if:
1. `!isnum(n)` - `n` is not a number
2. `n < 0` - `n` is less than zero
3. `n > MAX_SCRIPT_ELEMENT_SIZE` - the length of the result would be too large

Notes:
* `n = 0` is valid
* `n <= len(x)` is valid and produces and output of `x`

Impact of successful execution:
* stack memory use increased by `max(n - len(n), 0)`, maximum `MAX_SCRIPT_ELEMENT_SIZE`
* number of elements on stack is reduced by one

Unit tests:
1. `x n OP_PADLEFT -> failure` where `!isnum(n)` - fails if `n` not a number
2. `x -1 OP_PADLEFT -> failure` - fails if `n < 0`
3. `x 0 OP_PADLEFT -> x` for all number zero
4. `OP_0 MAX_SCRIPT_ELEMENT_SIZE+1 OP_PADLEFT -> failure` - too large
5. `OP_0 MAX_SCRIPT_ELEMENT_SIZE OP_PADLEFT -> out` - `out` is an array of `MAX_SCRIPT_ELEMENT_SIZE` zeros
6. `large 1 OP_PADLEFT -> failure` where `len(large) = MAX_SCRIPT_ELEMENT_SIZE`
7. valid samples

### OP_MATCHLEN

Match the length of `a` and `b` by padding the the shorter of the two to the left with zeroes.

    `a b OP_MATCHLEN` -> out

Examples:
* `0x1122 0xAA OP_MATCHLEN -> 0x1122 0x00AA`
* `0x11 0xAABB OP_MATCHLEN -> 0x0011 0xAABB`

Operand consumption:

Operand stack elements are not consumed by this operation.  However the shortest length operand will be replaced with it's padded
equivilent.

The operator must fail if:
1. `len(a) or len(b) > MAX_SCRIPT_ELEMENT_SIZE`

Notes:
* Where `len(a) == 0` `a` will be a replaced with a zero byte array of size `len(b)` 
* Where `len(b) == 0` `b` will be a replaced with a zero byte array of size `len(a)`
* Where `len(a) == len(b)` this is not a failure condition but is functionally equivilent to `OP_NOP`

Impact of successful execution:
* stack memory use increased by `abs(len(a) - len(b))`, maximum `MAX_SCRIPT_ELEMENT_SIZE`
* number of elements on stack is constant.

Unit tests:
1. `a b OP_MATCHLEN -> failure` where `len(a) or len(b) = MAX_SCRIPT_ELEMENT_SIZE`
1. `OP_0 b OP_MATCHLEN -> out` replaces OP_0 with zero byte array of `len(b)`
1. `a OP_0 OP_MATCHLEN -> out` replaces OP_0 with zero byte array of `len(a)`
1. `OP_0 OP_0 OP_MATCHLEN -> out` top two elements of stack remain unchanged
1. `long_a short_b OP_MATCHLEN -> out` `short_b` is padded to `len(long_a)`
1. `short_a long_b OP_MATCHLEN -> out` `short_a` is padded to `len(long_b)`

7. valid samples

## Reference implementation

TODO

## References

<a name="op_codes">[1]</a> https://en.bitcoin.it/wiki/Script#Opcodes
