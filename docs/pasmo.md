# PASMO Assembler Documentation

This document has been extracted from the official
[Pasmo documentation](http://pasmo.speccy.org/pasmodoc.html) and is copyright
(c) 2004-2005 Julián Albo.

The document has been updated to use the markdown format. Along with some
spelling and grammar fixes, there have are also been minor changes to the
text for clarification purposes -- Michael R. Cook.


# Source code format


### Generalities

Source code files must be valid text files in the platform used. The use of,
for example, UNIX text files under Pasmo in windows, is unsupported and the
result is undefined (may depend of the compiler used to build Pasmo, for
example). The result of the use of a file that contains vertical tab or form
feed characters is also undefined.

Everything after a `;` in a line is a comment (unless it is part of a string
literal, of course). There are no multi-line comments, you can use
`IF 0 .... ENDIF` instead (but see `INCLUDE`).

String literals are written to the object file without any character set
translation. Then the use of any character with a different meaning in the
platform were Pasmo is running and the destination machine must be avoided,
and the code of the character may be used instead. That also means that using
Pasmo in any machine that uses a non ASCII compatible character set may be
difficult, and that a source written in UTF-8 may give undesired results. This
may be changed in future versions of Pasmo.

A line may begin with a decimal number followed by blanks. This number is
ignored by the assembler, but is allowed for compatibility with old
assemblers. The line number reported in errors is the sequential number of the
line in the file, not this.

Blanks are significant only in string literals and when they separate lexical
elements. Any number of blanks has the same meaning as one. A blank between
operators and operands is allowed but not required, except when the same
character has meaning as a prefix (`$` and `%`, for example).


### Literals

#### Numeric literals

Numeric literals can be written in _decimal_, _binary_, _octal_ and
_hexadecimal_ formats. Several formats are accepted to provide compatibility
with other assemblers.

A literal that begins with `$` is a hexadecimal constant, except if the
literal is a `$` symbol by itself, in which case it is an operator, see below.

A literal that begins with `#` is a hexadecimal constant, except if there are
two consecutive `#`, see the `##` operator.

A literal that begins with `&` can be a hexadecimal, octal or binary constant,
dependant on the character that follows the `&`:

  - `&H` is hexadecimal
  - `&O` is octal
  - `&X` is hexadecimal

When no suffix if given, the character must be a valid hexadecimal digit and
the constant is hexadecimal.

A literal that begins with `%` is a binary constant, except if the literal is
a `%` symbol by itself, in which case it is an operator, see below.

A literal that begins with a decimal digit can be a decimal, binary, octal or
hexadecimal. If the digit starts with a `0`, followed by a `X` character, it
is treated as a hexadecimal. If not, the suffix of the literal is examined:

  - `D` is decimal
  - `B` is binary
  - `H` is hexadecimal
  - `O` or `Q` is octal

Any other case is treated as a decimal.

NOTE: literals such as `FFFFh` *are not hexadecimal constants, they are
identifiers*. To write this with the suffix notation, you must use `0FFFFh`.

All numeric formats can have `$` signs between the digits to improve
readability. They are ignored.

#### String literals

There are two formats of string literals: single or double quote delimited.

A string literal delimited with single quotes is the simpler format, all
characters are included in the string without any special interpretation, with
the only exception that two consecutive single quotes are taken as one single
quote character to be included in the string. For example: the single quote
delimited string `'That''s all folks'` generates the same string as the double
quote delimited `"That's all folks"`.

A string literal delimited with double quotes is interpreted in a way similar
to the C and C++ languages. The `\` character is taken as escape character,
with the following interpretations: `n` is a new line character (`0A` hex),
`r` is a carriage return (`0D` hex), `t` is a tabulator (`09` hex), `a` is a
bell (`07` hex), `x` indicates that the two next characters will be considered
the hexadecimal code of a char and a char with that code is inserted, an octal
digit prefixes and begins an octal number of up to three digits, and the
corresponding character is inserted into the string, the characters `\` and
`"` means to insert itself in the string, and any other char is reserved for
future use.

A string literal of length `1` can be used as a numeric constant with the
numeric value of the character contained. This allows expressions such as
`'A' + 80h` to be evaluated as expected.


### Identifiers

Identifiers are the names used for labels, `EQU` and `DEFL` symbols, and macro
names and parameters. The names of the Z80 mnemonics, registers and flag
names, and of Pasmo operands and assemble directives are reserved and can not
be used as names of identifiers. Reserved names are case insensitive, even if
case sensitive mode is used.

In the following, 'letter' means an English letter character in upper or lower
case. Characters that correspond to letters in other languages are not allowed
in identifiers.

Identifiers begins with a letter, `_`, `?`, `@` or `.`, followed for zero or
more letter, decimal digit, `_`, `?`, `@`, `.` or `$`. The `$` are ignored,
but a reserved word with a `$` embedded or appended is not recognized as such.

Identifiers that begins with `_` are special when using `autolocal` mode, see
the `--alocal` option and the chapter about labels for details.

Identifiers are case sensitive if the option `--nocase` is not used. When
using `--nocase`, they are always converted to upper case.


### File names

File names are used in the `INCLUDE` and `INCBIN` directives. They follow
special rules.

A file name that begins with a double quote character must end with another
double quote, and the file name contains all character between them without
any special interpretation.

A file name that begins with a single quote character must end with another
single quote, and the file name contains all character between them without
any special interpretation.

In any other case all characters until the next blank or the end of line are
considered part of the file name. Blank characters are `space` and `tab`.


### Labels

A label can be placed at the beginning of any line, before any assembler
mnemonic or directive. Optionally can be followed by a `:`, but is not
recommended to use it in directives, for compatibility with other assemblers.
A line that has a label with no mnemonic nor directive is also valid.

The label has special meaning in the `MACRO`, `EQU` and `DEFL` directives, in
any other case the value of the current code generation position is assigned
to the label.

Labels can be used before their definition, but the result of doing this with
labels assigned with `DEFL` is undefined.

The value of a label cannot be changed unless `DEFL` is used in all
assignments of that label. If the value assigned to a label is different in
the two passes of the assembly the program is illegal, but is not guaranteed
that an error is generated. However, is legal to assign a value undefined in
the first pass (by using an expression that contains a label not yet defined,
for example).

In the default mode a label is global unless declared as `LOCAL` into a
`MACRO`, `REPT` or `IRP` block, see the `LOCAL` directive for details.

In the `autolocal` mode, enabled by using the `--alocal` command line
option, all labels that begin with an `_` are locals. His ambit ends at the
next non local label or in the next `PROC`, `LOCAL`, `MACRO`, `ENDP` or `ENDM`
directive.

Both automatic and explicit local labels are represented in the symbol table
listing as 8 digit hexadecimal numbers, corresponding to the first use of the
label in the source.



# Directives

List of directives supported in Pasmo, in alphabetical order.

### `.ERROR`

Generates an error during assembly if the line is actively used, that is, in a
macro if it gets expanded, in an `IF` if the current branch is taken. All text
following the directive is used as error message.

### `.SHIFT`

Shift `MACRO` arguments, see the chapter about macros.

### `.WARNING`

Same as `.ERROR` but emitting a warning message instead of generating an error.

### `DB`

Define Byte. The argument is a comma separated list of string literals or
numeric expressions. The string literals are inserted in the object code,
and the result of the numeric expression is inserted as a single byte,
truncating it if needed.

### `DEFB`

DEFine Byte, same as `DB`.

### `DEFL`

DEFine Label. Must be preceded by a label. The argument must be a numeric
expression, the result is assigned to the label. The label used can be redefined
with other `DEFL` directive.

### `DEFM`

DEFine Message, same as `DB`.

### `DEFS`

DEFine Space, same as `DS`.

### `DEFW`

Same as `DW`.

### `DS`

Define Space. Take one or two comma separated arguments. The first or only
argument is the amount of space to define, in bytes. The second is the value
used to fill the space, if absent 0 will be used.

### `DW`

Define Word. The argument is a comma separated list of numeric expressions. Each
numeric expression is evaluated as a two byte word and the result inserted in
the code in the Z80 word format.

### `ELSE`

See `IF`.

### `END`

Ends the assembly. All lines after this directive are ignored. If it has an
argument is evaluated as a numeric expression and the result is set as the
program entry point. The result of setting an entry point depends of the type of
code generation used, may be none but even in this case may be used for
documentation purposes.

### `ENDIF`

See `IF`.

### `ENDM`

Ends a macro, see the chapter about macros.

### `ENDP`

Marks the end of a `PROC` block, see `PROC`.

### `EQU`

EQUate. Must be preceded by a label. The argument must be a numeric expression,
the result is assigned to the label. The label used can't be redefined.

### `EXITM`

Exits a macro, see the chapter about macros.

### `IF`

Conditional assembly. The argument must be a numeric expression, a result of `0`
is considered as `false`, any other as `true`. If the argument is `true` the following
code is assembled until the end of the `IF` section or an `ELSE` directive is
encountered, else is ignored. If the `ELSE` directive is present the following code
is ignored if the argument was `true`, or is assembled if was `false`.

The `IF` section is ended with a `ENDIF` or a `ENDM` directive (in the last case
the `ENDM` has also his usual effect).

`IF` can be nested, in that case each `ELSE` and `ENDIF` takes effect only on
his corresponding `IF`, but `ENDM` ends all pending `IF` sections.

### `INCLUDE`

Include a file. See the file names chapter for the conventions used in the
argument. The file is read and the result is the same as if the file were
copied in the current file instead of the `INCLUDE` line.

The file included may contain `INCLUDE` directives, and so on.

`INCLUDE` directives are processed before the assembly phases, so the use of
`IF` directives to conditionally include different files is not allowed.

### `INCBIN`

INClude BINary. include a binary file. Reads a binary file and insert his
content in the generated code at the current position. See the file names
chapter for the conventions used in the argument.

### `IRP`

Repeat a block of code substituting arguments. See the chapter about macros.

### `LOCAL`

Marks identifiers as local to the current block. The block may be a `MACRO`,
`REPT`, `IRP` or `PROC` directive, the local ambit ends in the corresponding
`ENDM` or `ENDP` directive. The ambit begins at the `LOCAL` directive, not at
the beginning of the block, take care with that.

If several `LOCAL` declarations of the same identifier are used in the same
block, only the first has effect, the others are ignored.

### `MACRO`

Defines a macro, see the chapter about macros.

### `ORG`

ORiGin. Establishes the origin position where to place generated code. Several
`ORG` directives can be used in the same program, but if the result is that
code generated overwrites previous, the result is undefined.

### `PROC`

Marks the begin of a `PROC` block. The only effect is to define an ambit for
`LOCAL` directives. The block ends with a corresponding `ENDP` directive. The
recommended use is to enclose a subroutine in a `PROC` block, but can also be
used in any other situation.

### `PUBLIC`

The argument is a comma separated list of identifiers. Each identifier is marked
as public. When using the `--public` command line option only the identifiers
marked as public are included in the symbol table listing.

### `REPT`

REPeaTs a block. See the chapter about macros.



# Operators


### Generalities

All numeric values are taken as 16 bits `unsigned`, using 2 complement or
truncating when required. Logical operators return `FFFF` hex for `true` and `0`
for `false`, in the arguments `0` is `false` and any other value `true`.

Parenthesis may be used to group parts of expressions. They are also used to
express indirections in the Z80 instructions that allows or require it. This can
cause some errors when a parenthesized expression is used in a place were an
indirections is allowed. Pasmo uses some heuristic to allow the expression to be
correctly interpreted, but are far from perfect.

Using the bracket only mode the parenthesis have the unique meaning of grouping
expressions, brackets are required for indirections, thus solving ambiguities.

Short circuit evaluation: the `&&` and `||` operators and the conditional
expression are short circuited. This means that if one of his operators need not
be evaluated, it can include undefined symbols or divisions by `0` without
generating an error (but still must have correct syntax). In the conditional
expression this applies to the branch not taken, in the `&&` operator to the
second operand if the first is `false`, and in the `||` operator to the second
operand if the first is `true`.


### Table of precedence

Table of operators by order of precedence, those in the same line have the same
precedence:

    ## (see note)
    $, NUL, DEFINED
    *, /, MOD, %, SHL, SHR, <<, >>
    +, - (binary)
    EQ, NE, LT, LE, GT, GE, =, !=, <, >, <=, >=
    NOT, ~, !, +, - (unary)
    AND, &
    OR, |, XOR
    &&
    ||
    HIGH, LOW
    ?

The `##` operator is an special case, is processed during the macro expansion,
see the section about MACROS.


### Alphabetic list of operators

- `!`:       Logical not. Returns true if his argument is `0`, false otherwise.
- `!=`:      Same as `NE`.
- `##`:      Identifier pasting operator, see the chapter about macros.
- `$`:       Gives the value of the position counter at the begin of the current
             sentence. For example, in a `DW` directive it gives the position of
             the first item in the list, not the current item.
- `%`:       Same as `MOD`.
- `&`:       Same as `AND`.
- `&&`:      Logical and. True if both operands are `true`.
- `*`:       Multiplication.
- `+`:       Addition or unary `+`.
- `-`:       Subtraction or unary `-`.
- `/`:       Integer division, truncated.
- `<`:       Same as `LT`
- `<<`:      Same as `SHL`
- `<=`:      Same as `LE`
- `=`:       Same as `EQ`
- `>`:       Same as `GT`
- `>=`:      Same as `GE`
- `>>`:      Same as `SHR`
- `?`:       Conditional expression. Must be followed by two expressions separated
             by a `:`, if the expression on the right of `?` is `true`, the
             first expressions is evaluated, if `false`, the second.
- `|`:       Same as `OR`.
- `||`:      Logical or. True if one of his operands is `true`.
- `~`:       Same as `NOT`
- `AND`:     Bitwise and operator.
- `DEFINED`: The argument must be a identifier. The result is `true` if the
             identifier is defined, `false` otherwise.
- `EQ`:      Equals. True if both operands are equal, false otherwise.
- `GE`:      Greater or equal. True if the left operand is greater or equal than the right.
- `GT`:      Greater than. True if the left operand is greater than the right.
- `HIGH`:    Returns the high order byte of the argument.
- `LE`:      Less or equal. True if the left operand is lesser or equal than the right.
- `LOW`:     Returns the low order byte of the argument.
- `LT`:      Less than. True if the left operand is lesser than the right.
- `MOD`:     Modulus. The remainder of the integer division.
- `NE`:      Not equal. False if both operands are equal, true otherwise.
- `NOT`:     Bitwise not. Return the ones complement of his operand.
- `NUL`:     Returns true if there is something at his right, false in other
             case. Useful if the arguments are parameters of macros.
- `OR`:      Bitwise or operator.
- `SHL`:     Shift left. Returns the left operand shifted to the left the number
             of bits specified in the right operand, filling with zeroes.
- `SHR`:     Shift right. Returns the left operand shifted to the right the number
             of bits specified in the right operand, filling with zeroes.
- `XOR`:     Bitwise XOR (exclusive or) operator.



# Macros


### Generalities

There are two types of macro directives: the proper `MACRO` directive and the
repetition directives `REPT` and `IRP`. In addition the `ENDM` and `EXITM` directives
controls the end of the macro expansion.

A macro parameter is an identifier that when the macro is expanded is
substituted by the value of the argument applied. The argument can be empty, or
be composed by one or more tokens. If a `MACRO` is defined inside another macro
directive the external parameters are not substituted, with the other macro
directives the parameter substitution is done beginning by the most external
directive. The `NUL` operator can be used to check if an argument is not empty.
The `.SHIFT` directive can be used to work with an undetermined number of
arguments.

Identifier pasting: inside a `MACRO` the operator `##` can be used to join two
identifiers resulting in another identifier. This is intended to allow the
creation of identifiers dependent of macro arguments.


### Directives

#### `.SHIFT`

Can be used only inside a `MACRO`. The `MACRO` arguments are shifted one place
to the left, the first argument is discarded. If there are not enough arguments
to fill the parameter list after the shift, the remaining arguments get
undefined.

#### `ENDM`

Marks the end of the current `MACRO` definition, or the current `REPT` or `IRP`
block. All `IF` blocks contained in the macro block are also closed.

#### `EXITM`

Exits the current `MACRO`, `REPT` or `IRP` block. In the case of `MACRO`, the
macro expansion is finished, in the other cases the code generation of the block
is terminated and the assembly continues after the corresponding `ENDM`.

#### `IRP`

    IRP parameter, argument list.

Repeats the block of code between the `IRP` directive and his corresponding
`ENDM` one time for each of the arguments.

#### `MACRO`

Defines a macro. There are two forms that can be used:

    name  MACRO [ list of parameters]

or:

    MACRO name [ , list of parameters]

In all cases, list of parameters is a comma separated list of identifiers, and
name is the name assigned to the macro created.

A macro is used by simply specifying his name, and optionally a list of
arguments. The arguments list does not need to have the same length as the
parameter list of the macro. If it is longer, the extra arguments are not used,
but can be retrieved by using `.SHIFT` inside the `MACRO`. If it is shorter some
parameters get undefined, this can be tested inside the `MACRO` by using the
`NUL` operator.

#### `REPT`

Repeats the block of code between the `REPT` directive and his corresponding
`ENDM` the number of times specified in his argument. The argument can be `0`,
in that case the block is skipped.

Additionally a loop variable can be specified. This variable is not a macro
parameter, is used as a `LOCAL` `DEFL` symbol, whose value is incremented in
every loop iteration. The initial value and increment can be specified, with
defaults of `0` and `1` respectively.
