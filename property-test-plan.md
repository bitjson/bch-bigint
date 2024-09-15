# Property Test Plan

WIP implementation: https://gitlab.com/cculianu/bitcoin-cash-node/-/blob/wip_bca_script_big_int/src/test/bigint_script_property_tests.cpp

## OP_1ADD (0x8b)

1. Stack depth
    - Fail: `{undersized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Pass: `{exact-sized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Fail: `{oversized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
2. Minimal encoding
    - Fail: `{stack: 0, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN <0x80> OP_CAT {opcode} OP_DROP OP_1`
    - Fail: `{stack: a, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN {opcode} OP_DROP OP_1`
3. Successor
    - Pass: `<a> OP_DUP OP_1ADD OP_LESSTHAN`
4. Result is always one more than the input
    - Pass: `<a> OP_DUP OP_1ADD OP_SWAP OP_SUB OP_1 OP_NUMEQUAL`
5. Symmetry with 1SUB
    - Pass: `<a> OP_DUP OP_1ADD OP_1SUB OP_NUMEQUAL`
6. Applying OP_1ADD a number of times is equivalent to adding the number.
    - Pass: `<a> OP_DUP OP_3 OP_ADD OP_SWAP OP_1ADD OP_1ADD OP_1ADD OP_EQUAL`
7. Overflow
    - Pass: `<a> OP_1ADD OP_DROP OP_1`, for a < MAX
    - Fail: `<a> OP_1ADD OP_DROP OP_1`, for a = MAX
8. Output minimal encoding: implicitly tested in test 3.

## OP_1SUB (0x8c)

1. Stack depth
    - Fail: `{undersized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Pass: `{exact-sized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Fail: `{oversized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
2. Minimal encoding
    - Fail: `{stack: 0, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN <0x80> OP_CAT {opcode} OP_DROP OP_1`
    - Fail: `{stack: a, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN {opcode} OP_DROP OP_1`
3. Predecessor
    - Pass: `<a> OP_DUP OP_1SUB OP_GREATERTHAN`
4. Result is always one less than the input.
    - Pass: `<a> OP_DUP OP_1SUB OP_SUB OP_1 OP_NUMEQUAL`
5. Symmetry with 1ADD
    - Pass: `<a> OP_DUP OP_1SUB OP_1ADD OP_NUMEQUAL`
6. Applying OP_1SUB a number of times is equivalent to subtracting the number.
    - Pass: `<a> OP_DUP OP_3 OP_SUB OP_SWAP OP_1SUB OP_1SUB OP_1SUB OP_NUMEQUAL`
7. Underflow
    - Pass: `<a> OP_1SUB OP_DROP OP_1`, for a > MIN
    - Fail: `<a> OP_1SUB OP_DROP OP_1`, for a = MIN
8. Output minimal encoding: implicitly tested in test 3.

## OP_NEGATE (0x8f)

1. Stack depth
    - Fail: `{undersized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Pass: `{exact-sized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Fail: `{oversized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
2. Minimal encoding
    - Fail: `{stack: 0, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN <0x80> OP_CAT {opcode} OP_DROP OP_1`
    - Fail: `{stack: a, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN {opcode} OP_DROP OP_1`
3. Double negation
    - Pass: `<a> OP_DUP OP_NEGATE OP_NEGATE OP_NUMEQUAL`
4. Negation is equivalent to multiplying with -1
    - Pass: `<a> OP_DUP OP_NEGATE OP_1NEGATE OP_MUL OP_NUMEQUAL`
5. Sum of a number and its negation is zero
    - Pass: `<a> OP_DUP OP_NEGATE OP_ADD OP_0 OP_NUMEQUAL`
6. Negating zero returns zero
    - Pass: `OP_0 OP_NEGATE OP_0 OP_NUMEQUAL`
7. Output minimal encoding: implicitly tested in test 3.

## OP_ABS (0x90)

1. Stack depth
    - Fail: `{undersized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Pass: `{exact-sized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Fail: `{oversized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
2. Minimal encoding
    - Fail: `{stack: 0, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN <0x80> OP_CAT {opcode} OP_DROP OP_1`
    - Fail: `{stack: a, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN {opcode} OP_DROP OP_1`
3. Absolute value of any positive number is the number itself
    - Pass: `<a> OP_DUP OP_ABS OP_NUMEQUAL` for a >= 0
    - Fail: `<a> OP_DUP OP_ABS OP_NUMEQUAL` for a < 0
4. Negated absolute value of any negative number is the number itself
    - Pass: `<a> OP_DUP OP_ABS OP_NEGATE OP_NUMEQUAL` for a <= 0
    - Fail: `<a> OP_DUP OP_ABS OP_NEGATE OP_NUMEQUAL` for a > 0
5. Output minimal encoding: implicitly tested in tests 3 & 4.

## OP_NOT (0x91)

1. Stack depth
    - Fail: `{undersized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Pass: `{exact-sized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Fail: `{oversized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
2. Minimal encoding
    - Fail: `<0> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN <0x80> OP_CAT {opcode} OP_DROP OP_1`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN {opcode} OP_DROP OP_1`
3. NOT of zero is one
    - Pass: `OP_0 OP_NOT OP_1 OP_NUMEQUAL`
4. NOT of non-zero is zero
    - Pass: `<a> OP_NOT OP_0 OP_NUMEQUAL`
5. Double not
    - Pass: `<a> OP_DUP OP_NOT OP_NOT OP_SWAP OP_0NOTEQUAL OP_NUMEQUAL`
6. Distributivity I
    - Pass: `<a> <b> OP_2DUP OP_BOOLOR OP_NOT OP_ROT OP_NOT OP_ROT OP_NOT OP_BOOLAND OP_EQUAL`
7. Distributivity II
    - Pass: `<a> <b> OP_2DUP OP_BOOLAND OP_NOT OP_ROT OP_NOT OP_ROT OP_NOT OP_BOOLOR OP_EQUAL`
8. Output minimal encoding: implicitly tested in test 5.

## OP_0NOTEQUAL (0x92)

1. Stack depth
    - Fail: `{undersized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Pass: `{exact-sized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
    - Fail: `{oversized stack} {opcode} OP_DEPTH OP_{depthOut} OP_NUMEQUALVERIFY {OP_DROP x depthOut} OP_1`
2. Minimal encoding
    - Fail: `{stack: 0, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN <0x80> OP_CAT {opcode} OP_DROP OP_1`
    - Fail: `{stack: a, n} OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN {opcode} OP_DROP OP_1`
3. 0NOTEQUAL of zero is zero
    - Pass: `OP_0 OP_0NOTEQUAL OP_0 OP_NUMEQUAL`
4. 0NOTEQUAL of non-zero is one
    - Pass: `{stack: a} OP_0NOTEQUAL OP_1 OP_NUMEQUAL`
5. Double 0NOTEQUAL
    - Pass: `{stack: a} OP_DUP OP_0NOTEQUAL OP_0NOTEQUAL OP_SWAP OP_NOT OP_NOT OP_NUMEQUAL`
6. Output minimal encoding: implicitly tested in test 4.

## OP_ADD (0x93)

1. Stack underflow must fail
    - Fail: `OP_ADD OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
    - Fail: `<1> OP_ADD OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
2. Any non-numerical input value must fail
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_0 OP_ADD OP_DROP OP_1`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_0 OP_SWAP OP_ADD OP_DROP OP_1`
3. Any non-numerical zero input value must fail
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_0 OP_ADD OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_0 OP_ADD OP_DROP OP_1`
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_0 OP_SWAP OP_ADD OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_0 OP_SWAP OP_ADD OP_DROP OP_1`
4. Commutativity: a + b == b + a
    - Pass: `<a> <b> OP_2DUP OP_ADD OP_SWAP OP_ROT OP_ADD OP_NUMEQUAL`
5. Associativity: (a + b) + c == a + (b + c)
    - Pass: `<a> <b> <c> OP_3DUP OP_ROT OP_ROT OP_ADD OP_ADD OP_SWAP OP_2SWAP OP_ROT OP_ADD OP_ADD OP_NUMEQUAL`
6. Identity: a + 0 == 0 + a == a
    - Pass: `<a> OP_DUP OP_0 OP_2DUP OP_SWAP OP_ADD OP_ROT OP_ROT OP_ADD OP_DUP OP_ROT OP_ROT OP_NUMEQUALVERIFY OP_NUMEQUAL`
7. Successor: a + b > a
    - Pass: `<a> <b> OP_OVER OP_SWAP OP_ADD OP_SWAP OP_GREATERTHAN`
8. Symmetry with subtraction: (a + b) - b == a
    - Pass: `<a> <b> OP_2DUP OP_ADD OP_SWAP OP_SUB OP_NUMEQUAL`
9. Output overflow & underflow behavior
    - Pass: `<a> <b> OP_ADD OP_DROP OP_1` when `a + b` is inside `[MIN, MAX]` range
    - Fail: `<a> <b> OP_ADD OP_DROP OP_1` when `a + b` is outside `[MIN, MAX]` range
10. Output minimal encoding: implicitly tested by OP_NUMEQUAL in test 4.

## OP_SUB (0x94)

1. Stack underflow must fail
    - Fail: `OP_SUB OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
    - Fail: `<1> OP_SUB OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
2. Any non-numerical input value must fail
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_0 OP_SUB OP_DROP OP_1`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_0 OP_SWAP OP_SUB OP_DROP OP_1`
3. Any non-numerical zero input value must fail
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_0 OP_SUB OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_0 OP_SUB OP_DROP OP_1`
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_0 OP_SWAP OP_SUB OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_0 OP_SWAP OP_SUB OP_DROP OP_1`
4. Anti-commutativity: a - b == -(b - a)
    - Pass: `<a> <b> OP_2DUP OP_SUB OP_SWAP OP_ROT OP_SUB OP_NEGATE OP_NUMEQUAL`
5. Non-associativity: (a - b) - c == a - (b + c)
    - Pass: `<a> <b> <c> OP_3DUP OP_ADD OP_SUB OP_SWAP OP_2SWAP OP_SUB OP_SWAP OP_SUB OP_NUMEQUAL`
6. Identity: a - 0 == a
    - Pass: `<a> OP_DUP OP_0 OP_SUB OP_NUMEQUAL`
7. Sign: 0 - a == -a
    - Pass: `<a> OP_DUP OP_0 OP_SWAP OP_SUB OP_SWAP OP_NEGATE OP_NUMEQUAL`
8. Predecessor: a - b < a
    - Pass: `<a> <b> OP_OVER OP_SWAP OP_SUB OP_SWAP OP_LESSTHAN`
9. Symmetry with addition: (a - b) + b == a
    - Pass: `<a> <b> OP_2DUP OP_SUB OP_SWAP OP_ADD OP_NUMEQUAL`
10. Output overflow & underflow behavior
    - Pass: `<a> <b> OP_SUB OP_DROP OP_1` when `a - b` is inside `[MIN, MAX]` range
    - Fail: `<a> <b> OP_SUB OP_DROP OP_1` when `a - b` is outside `[MIN, MAX]` range
11. Output minimal encoding: implicitly tested by OP_NEGATE in test 4.

## OP_MUL (0x95)

1. Stack underflow must fail
    - Fail: `OP_MUL OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
    - Fail: `<1> OP_MUL OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
2. Any non-numerical input value must fail
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_0 OP_MUL OP_DROP OP_1`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_0 OP_SWAP OP_MUL OP_DROP OP_1`
3. Any non-numerical zero input value must fail
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_0 OP_MUL OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_0 OP_MUL OP_DROP OP_1`
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_0 OP_SWAP OP_MUL OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_0 OP_SWAP OP_MUL OP_DROP OP_1`
4. Commutativity: a * b == b * a
    - Pass: `<a> <b> OP_2DUP OP_MUL OP_SWAP OP_ROT OP_MUL OP_NUMEQUAL`
5. Associativity: (a * b) * c == a * (b * c)
    - Pass: `<a> <b> <c> OP_3DUP OP_ROT OP_ROT OP_MUL OP_MUL OP_SWAP OP_2SWAP OP_ROT OP_MUL OP_MUL OP_NUMEQUAL`
6. Distributivity: a * (b + c) == (a * b) + (a * c)
    - Pass: `<a> <b> <c> OP_3DUP OP_ADD OP_MUL OP_SWAP OP_2SWAP OP_OVER OP_SWAP OP_MUL OP_SWAP OP_ROT OP_MUL OP_ADD OP_NUMEQUAL`
7. Identity: x * 1 == 1 * x == x
    - Pass: `<a> OP_DUP OP_1 OP_2DUP OP_SWAP OP_MUL OP_ROT OP_ROT OP_MUL OP_DUP OP_ROT OP_ROT OP_NUMEQUALVERIFY OP_NUMEQUAL`
8. Negation: x * (-1) == -x
    - Pass: `<a> OP_DUP OP_1NEGATE OP_MUL OP_SWAP OP_NEGATE OP_NUMEQUAL`
9. Zero: x * 0 == 0 * x == 0
    - Pass: `<a> OP_DUP OP_0 OP_MUL OP_NOT OP_VERIFY OP_0 OP_SWAP OP_MUL OP_NOT`
10. Order: a * b < a * c
    - Pass: `<a> <b> <c> OP_SWAP OP_ROT OP_DUP OP_ROT OP_MUL OP_SWAP OP_ROT OP_MUL OP_LESSTHAN`, for a > 0 and b < c
    - Fail: `<a> <b> <c> OP_SWAP OP_ROT OP_DUP OP_ROT OP_MUL OP_SWAP OP_ROT OP_MUL OP_LESSTHAN`, for a >= 0 and b >= c
11. Symmetry with division: (a * b) / b == a
    - Pass: `<a> <b> OP_2DUP OP_MUL OP_SWAP OP_DIV OP_NUMEQUAL`
12. Adding a value to itself a number of times is equivalent to multiplying with a positive number.
    - Pass: `<a> OP_DUP OP_4 OP_MUL OP_SWAP OP_DUP OP_DUP OP_DUP OP_ADD OP_ADD OP_ADD OP_NUMEQUAL`
13. Subtracting a value from itself a number of times is equivalent to multiplying with a negative number.
    - Pass: `<a> OP_DUP OP_4 OP_NEGATE OP_MUL OP_SWAP OP_DUP OP_DUP OP_DUP OP_DUP OP_DUP OP_SUB OP_SWAP OP_SUB OP_SWAP OP_SUB OP_SWAP OP_SUB OP_SWAP OP_SUB OP_NUMEQUAL`
14. Output overflow & underflow behavior
    - Pass: `<a> <b> OP_MUL OP_DROP OP_1` when `a * b` is inside `[MIN, MAX]` range
    - Fail: `<a> <b> OP_MUL OP_DROP OP_1` when `a * b` is outside `[MIN, MAX]` range
15. Output minimal encoding: implicitly tested by OP_NUMEQUAL in test 4.

## OP_DIV (0x96)

1. Stack underflow must fail
    - Fail: `OP_DIV OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
    - Fail: `<1> OP_DIV OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
2. Any non-numerical input value must fail
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_1 OP_DIV OP_DROP OP_1`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_1 OP_SWAP OP_DIV OP_DROP OP_1`
3. Any non-numerical zero input value must fail
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_1 OP_DIV OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_1 OP_DIV OP_DROP OP_1`
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_1 OP_SWAP OP_DIV OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_1 OP_SWAP OP_DIV OP_DROP OP_1`
4. Consistency with inverse operation: (a / b) * b + (a % b) == a (for b != 0)
    - Pass: `<a> <b> OP_2DUP OP_MOD OP_SWAP OP_ROT OP_SWAP OP_2DUP OP_DIV OP_MUL OP_ROT OP_ADD OP_NUMEQUAL`
5. Distributivity: (a + b) / c == a / c + b / c + (a % c + b % c - (a + b) % c) / c (for c != 0)
    - Pass: `<a> <b> <c> OP_2 OP_PICK OP_2 OP_PICK OP_ADD OP_OVER OP_DIV OP_3 OP_PICK OP_2 OP_PICK OP_DIV OP_2OVER OP_DIV OP_ADD OP_4 OP_PICK OP_3 OP_PICK OP_MOD OP_4 OP_PICK OP_4 OP_PICK OP_MOD OP_ADD OP_2ROT OP_ADD OP_4 OP_PICK OP_MOD OP_SUB OP_3 OP_ROLL OP_DIV OP_ADD OP_NUMEQUAL`
6. Identity: a / 1 == a
    - Pass: `<a> OP_DUP OP_1 OP_DIV OP_NUMEQUAL`
7. Negation: a / (-1) == -a
    - Pass: `<a> OP_DUP OP_1NEGATE OP_DIV OP_SWAP OP_NEGATE OP_NUMEQUAL`
8. Self-division: a / a == 1 (for a != 0)
    - Pass: `<a> OP_DUP OP_DIV OP_1 OP_NUMEQUAL`
9. Dividing a zero: 0 / a == 0 (for a != 0)
    - Pass: `<a> OP_0 OP_SWAP OP_DIV OP_0 OP_NUMEQUAL`
10. Division by zero: a / 0 must fail.
    - Fail: `<a> OP_0 OP_DIV OP_0 OP_NUMEQUAL`
11. Output minimal encoding: implicitly tested by OP_NUMEQUAL in test 6.

## OP_MOD (0x97)

1. Stack underflow must fail
    - Fail: `OP_MOD OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
    - Fail: `<1> OP_MOD OP_DEPTH OP_1 OP_NUMEQUAL OP_NIP`
2. Any non-numerical input value must fail
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_1 OP_MOD OP_DROP OP_1`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_ADD OP_NUM2BIN OP_1 OP_SWAP OP_MOD OP_DROP OP_1`
3. Any non-numerical zero input value must fail
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_1 OP_MOD OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_1 OP_MOD OP_DROP OP_1`
    - Fail: `<n> OP_0 OP_SWAP OP_NUM2BIN OP_1 OP_SWAP OP_MOD OP_DROP OP_1`
    - Fail: `<n> OP_1SUB OP_0 OP_SWAP OP_NUM2BIN <0x80> OP_CAT OP_1 OP_SWAP OP_MOD OP_DROP OP_1`
4. Power Identity: (a * a) % a == 0 (for a != 0).
    - Pass: `<a> OP_DUP OP_DUP OP_MUL OP_SWAP OP_MOD OP_NOT`
5. Modulo by Zero:
    - Fail: `<a> OP_0 OP_MOD OP_1 OP_DROP`
6. Repeat Identity: (a % b) % b == a % b (for b != 0).
    - Pass: `<a> <b> OP_2DUP OP_SWAP OP_OVER OP_MOD OP_SWAP OP_MOD OP_ROT OP_ROT OP_MOD OP_NUMEQUAL`
7. Sign: a % b == a % (-b) (for b != 0).
    - Pass: `<a> <b> OP_2DUP OP_MOD OP_ROT OP_ROT OP_NEGATE OP_MOD OP_NUMEQUAL`
8. Negation: -(a % b) == (-a) % (-b) (for b != 0).
    - Pass: `<a> <b> OP_2DUP OP_MOD OP_NEGATE OP_ROT OP_NEGATE OP_ROT OP_NEGATE OP_MOD OP_NUMEQUAL`
9. Inverse: ((−a % b) + (a % b)) % b == 0 (for b != 0).
    - Pass: `<a> <b> OP_2DUP OP_MOD OP_SWAP OP_ROT OP_NEGATE OP_SWAP OP_TUCK OP_MOD OP_ROT OP_ADD OP_SWAP OP_MOD OP_NOT`
10. Output minimal encoding: implicitly tested by OP_NUMEQUAL in test 6.
11. Consistency with MUL & DIV operations: (a / b) * b + (a % b) == a (for b != 0)
    - Test is part of OP_DIV.

## OP_BOOLAND (0x9a)

1. Stack underflow must fail
    - Fail: `OP_BOOLAND <1>`
    - Fail: `<1> OP_BOOLAND <1>`
2. Any non-numerical value must fail.
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_SUB OP_NUM2BIN <1> OP_BOOLAND OP_DROP <1>`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_SUB OP_NUM2BIN <1> OP_SWAP OP_BOOLAND OP_DROP <1>`
3. Result is 1 if both inputs are non-zero, 0 otherwise
    - Pass: `<a> <b> OP_2DUP OP_BOOLAND OP_ROT OP_0NOTEQUAL OP_ROT OP_0NOTEQUAL OP_BOOLAND OP_EQUAL`
4. Commutativity: (a && b) == (b && a)
    - Pass: `<a> <b> OP_2DUP OP_BOOLAND OP_SWAP OP_ROT OP_BOOLAND OP_EQUAL`
5. Idempotence: (a && a) == (a != 0)
    - Pass: `<a> OP_DUP OP_DUP OP_BOOLAND OP_SWAP OP_0NOTEQUAL OP_EQUAL`
6. De Morgan's law: !(a && b) == (!a || !b)
    - Pass: `<a> <b> OP_2DUP OP_BOOLAND OP_NOT OP_ROT OP_NOT OP_ROT OP_NOT OP_BOOLOR OP_EQUAL`
7. Distributive law: (a || b) && c == (a && c) || (b && c)
    - Pass: `<a> <b> <c> OP_3DUP OP_ROT OP_ROT OP_BOOLOR OP_ROT OP_BOOLAND OP_ROT OP_ROT OP_3DUP OP_ROT OP_BOOLAND OP_ROT OP_ROT OP_BOOLAND OP_BOOLOR OP_EQUAL`

## OP_BOOLOR (0x9b)

1. Stack underflow must fail
    - Fail: `OP_BOOLOR <1>`
    - Fail: `<1> OP_BOOLOR <1>`
2. Any non-numerical value must fail.
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_SUB OP_NUM2BIN <1> OP_BOOLOR OP_DROP <1>`
    - Fail: `<a> <n> OP_SWAP OP_SIZE OP_ROT OP_SUB OP_NUM2BIN <1> OP_SWAP OP_BOOLOR OP_DROP <1>`
3. Result is 1 if at least one input is non-zero, 0 otherwise
    - Pass: `<a> <b> OP_2DUP OP_BOOLOR OP_ROT OP_0NOTEQUAL OP_ROT OP_0NOTEQUAL OP_BOOLOR OP_EQUAL`
4. Commutativity: (a || b) == (b || a)
    - Pass: `<a> <b> OP_2DUP OP_BOOLOR OP_SWAP OP_ROT OP_BOOLOR OP_EQUAL`
5. Idempotence: (a || a) == (a != 0)
    - Pass: `<a> OP_DUP OP_DUP OP_BOOLOR OP_SWAP OP_0NOTEQUAL OP_EQUAL`
6. De Morgan's law: !(a || b) == (!a && !b)
    - Pass: `<a> <b> OP_2DUP OP_BOOLOR OP_NOT OP_ROT OP_NOT OP_ROT OP_NOT OP_BOOLAND OP_EQUAL`
7. Absorption law: a || (a && b) == a
    - Pass: `<a> <b> OP_2DUP OP_BOOLAND OP_ROT OP_DUP OP_ROT OP_BOOLOR OP_ROT OP_0NOTEQUAL OP_EQUAL`

## OP_NUMEQUAL (0x9c)

1. Result is 1 if inputs are equal, 0 otherwise
   Test template: `<a> <b> OP_2DUP OP_NUMEQUAL OP_ROT OP_ROT OP_SUB <0> OP_EQUAL OP_EQUAL`
2. Reflexivity: x NUMEQUAL x == 1
   Test template: `<a> OP_DUP OP_NUMEQUAL <1> OP_EQUAL`
3. Symmetry: a NUMEQUAL b == b NUMEQUAL a
   Test template: `<a> <b> OP_NUMEQUAL <b> <a> OP_NUMEQUAL OP_EQUAL`

## OP_NUMEQUALVERIFY (0x9d)

1. Same as NUMEQUAL, but fails if result is 0
   Test template: `<a> OP_DUP OP_NUMEQUALVERIFY <1>`

## OP_NUMNOTEQUAL (0x9e)

1. Result is 0 if inputs are equal, 1 otherwise
   Test template: `<a> <b> OP_2DUP OP_NUMNOTEQUAL OP_ROT OP_ROT OP_NUMEQUAL OP_NOT OP_EQUAL`

## OP_LESSTHAN (0x9f)

1. Result is 1 if first input is less than second, 0 otherwise
   Test template: `<a> <b> OP_2DUP OP_LESSTHAN OP_ROT OP_ROT OP_SUB <0> OP_LESSTHAN OP_EQUAL`
2. Transitivity: if a < b and b < c, then a < c
   Test template: `<a> <b> <c> OP_3DUP OP_ROT OP_SWAP OP_LESSTHAN OP_SWAP OP_ROT OP_LESSTHAN OP_2SWAP OP_LESSTHAN OP_BOOLAND OP_LESSTHAN OP_NOT`

## OP_GREATERTHAN (0xa0)

1. Result is 1 if first input is greater than second, 0 otherwise
   Test template: `<a> <b> OP_GREATERTHAN <b> <a> OP_LESSTHAN OP_EQUAL`

## OP_LESSTHANOREQUAL (0xa1)

1. Result is 1 if first input is less than or equal to second, 0 otherwise
   Test template: `<a> <b> OP_LESSTHANOREQUAL <a> <b> OP_LESSTHAN <a> <b> OP_NUMEQUAL OP_BOOLOR OP_EQUAL`

## OP_GREATERTHANOREQUAL (0xa2)

1. Result is 1 if first input is greater than or equal to second, 0 otherwise
   Test template: `<a> <b> OP_GREATERTHANOREQUAL <b> <a> OP_LESSTHANOREQUAL OP_EQUAL`

## OP_MIN (0xa3)

1. Result is the smaller of the two inputs
   Test template: `<a> <b> OP_MIN OP_DUP <a> OP_LESSTHANOREQUAL OP_SWAP <b> OP_LESSTHANOREQUAL OP_BOOLAND`

## OP_MAX (0xa4)

1. Result is the larger of the two inputs
   Test template: `<a> <b> OP_MAX OP_DUP <a> OP_GREATERTHANOREQUAL OP_SWAP <b> OP_GREATERTHANOREQUAL OP_BOOLAND`

## OP_WITHIN (0xa5)

1. Result is 1 if x is within the range [min, max], 0 otherwise
   Test template: `<x> <min> <max> OP_WITHIN <x> <min> OP_GREATERTHANOREQUAL <x> <max> OP_LESSTHANOREQUAL OP_BOOLAND OP_EQUAL`

These test scripts can be used in conjunction with property-based testing to generate a wide range of inputs and verify the behavior of each operation in the BCHN virtual machine.
