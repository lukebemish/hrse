---
layout: home
---

HRSE is a config and data-serialization language offering a compromise bwtween the flexibility of a language such as JSON and
the human-readability of a config language such as TOML. HRSE is designed to be easy for a human to read or write, and
sacrifices some ease of parsing for this purpose. Any HRSE document should have an equivalent representation as an 
s-expression, and in fact s-expressions should be valid members within a HRSE document.

## Implementations
* Julia - [Hrse.jl](https://github.com/lukebemish/Hrse.jl)

## Specification

* HRSE does not distinguish between strings and symbols. All strings are symbols, and all symbols are strings.
* A HRSE file should be UTF-8 encoded.
* Lines should end with either a newline ('U+000A') or a newline and a carriage return ('U+000D').
* "whitespace", which does not include line breaks, consists of any nnumber of tabs ('U+0009') or characters in the `Zs`
  (space separator) unicode category.

## Comments

Single-line comments are denoted by a semicolon at the beginning of the line:
    
```hrse
; This is a comment
```

Multi-line comments are denoted by surrounding the comment with parentheses with any number of semicolons. The comment will
only end when the closing parenthesis with the same number of semicolons is reached, without a semicolon before it.

```hrse
(;;; This is
     a multi-line comment ;;;)
(; This is also a valid ;;)
   comment ;)
```

## S-Expressions

HRSE allows s-expressions to be used within it. The s-expression syntax used by HRSE is as follows:

```hrse
(1 2 3) ; A list
(a . b) ; a pair
()      ; The empty list
```

A list of three elements, the second of which is the special symbol `.`, will always be parsed as a pair.

### Indented S-Expression Format

HRSE supports an alternative representation of s-expressions based on indentation. This format is designed to be more human
readable, at the cost of not being able to represent all possible s-expressions. The root of a HRSE file takes the form of
one of these expressions.

The use of the `=` character can be used to denote a key-value pair, with implicit parentheses. For instance, the following
two representations are equivalent:

```hrse
(a . 1)
a=1
```

However, be careful! The implied parentheses are present even if the expression is placed within actual parentheses! For
instance, the following two representations are equivalent:

```hrse
(a = 1)
((a . 1))
```

The use of the `:` character can also be used to represent a key-value pair. However, if the `:` character appears at the
end of a line, followed only by whitespace, the next block is treated differently. The initial indentation level of the block
is the sequence of whitespace characters from the start of the line which ended with ':' to the first non-whitespace character.
The indented block continues until either an outer s-expression nesting the block closes or the indentation returns to the
initial indentation. It is not valid for the indentation to shrink to less than the initial indentation.

The value of a pair with an indented block is a list of every expression in the block, with each line of the block forming a
new element in the list. The value of a pair with an indented block may be an empty list.

```hrse
alphabet:
    a
    b
    c
    d
```

This block is equivalent to the s-expression `(alphabet . (a b c d))`.

If multiple values are present in the same line, they are treated as a list. For instance, the following is equivalent to
`(matrix . ((1 0) (0 1)))`

```hrse
matrix:
    1 0
    0 1
```

If you wish to have a single-element list, you must wrap it as an s-expression:

```hrse
count:
    (1)
    1 2
    1 2 3
```

Indented blocks can contain s-expressions (within which indentation is ignored) and s-expressions may contain indented blocks.

```hrse
outer:
    1 2 3
    (s-expr a:1 b:2 c:
        (1)
        1 2
        1 2 3)
```

A full HRSE file is treated as a single indented block, with the implicit indentation level of 0. This means that every line
of the file will become an element of a list.

## Literals

### Booleans

The boolean literals `#t` and `#f` are supported.

### Symbols and Strings

Symbols and strings represent the same data in HRSE. A symbol is a sequence of characters that obeys the following rules:
* The first character may be any unicode character excepting the following:
  * characters in the `Zs` (space separator) and `Cc` (control) categories.
  * the characters `+`, `-`, `(`, `)`, `"`, `'`, `'`, `:`, `;`, and `#`.
  * the numeric characters `0` through `9`.
* Subsequent characters follow the same rule, but allow the numeric characters `0` through `9`, `+`, and `-`.

Strings are a sequence of characters surrounded by double quotes. They may not cross lines or contain control characters
(those in unicode category `Cc`). The following escape sequences are supported:
* `\n` - newline
* `\r` - carriage return
* `\t` - tab
* `\b` - backspace
* `\f` - form feed
* `\v` - vertical tab
* `\a` - alert
* `\e` - escape
* `\\` - backslash
* `\"` - double quote
* `\uXXXX` - unicode character with the given codepoint. Can have anywhere from 1 to 4 hex digits, taking the longest case-insensitive match.
* `\UXXXXXXXX` - unicode character with the given codepoint. Can have anywhere from 1 to 8 hex digits, taking the longest case-insensitive match.
* `\xXX` - unicode character with the given codepoint. Can have anywhere from 1 to 2 hex digits, taking the longest case-insensitive match.
* `\000` - unicode character with the given codepoint. Can have anywhere from 1 to 3 octal digits, taking the longest match.

### Integers

All types of integer literals may optionally be preceded by a `+` or `-` sign.

HRSE supports decimal, hexidecimal, and binary integer literals. Hexidecimal and binary literals must be prefixed with `0x`
or `0b` (case-insensitive), respectively. Underscores may be placed in integer literals and will be ignored. Hexidecimal
literals are case-insensitive.

```hrse
1234
-1234
0xAABC67
0b10011001
1_000_000
```

### Floats

Float literals take the form of a point character, `.`, surrounded on at least one side by a series of binary digits. Floats
may optionally be preceded by a `+` or `-` sign. Floats may optionally be suffixed with an exponent, which is a `e` or `E`
followed optionally by a `+` or `-` sign, followed by a series of binary digits. Underscores may be placed in float literals.

```hrse
1.0
-1.0
1.0e-10
1.0e+10
+.05
1.
```

The special values `#inf` (or `+#inf`), `-#inf`, and `#nan` are also supported.
