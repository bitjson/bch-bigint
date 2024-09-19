# Specification: High-Precision Arithmetic Opcodes for Bitcoin Cash

Deployment of this specification is proposed for the May 2025 upgrade.

- Activation on `1731672000` MTP, (`2024-11-15T12:00:00.000Z`) on `chipnet`.
- Activation on `1747310400` MTP, (`2025-05-15T12:00:00.000Z`) on the BCH network (`mainnet`), `testnet3`, `testnet4`, and `scalenet`.

This proposal requires [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits).

## Script Number Encoding

The numbers are encoded as variable-length byte arrays in little endian byte order (least significant byte first).

Sign is encoded with a sign bit, and sign bit is defined as the highest order bit, e.g. `ff7f` encodes 32767, `ffff` encodes -32767.
This means that some positive values must have `00` appended so to move the sign bit to the next byte, and allow encoding positive values which would need to use the sign bit (`0x80`), e.g. `ff8000` encodes 33023 and `ff8080` encodes -33023.

Encoding of value 0 is special, it is encoded as an empty stack item of 0 length and NOT with 0-byte (`00`) or "negative" 0-byte (`80`).

Numbers must be minimally encoded, e.g. `0200` could be decoded to number 2 but it is not valid and encoding it as `02` is mandatory.

This upgrade affects two non-arithmetic opcodes that are used to convert an arbitrary byte array to a minimally encoded script number and back.

For reference, we will specify them below as well.

## Casting Operations

### OP_NUM2BIN (0x80)

Pop two items from stack.  
The top-most value is read as a desired length of the output's stack item, and the other one as binary value to be converted.  
The length must be a minimally encoded script number, otherwise the script fails immediately.  
The binary value may be any length and does not need to start out as a minimally encoded script number, however.
If the requested length is larger than `MAX_SCRIPT_ELEMENT_SIZE`, fail immediately.
The value is then transformed to a minimally-encoded script number (meaning trailing zeroes may be popped off).
At this point the value's size may shrink.  
Then, if the new (possibly reduced) length of the value is larger than the requested length, fail immediately.  
Otherwise, taking account of the sign bit, pad the value with 0-bytes until the desired length is reached and then push the result to stack.

Executing the operation on value 0 and length 0 is valid and will return 0 as an empty stack item.  
When positive values are padded the operation will simply add 0-bytes on the higher end, e.g. executing it on `7b` (123) and `05` (5) will return `7b00000000`.  
When negative values are padded the operation will similarly add 0-bytes on the higher end but it must also move the sign bit to highest byte, e.g. executing it on `fb` (-123) and `05` (5) will return `7b00000080`.  
The value to be converted can be a padded number, so this opcode can be used to change padding, too, e.g. executing it on `7b00000000` (123) and `03` (3) will return `7b0000`.  
Similarly, for negative numbers, executing it on `7b00000080` (-123) and `03` (3) will return `7b0080`.  
If called on other encodings of 0 (like `00`, `80`, `0000`, `0080`) etc. it WILL NOT preserve the sign bit, and it will return a "positive" encoding of 0, e.g. executing it on `0080` (-0) and `03` (3) will return `000000` (0).

### OP_BIN2NUM (0x81)

Pop one item from stack.  
Decode the stack item to a numerical value using script number encoding scheme.  
Push the value on stack as a minimally-encoded script number.  

For example, byte sequence `0080` is not a valid script number encoding, but the opcode would convert it to value 0 and return an empty stack item which is the only valid encoding for value 0.
Similarly, byte sequence `ff0080` is not a valid script number encoding, but the opcode would convert it to value -255 and return `ff80` which is the only valid encoding for value -255.

This operation may reduce the size of the stack item but it will never increase it.

Before this upgrade, the operation would fail if it would have to return a value outside the int64 range.
After this upgrade, the operation will never fail because it can not increase the size of the stack item so any byte sequence on stack will be decodeable.

## Arithmetic Operations

**General requirements**

If any of the input stack items is not a minimally-encoded script number then the operation must fail, e.g. trying to add `0100` and `01` must fail rather than return `02`.

The operation must fail if any resulting stack item would exceed `MAX_SCRIPT_ELEMENT_SIZE`.  
Before this upgrade, the operation would fail if any resulting stack item would exceed `nMaxNumSize` which was set to 8 bytes.
With this upgrade, that requirement has been removed and `MAX_SCRIPT_ELEMENT_SIZE` will be the new limit.
The [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) will set `MAX_SCRIPT_ELEMENT_SIZE` to 10,000 bytes, therefore valid range for results of arithmetic operations shall be the inclusive range:

- `[-2^79999 + 1, 2^79999 - 1]`.

Any result must be returned as a minimally encoded script number, e.g. number 1 is to be returned as `01` rather than `0100`, number -1 is to be returned as `81` rather than `0180`, and number 0 is to be returned as empty stack item rather than `00`.

### Unary Operations

Any unary operation must fail if executed on empty stack (stack depth of 0).

#### OP_1ADD (0x8b)

Pop one item from stack, add 1 to the value, push the result on stack.
This may increase the size of the stack item, e.g. if executed on `7f` (127) the result must be `ff00` (128).
This may reduce the size of the stack item, e.g. if executed on `8080` (-128) the result must be `ff` (-127).

#### OP_1SUB (0x8c)

Pop one item from stack, subtract 1 from the value, push the result on stack.
This may increase the size of the stack item, e.g. if executed on `ff` (-127) the result must be `8080` (-128).
This may reduce the size of the stack item, e.g. if executed on `ff00` (128) the result must be `7f` (127).

#### OP_NEGATE (0x8f)

Pop one item from stack, if it is nonzero, flip the sign, if it is zero, leave the item unchanged. In either case, push the result on stack.
This will always result in the input operand and the result having the same byte size.

#### OP_ABS (0x90)

Pop one item from stack, if value is negative flip the sign else do nothing, push the result on stack.
This will result in same size of the result.

#### OP_NOT (0x91)

Pop one item from stack.
If value is 0 change it to 1, else (for any other value) change it to 0.
Push the result on the stack.
This may reduce the size of the stack item to 0, e.g. if executed on `aabbdd` the result will be an empty stack item (0), size of which is 0.
Or, it may increase the size of the stack item from 0 to 1, e.g. if executed on an empty stack item (0) it will yield `01`.

#### OP_0NOTEQUAL (0x92)

Pop one item from stack.
If value is not 0 change it to 1.
If it is 0, leave it unchanged.
Push the result on stack.
This may reduce the size of the stack item to 1, e.g. if executed on `aabbdd` the result will be `01` (1), size of which is 1.
If executed for an empty stack item (0), it will leave the top stack item unchanged.

### Binary Operations

Any binary operation must fail if executed on stack depth of 1 or less.

#### OP_ADD (0x93)

Pop two items from stack, add the values together, push the result on stack.

#### OP_SUB (0x94)

Pop two items from stack, subtract the top-most value from the other one, push the result on stack.

#### OP_MUL (0x95)

Pop two items from stack, multiply the values together, push the result on stack.

#### OP_DIV (0x96)

Pop two items from stack.
If the top-most item (divisor) is 0, fail immediately.
Using *truncated division* divide the 2nd-to-top value (dividend) with the top-most value (divisor).
Push the **quotient** to stack, e.g.

- 3/2 will return 1,
- -3/2 will return -1,
- 3/-2 will return -1, and
- -3/-2 will return 1.

#### OP_MOD (0x97)

Pop two items from stack.
If the top-most item (divisor) is 0, fail immediately.
Using *truncated division* divide the 2nd-to-top value (dividend) with the top-most value (divisor).
Push the **remainder** to stack.
The sign of the result will match the sign of the dividend, e.g.

- 7/3 will return 1,
- -7/3 will return -1,
- 7/-3 will return 1, and
- -7/-3 will return -1.

#### OP_BOOLAND (0x9a)

Pop two items from stack.
If both numbers are non-zero then return 1, else return 0.

#### OP_BOOLOR (0x9b)

Pop two items from stack.
If at least one of the numbers is non-zero then return 1, else return 0.

#### OP_NUMEQUAL (0x9c)

Pop two items from stack.
If the two numbers are equal then return 1, else return 0.

#### OP_NUMEQUALVERIFY (0x9d)

Pop two items from stack.
If the two numbers are equal then continue evaluation with nothing returned to stack, else fail.

#### OP_NUMNOTEQUAL (0x9e)

Pop two items from stack.
If the two numbers are not equal then return 1, else return 0.

#### OP_LESSTHAN (0x9f)

Pop two items from stack.
If the 2nd-to-top value is less than the top-most value then return 1, else return 0.

#### OP_GREATERTHAN (0xa0)

Pop two items from stack.
If the 2nd-to-top value is greater than the top-most value then return 1, else return 0.

#### OP_LESSTHANOREQUAL (0xa1)

Pop two items from stack.
If 2nd-to-top value is less than or equal to the top-most value then return 1, else return 0.

#### OP_GREATERTHANOREQUAL (0xa2)

Pop two items from stack.
If 2nd-to-top value is greater than or equal to the top-most value then return 1, else return 0.

#### OP_MIN (0xa3)

Pop two items from stack.
Return the lesser of the 2 values (if they're equal just return any one value).

#### OP_MAX (0xa4)

Pop two items from stack.
Return the greater of the 2 values (if they're equal just return any one value).

### Ternary Operations

Any ternary operation must fail if executed on stack depth of 2 or less.

#### OP_WITHIN (0xa5)

Pop three items from stack.  
The top-most item defines the "right" and non-inclusive boundary of a range.  
The 2nd-to-top item defines the "left" and inclusive boundary of a range.  
The 3rd-to-top item is the "value" to be evaluated.  
If "value" being evaluated is in the range (greater or equal to the "left" value, and less than the "right" value) then return 1, else return 0.

Consider the following examples (shorthand for `<value> <left> <right> OP_WITHIN`):

- evaluating `1 5 8 OP` will return 0,
- evaluating `5 5 8 OP` will return 1,
- evaluating `7 5 8 OP` will return 1, and
- evaluating `8 5 8 OP` will return 0.

Uses where the "left" value is greater or equal to the "right" value are valid and will always return 0, e.g.

- evaluating `X 8 5 OP` will return 0 for any value of X.
