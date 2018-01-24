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
* Signed integer.  Whether signed integers are permitted operands and whether any special handling is required.
* Stack size.  Both number of elements and total size of elements. 
* TODO resource (CPU) utilization - exponential cycle cost attacks
* Overflows.  Defined behaviour in the instance that result of the operation exceeds MAX_SCRIPT_ELEMENT_SIZE
* TODO endian issues ???
* Empty byte vector operands.  Whether empty byte vectors should be allowed as a representation of zero.

## Definitions

* *Stack memory use* - sum of the size of the elements on the stack - gives an indication of impact on memory use
* *Operand order* - in keeping with convention where multiple operands are specified the top most stack item is the 
last operand.  e.g. `x1 x2 OP_CAT` --> x2 is the top stack item and x1 is the next from the top

## Specification

Global failure conditions apply to all operations. These failure conditions must be checked by the implementation when 
it is possible that they will occur:
* for all e : elements on the stack, `0 <= len(e) <= MAX_SCRIPT_ELEMENT_SIZE`
* for each operator, the required number of operands are present on the stack when the operand is executed

These unit tests should be included for every operation:
1. executing the operation with an incorrect number of operands causes a failure

### OP_CAT
Concatenates two operands.

    x1 x2 OP_CAT → out
    
The operator must fail if:
* `0 <= len(out) <= MAX_SCRIPT_ELEMENT_SIZE` - the operation cannot output elements that violate the constraint on the element size
  * note that the concatentation of a zero length operand is valid

Impact of successful execution:
* stack memory use is constant
* number of elements on stack is reduced by one

The limit on the length of the output prevents the memory exhaustion attack and results in the operation having less 
impact on stack size than existing OP_DUP operators.

Unit tests:
1. `maxlen_x y OP_CAT → failure` – concatenating any operand, including a single byte value (e.g. `OP_1`), onto a maximum sized array causes failure
3. `large_x large_y OP_CAT → failure` – concatenating two operands, where the total length is greater than `MAX_SCRIPT_ELEMENT_SIZE`, causes failure
4. `OP_0 OP_0 OP_CAT → OP_0` – concatenating two empty arrays results in an empty array
5. `x OP_0 OP_CAT → x` – concatenating an empty array onto any operand results in the operand, including when `len(x) = MAX_SCRIPT_ELEMENT_SIZE`
6. `OP_0 x OP_CAT → x` – concatenating any operand onto an empty array results in the operand, including when `len(x) = MAX_SCRIPT_ELEMENT_SIZE`
7. `x y OP_CAT → concat(x,y)` – concatenating two operands generates the correct result

### OP_SPLIT
Split the operand at the given position.

    x n OP_SPLIT -> x1 x2

Notes:
* `x` is split at position `n`, where `n` is the number of bytes from the beginning
* `x1` will be the first `n` bytes of `x` and `x2` will be the remaining bytes 
* if `n == 0`, then `x1` is the empty array and `x2 == x`
* *RULE OPTIONS*
    * Liberal: if `n >= len(x)`, then `x1 == x` and `x2` is the empty array. OR
    * Restrictive: if `n > len(x)`, then the operator fail.
* `x n OP_SPLIT OP_CAT` -> `x` - for all `x` and for all `n >= 0`
    
The operator must fail if:
* `!isnum(n)` - `n` is not a number
* `n < 0` - `n` is negative

Impact of successful execution:
* stack memory use is constant (slight reduction by `len(n)`)
* number of elements on stack is constant

Unit tests:
* `OP_0 n OP_SPLIT -> OP_0 OP_0`, for all positive numbers n - execution of OP_SPLIT on empty array results in two empty arrays
* `x 0 OP_SPLIT -> OP_0 x`
* `x len(x) OP_SPLIT -> x OP_0` 

### OP_AND

Boolean *and* between each bit in the operands.

	x1 x2 OP_AND → out

The operator must fail if:
1. `len(x1) != len(x2)` - the length, in bytes, of the two values is not equal

Impact of successful execution:
* stack memory use reduced by `len(x1)`
* number of elements on stack is reduced by one

Unit tests:
1. `x1 x2 OP_AND -> failure`, where `len(x1) != len(x2)` - operation fails when size of operands not equal
2. `x1 x2 OP_AND -> x1 & x2` - check valid results

TODO: minimal encoding of numbers – would this cause numbers (byte arrays where len <= 4) to be automatically left truncated? Is it possible to AND the values 0x0005 and 0x0100?

### OP_OR

Boolean *or* between each bit in the operands.

	x1 x2 OP_OR → out
	
The operator must fail if:
1. `len(x1) != len(x2)` - the length, in bytes, of the two values is not equal

Impact of successful execution:
* stack memory use reduced by `len(x1)`
* number of elements on stack is reduced by one

Unit tests:
1. `x1 x2 OP_OR -> failure`, where `len(x1) != len(x2)` - operation fails when size of operands not equal
2. `x1 x2 OP_OR -> x1 | x2` - check valid results

### OP_XOR
Boolean *xor* between each bit in the operands.

	x1 x2 OP_XOR → out
	
The operator must fail if:
1. `len(x1) != len(x2)` - the length, in bytes, of the two operands is not equal

Impact of successful execution:
* stack memory use reduced by `len(x1)`
* number of elements on stack is reduced by one

Unit tests:
1. `x1 x2 OP_XOR -> failure`, where `len(x1) != len(x2)` - operation fails when size of operands not equal
2. `x1 x2 OP_XOR -> x1 xor x2` - check valid results
    
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

### OP_ZEROES
Produces byte string of zero bytes.

	n OP_ZEROES → out
	
The operator must fail if:
1. `!isnum(n)` - `n` is not a number
2. `n < 0` - the length must be positive
2. `n > MAX_SCRIPT_ELEMENT_SIZE` - the length of the result would be too large

Notes:
* `n = 0` is valid, a zero length byte string is produced

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
	
The operator must fail if:
1. `!isnum(n)` - `n` is not a number
2. `n < 0` - `n` is negative 
2. `len(x)*n > MAX_SCRIPT_ELEMENT_SIZE` - the length of the result would be too large

Note that:
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

### OP_PAD_LEFT
Pad the left of the byte array with zeroes.

	x n OP_PAD_LEFT → out
	
The operator must fail if:
1. `!isnum(n)` - `n` is not a number
2. `n < 0` - `n` is less than zero
3. `len(x)+n > MAX_SCRIPT_ELEMENT_SIZE` - the length of the result would be too large

Note that:
* `n = 0` is valid

Impact of successful execution:
* stack memory use increased by `n - len(n)`, maximum `MAX_SCRIPT_ELEMENT_SIZE`
* number of elements on stack is reduced by one

Unit tests:
1. `x n OP_PAD_LEFT -> failure` where `!isnum(n)` - fails if `n` not a number
2. `x -1 OP_PAD_LEFT -> failure` - fails if `n < 0`
3. `x 0 OP_PAD_LEFT -> x` for all number zero
4. `OP_1 MAX_SCRIPT_ELEMENT_SIZE OP_PAD_LEFT -> failure` - too large
5. `OP_0 MAX_SCRIPT_ELEMENT_SIZE OP_PAD_LEFT -> out` - `out` is an array of `MAX_SCRIPT_ELEMENT_SIZE` zeros
6. `large 1 OP_PAD_LEFT -> failure` where `len(large) = MAX_SCRIPT_ELEMENT_SIZE`
7. valid samples

## Test plan

## References

<a name="op_codes">[1]</a> https://en.bitcoin.it/wiki/Script#Opcodes
