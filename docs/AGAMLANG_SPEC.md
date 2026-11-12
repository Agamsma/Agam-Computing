# AgamLang Specification — v0.1

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 2 — NOT APPROVED.** Owner's provisional decisions of 2026-11-12 incorporated. No interpreter code may be written until the owner opens gate G-LANG in `DECISIONS.md`. |
| Date | 2026-11-12 (revision 2) |
| Location | Implementation in `agam-lang/` (Claude); tests in `agam-lang/tests/` (Codex) |
| Decided by the owner (provisional) | Fusion is the default and only mode (O-07); Python-style indentation with no `end`/`khatam` terminators (O-05); 0-based list indexes (O-06) |
| Still open | The Hinglish keyword table (Q-07), output words for booleans (Q-08), Devanagari keywords (Q-09) |
| Scope | Source format, Fusion keyword system, lexer, grammar, AST, interpreter semantics, diagnostics, command line |
| Out of scope (v0.1) | Dictionaries, string interpolation, modules/imports, classes, file and network I/O, REPL, native compilation, running on AOS |
| Audience | An engineer implementing or testing the interpreter without access to any conversation |

Normative sections define behaviour that tests depend on. Nothing here has been implemented or run.

---

## 1. Design goals

1. **A first program should read like a sentence**: `show "Hello"` / `dikhao "Namaste"`.
2. **Fusion Mode is the default and the only mode** (C-08, O-07): English, Hinglish (Hindi written in Latin letters) and any mixture are valid in the same `.agam` file, even on the same line. There is no flag, setting, file header or per-file language declaration that turns Fusion on or off. There is one lexer, one grammar, one AST and one interpreter.
3. **Errors teach.** Every error says what went wrong, where, and what to try, in plain English or Hinglish.
4. **Python-compatible habits.** Indentation, colons, operators and 0-based lists behave as in Python, so skills transfer (O-05, O-06).
5. **Small and fully specified.** Every construct in v0.1 has defined behaviour, including every error.

---

## 2. Source text

- File extension `.agam`; encoding UTF-8 (a leading byte-order mark is ignored); invalid UTF-8 is error E0100.
- Line endings `\n` or `\r\n` are both accepted and treated identically.
- A comment starts with `#` outside a text literal and runs to the end of the line.
- Blank lines and comment-only lines are ignored, including for indentation.
- A file is a sequence of statements. There is no required `main`.

---

## 3. Fusion keyword system (P-22, P-23, P-24)

### 3.1 Mechanism (normative)

1. The lexer splits each line into raw tokens. A **word** is an identifier-shaped token (§4.3).
2. The **keyword normaliser** walks the words of a logical line from left to right. At each word it tries, in order:
   a. a **two-word phrase**: this word and the next word, if they are adjacent on the same line separated only by spaces or tabs, looked up (lower-cased) in the phrase table;
   b. a **single word**, looked up (lower-cased) in the word table.
   The first match wins (longest match first). A match is replaced by **one** canonical keyword token.
3. A word that matches nothing is an identifier (`IDENT`) — unless it is a *reserved component* (see below), which is error E0208 when it appears outside its phrase.
4. The parser receives only canonical token kinds. It never sees which language was used.

Each keyword token records `lexeme` (exactly what the user typed) and `lang` (`en` or `hi`), used only by diagnostics and tooling.

**Reserved words.** Every word in the word table, and every component of every phrase, is reserved in every file. Components that are not keywords on their own are: `each`, `jab`, `tak`, `to`, `ek`, `kuch`.

**Case.** Keyword lookup is case-insensitive (`If`, `IF`, `Agar` are keywords). Identifiers are case-sensitive.

**Single source of truth.** The tables below exist in exactly one place in the implementation, `agam-lang/src/agamlang/keywords.py`. The lexer, the diagnostics ("did you mean") and the tests all read that module. Adding a language or a spelling is a data change there plus a row here.

### 3.2 Keyword table (proposal; becomes normative when the owner approves it, Q-07)

| Canonical token | English | Hinglish | Meaning |
|---|---|---|---|
| `KW_SHOW` | `show`, `print`, `say` | `dikhao`, `bolo`, `likho` | Print values |
| `KW_IF` | `if` | `agar` | Condition |
| `KW_ELIF` | `elif`, `else if` | `warna agar`, `varna agar` | Further condition |
| `KW_ELSE` | `else`, `otherwise` | `warna`, `varna`, `nahi to` | Fallback branch |
| `KW_WHILE` | `while` | `jabtak`, `jab tak` | Loop while true |
| `KW_REPEAT` | `repeat` | `dohrao` | Counted loop |
| `KW_TIMES` | `times` | `baar` | Counted-loop marker |
| `KW_FOR` | `for`, `for each` | `har`, `har ek` | Loop over a list |
| `KW_IN` | `in` | `mein` | List marker in a for loop |
| `KW_DEFINE` | `define`, `function` | `kaam` | Define a function |
| `KW_RETURN` | `return` | `wapas`, `lautao` | Return from a function |
| `KW_STOP` | `stop`, `break` | `ruko`, `bas` | Leave the nearest loop |
| `KW_SKIP` | `skip`, `continue` | `agla`, `chhodo` | Next iteration of the nearest loop |
| `KW_AND` | `and` | `aur` | Logical and |
| `KW_OR` | `or` | `ya` | Logical or |
| `KW_NOT` | `not` | `nahi`, `nahin` | Logical not |
| `KW_TRUE` | `true` | `sach` | Boolean true |
| `KW_FALSE` | `false` | `jhooth`, `galat` | Boolean false |
| `KW_NOTHING` | `nothing`, `none` | `khaali`, `kuch nahi` | The "no value" value |

Deliberately left out because they are common variable names in English code: `bar` (for `baar`) and `me` (for `mein`).

Phrase resolution examples: `nahi to` → `KW_ELSE`, while `nahi x` → `KW_NOT IDENT`; `kuch nahi` → `KW_NOTHING`; `else if` → `KW_ELIF`, while `else:` → `KW_ELSE`.

### 3.3 Built-in function names

Built-ins are ordinary names in the global scope, not keywords. Each is bound under an English and a Hinglish name that refer to the same function (§7.6). Users may shadow them; doing so is allowed in v0.1.

### 3.4 Example (one file, three styles)

```
# English
name = ask("Your name? ")
if name == "":
    show "Hello, friend!"
else:
    show "Hello,", name

# Hinglish
n = 0
jab tak n < 3:
    dikhao "Ginti:", n
    n = n + 1

# Mixed
kaam double(x):
    return x * 2

repeat 2 baar:
    dikhao double(21)
```

---

## 4. Lexical structure (normative)

### 4.1 Token kinds

| Group | Kinds |
|---|---|
| Structure | `NEWLINE`, `INDENT`, `DEDENT`, `EOF` |
| Literals and names | `NUMBER`, `TEXT`, `IDENT` |
| Keywords | the 19 `KW_*` kinds in §3.2 |
| Operators | `PLUS +`, `MINUS -`, `STAR *`, `SLASH /`, `DSLASH //`, `PERCENT %`, `DSTAR **`, `EQ ==`, `NE !=`, `LT <`, `LE <=`, `GT >`, `GE >=`, `ASSIGN =`, `PLUS_ASSIGN +=`, `MINUS_ASSIGN -=`, `STAR_ASSIGN *=`, `SLASH_ASSIGN /=` |
| Punctuation | `LPAREN (`, `RPAREN )`, `LBRACKET [`, `RBRACKET ]`, `COMMA ,`, `COLON :` |

A token is the record `Token(kind, lexeme, value, span, lang)`. `span` is `(line, column, end_line, end_column)`, 1-based, columns counted in Unicode code points, end exclusive. `value` is the parsed number or the unescaped text for literals, otherwise `None`. `lang` is `en`/`hi` for keywords, otherwise `None`.

### 4.2 Lines and indentation

- Blocks are delimited by indentation only (O-05). There is no block-terminator keyword: `end` and `khatam` are ordinary names, not keywords.
- A logical line ends with `NEWLINE`. Inside `( )` or `[ ]`, line breaks and indentation are ignored (implicit continuation). There is no backslash continuation.
- Indentation is the run of spaces and tabs at the start of a logical line. The lexer keeps a stack of indentation strings, starting with the empty string.
  - Same as the top of the stack: no token.
  - Longer, and the top of the stack is a prefix of it: push, emit `INDENT`.
  - Shorter, and equal to an earlier stack entry: pop to it, emit one `DEDENT` per pop.
  - Shorter but equal to no stack entry: E0104.
  - Neither is a prefix of the other (tabs and spaces mixed inconsistently): E0103.
- At end of file a `NEWLINE` is emitted if the last line lacks one, then one `DEDENT` per remaining stack level, then `EOF`.
- An `INDENT` that the grammar does not allow is reported by the parser as E0203.

### 4.3 Identifiers

A word is a maximal sequence of characters `c` such that the whole sequence satisfies Python's `str.isidentifier()` (letters of any script, digits, underscore; not starting with a digit). Devanagari and other non-Latin identifiers are therefore allowed.

### 4.4 Numbers

- Whole number: one or more ASCII digits (`0`, `42`, `007` — leading zeros allowed, value decimal).
- Decimal number: digits, a dot, digits (`3.14`); optional exponent `e`/`E`, optional sign, digits (`1.5e3`).
- A number immediately followed by an identifier character (`12abc`) is E0105. `.5` and `5.` are E0105 with a hint to write `0.5` / `5.0`.
- There is no negative literal; `-5` is unary minus applied to `5`.

### 4.5 Text

- Delimited by `"` or `'`; must end on the same line (else E0102).
- Escapes: `\n`, `\t`, `\\`, `\"`, `\'`. Any other backslash sequence is E0106.

### 4.6 Other characters

Any character that cannot start a token is E0101. Common look-alikes get a specific hint: curly quotes (`“ ” ‘ ’`) → "use straight quotes"; `;` → "one statement per line"; `{ }` → "blocks use a colon and indentation".

---

## 5. Grammar (normative, EBNF over canonical tokens)

```
program        = { statement } EOF ;

statement      = simple_stmt NEWLINE
               | if_stmt | while_stmt | repeat_stmt | for_stmt | func_def ;

simple_stmt    = show_stmt | return_stmt | KW_STOP | KW_SKIP
               | assign_or_expr ;

show_stmt      = KW_SHOW [ expression { COMMA expression } ] ;
return_stmt    = KW_RETURN [ expression ] ;

assign_or_expr = expression [ assign_op expression ] ;
assign_op      = ASSIGN | PLUS_ASSIGN | MINUS_ASSIGN | STAR_ASSIGN | SLASH_ASSIGN ;
                 (* the left side must be IDENT or an index expression, else E0212 *)

if_stmt        = KW_IF expression COLON block
                 { KW_ELIF expression COLON block }
                 [ KW_ELSE COLON block ] ;
while_stmt     = KW_WHILE expression COLON block ;
repeat_stmt    = KW_REPEAT expression KW_TIMES COLON block ;
for_stmt       = KW_FOR IDENT KW_IN expression COLON block ;
func_def       = KW_DEFINE IDENT LPAREN [ IDENT { COMMA IDENT } ] RPAREN COLON block ;

block          = NEWLINE INDENT statement { statement } DEDENT ;

expression     = or_expr ;
or_expr        = and_expr { KW_OR and_expr } ;
and_expr       = not_expr { KW_AND not_expr } ;
not_expr       = KW_NOT not_expr | comparison ;
comparison     = sum [ comp_op sum ] ;
comp_op        = EQ | NE | LT | LE | GT | GE ;
sum            = term { ( PLUS | MINUS ) term } ;
term           = unary { ( STAR | SLASH | DSLASH | PERCENT ) unary } ;
unary          = MINUS unary | power ;
power          = postfix [ DSTAR unary ] ;
postfix        = primary { LPAREN [ expression { COMMA expression } ] RPAREN
                         | LBRACKET expression RBRACKET } ;
primary        = NUMBER | TEXT | KW_TRUE | KW_FALSE | KW_NOTHING | IDENT
               | LPAREN expression RPAREN
               | LBRACKET [ expression { COMMA expression } [ COMMA ] ] RBRACKET ;
```

Notes:

- Comparisons do not chain. `1 < x < 10` is E0207, with the hint to write `1 < x and x < 10`.
- `**` is right-associative and binds tighter than unary minus on its left: `-2 ** 2` is `-(2 ** 2)`.
- A block needs at least one statement. A block that should do nothing can contain the expression statement `nothing`.
- `KW_RETURN` outside a function is E0209. `KW_STOP`/`KW_SKIP` outside a loop is E0210. Both are detected by the parser.
- Function definitions are allowed at any nesting level.

Parser: hand-written recursive descent, one token of look-ahead, stops at the first error (P-21, P-29).

---

## 6. Abstract syntax tree (normative)

Nodes are immutable records. Every node has a `span`. Keyword spelling and language do not appear in the tree.

| Statements | Fields |
|---|---|
| `Program` | `body: [stmt]` |
| `Show` | `values: [expr]` |
| `Assign` | `target: Name \| Index`, `op: "=" \| "+=" \| "-=" \| "*=" \| "/="`, `value: expr` |
| `ExprStmt` | `value: expr` |
| `If` | `branches: [(condition: expr, body: [stmt])]`, `else_body: [stmt] \| None` |
| `While` | `condition: expr`, `body: [stmt]` |
| `Repeat` | `count: expr`, `body: [stmt]` |
| `ForEach` | `variable: str`, `iterable: expr`, `body: [stmt]` |
| `FunctionDef` | `name: str`, `params: [str]`, `body: [stmt]` |
| `Return` | `value: expr \| None` |
| `Stop`, `Skip` | — |

| Expressions | Fields |
|---|---|
| `NumberLit` | `value: int \| float` |
| `TextLit` | `value: str` |
| `BoolLit` | `value: bool` |
| `NothingLit` | — |
| `ListLit` | `items: [expr]` |
| `Name` | `id: str` |
| `Unary` | `op: "-" \| "not"`, `operand: expr` |
| `Binary` | `op: "+" "-" "*" "/" "//" "%" "**"`, `left`, `right` |
| `Compare` | `op: "==" "!=" "<" "<=" ">" ">="`, `left`, `right` |
| `Logical` | `op: "and" \| "or"`, `left`, `right` |
| `Call` | `callee: expr`, `args: [expr]` |
| `Index` | `target: expr`, `index: expr` |

**JSON form** (used by `agam ast` and by tests): each node is an object with `"type"` set to the node name and one key per field; lists become arrays; `None` becomes `null`. Spans are omitted unless `--spans` is given, in which case each node also has `"span": [line, column, end_line, end_column]`. Keys are emitted in the field order above, with two-space indentation.

**Fusion equivalence property (normative):** two programs that differ only in which alias is used for each keyword (and in the names of built-ins) produce identical canonical token-kind sequences and, apart from built-in names and spans, identical ASTs and identical output.

---

## 7. Semantics (normative)

### 7.1 Values

| Type name | Examples | Host representation |
|---|---|---|
| `number` | `42`, `3.14` | Python `int` (unbounded) or `float` |
| `text` | `"hi"` | Python `str` |
| `boolean` | `true`, `false` | Python `bool` |
| `nothing` | `nothing` | Python `None` |
| `list` | `[1, "a", [2]]` | Python `list` (mutable, shared by reference) |
| `function` | user-defined or built-in | internal object |

A boolean is never treated as a number: the interpreter MUST test for `bool` before `int`.

### 7.2 Variables and scope

- Assignment creates the variable if it does not exist in the current scope, otherwise rebinds it.
- Scopes: one global scope; each function call creates a local scope whose parent is the scope where the function was **defined** (lexical scoping; closures work).
- Reading a name searches the current scope, then outward; not found is E0301 with a "did you mean" suggestion.
- Assigning inside a function always targets the function's local scope; a function cannot rebind an outer variable in v0.1 (it can still modify an outer list in place).
- The loop variable of a for loop is an ordinary variable in the current scope and keeps its last value after the loop.
- Compound assignment `x += e` means `x = x + e`, with `x` required to exist.

### 7.3 Operators

| Operator | Operand types | Result |
|---|---|---|
| `+` | number, number | number |
| `+` | text, text | concatenated text |
| `+` | list, list | new list |
| `-`, `*` | number, number | number |
| `/` | number, number | always a decimal (`float`); divisor zero is E0401 |
| `//`, `%` | number, number | floor division and remainder with Python semantics; divisor zero is E0401 |
| `**` | number, number | number; results follow Python (a negative exponent gives a decimal) |
| unary `-` | number | number |
| `==`, `!=` | any, any | boolean; values of different types are unequal, except that whole and decimal numbers compare by value; lists compare element-wise |
| `<`, `<=`, `>`, `>=` | number with number, or text with text | boolean (text compares by code point) |
| `and`, `or` | boolean, boolean | boolean; the right side is evaluated only if needed |
| `not` | boolean | boolean |

Any other combination is E0302. The message names both types and gives a targeted tip; for `text + number` the tip is to write `show "Age:", age` or to convert with `text(age)`.

### 7.4 Statements

- **`show a, b, c`** writes each value's display form separated by one space, then a newline. `show` alone writes an empty line. Display forms: whole numbers in decimal; decimals as Python's `repr` (`2.0`, `0.1`); text as-is (no quotes); `true`/`false`; `nothing`; lists as `[1, "a", true]` with text items in double quotes; functions as `<function name>`.
- **`if` / `while`**: the condition MUST be a boolean, else E0302 with the tip "use a comparison such as `x > 0`".
- **`repeat n times:`**: `n` is evaluated once; it MUST be a whole number ≥ 0, else E0404.
- **`for x in items:`**: `items` MUST be a list (E0305) or text (iterates characters). Iteration is over a snapshot taken when the loop starts.
- **`stop`** leaves the nearest enclosing loop; **`skip`** jumps to its next iteration.
- **Functions**: parameters are positional; the argument count must match exactly (E0304). A function that ends without `return`, or executes `return` with no value, returns `nothing`. Functions are values and can be passed and stored.
- An expression statement evaluates its expression and discards the result.

### 7.5 Limits

- Call depth limit: 200 nested AgamLang calls, then E0402. The implementation MUST raise Python's recursion limit enough that this limit, not a Python `RecursionError`, is reached first; any `RecursionError` is still caught and reported as E0402.
- Optional step limit (`--max-steps N`): every statement executed and every loop iteration counts one step; exceeding it is E0403. Default: unlimited.

### 7.6 Built-in functions

| English | Hinglish | Behaviour |
|---|---|---|
| `ask(prompt)` | `pucho(prompt)` | Writes the prompt (no newline), reads one line from standard input, returns it as text without the line ending. End of input is E0405. With no argument the prompt is empty. |
| `number(x)` | `sankhya(x)` | Text → whole number if it looks like one, else decimal; surrounding spaces ignored; a number is returned unchanged; anything else is E0307. |
| `text(x)` | `shabd(x)` | The display form of `x` as text. |
| `length(x)` | `lambai(x)` | Number of items in a list or characters in text; otherwise E0302. |
| `append(list, item)` | `jodo(list, item)` | Adds `item` to the end of `list` in place; returns `nothing`. |
| `range(stop)`, `range(start, stop)` | `ginti(...)` | A new list of whole numbers from `start` (default 0) up to but not including `stop`. Arguments must be whole numbers. |
| `type(x)` | `prakar(x)` | One of the texts `number`, `text`, `boolean`, `nothing`, `list`, `function`. |

List indexing is 0-based (O-06): `items[i]` with a whole-number index from `0` to `length − 1`; anything else is E0306 (the message states the valid range and reminds that counting starts at 0). Negative indexes are not supported in v0.1. Text can be indexed the same way. `items[i] = v` replaces an element; text is immutable (E0302).

---

## 8. Diagnostics (normative; P-29)

### 8.1 Model

```
Diagnostic(code, span, args)        # args: named values for the message template
```

One diagnostic ends the run. Lexer, parser and interpreter raise a single internal exception type carrying a `Diagnostic`; the command line catches it and renders it. **No Python traceback is ever shown to the user**; an unexpected internal exception is reported as E0900 ("this is a bug in AgamLang, not in your program") with exit status 70.

### 8.2 Rendering

English:

```
Error E0201 in hello.agam, line 3:

    agar x > 5
              ^
I expected a colon ":" at the end of this line.
Tip: a line that starts with "agar" must end with ":".
```

Hinglish:

```
Galti E0201, hello.agam, line 3:

    agar x > 5
              ^
Is line ke end mein ":" hona chahiye.
Sujhav: "agar" se shuru hone wali line ":" par khatam hoti hai.
```

Rules:

- Header, blank line, the source line indented by four spaces, a caret line marking the span (one `^` per column, at least one), the message, then an optional tip line.
- Messages refer to keywords **as the user typed them** (the token's `lexeme`), never by canonical token names.
- Messages avoid jargon: say "text" not "string", "whole number" not "integer", "name" not "identifier".
- Message language: `--messages en|hi|auto` (default `auto`). `auto` uses Hinglish when the file has more `hi` keyword tokens than `en` ones, otherwise English.
- Catalogues live in `agam-lang/src/agamlang/messages/en.py` and `hi.py`; both MUST define every code (a test enforces this).
- Diagnostics go to standard error; program output goes to standard output.

### 8.3 "Did you mean"

For E0301 (unknown name) and E0204 (unexpected word), candidates are all names in scope plus all keyword aliases. A suggestion is given if the best candidate is within edit distance 2 (distance 1 for words shorter than 5 characters). Example: `dikhoa "hi"` → tip `Did you mean "dikhao"?`.

### 8.4 Error catalogue

| Code | Stage | Condition |
|---|---|---|
| E0100 | lex | File is not valid UTF-8 |
| E0101 | lex | Character cannot be used here (with look-alike hints, §4.6) |
| E0102 | lex | Text literal not closed before end of line |
| E0103 | lex | Tabs and spaces mixed inconsistently in indentation |
| E0104 | lex | Indentation does not match any earlier level |
| E0105 | lex | Malformed number |
| E0106 | lex | Unknown escape sequence in text |
| E0201 | parse | Missing `:` after `if`/`elif`/`else`/`while`/`repeat`/`for`/`define` header |
| E0202 | parse | Expected an indented block after `:` |
| E0203 | parse | Unexpected indentation |
| E0204 | parse | Unexpected token (general fallback; says what was expected) |
| E0205 | parse | Bracket `(` or `[` never closed, or closed with the wrong bracket |
| E0206 | parse | `=` where a comparison was expected (tip: use `==`) |
| E0207 | parse | Chained comparison |
| E0208 | parse | Reserved word used as a name |
| E0209 | parse | `return` outside a function |
| E0210 | parse | `stop`/`skip` outside a loop |
| E0211 | parse | `repeat` without `times` |
| E0212 | parse | Left side of an assignment cannot be assigned to |
| E0301 | run | Unknown name |
| E0302 | run | Operation not supported for these types |
| E0303 | run | Value is not a function but was called |
| E0304 | run | Wrong number of arguments |
| E0305 | run | Value is not a list where one is required |
| E0306 | run | Index out of range or not a whole number |
| E0307 | run | Cannot convert to a number |
| E0401 | run | Division by zero |
| E0402 | run | Function calls nested too deeply |
| E0403 | run | Step limit exceeded |
| E0404 | run | Repeat count is not a whole number ≥ 0 |
| E0405 | run | Input ended while `ask` was waiting |
| E0900 | internal | Interpreter bug |

Codes are stable: a code is never reused for a different condition.

---

## 9. Command line (normative)

Invocation: `python -m agamlang <command> [options] <file>` (an `agam` console script is provided by the package as an alias).

| Command | Behaviour |
|---|---|
| `run <file>` | Execute the program |
| `check <file>` | Lex and parse only; print nothing on success |
| `tokens <file>` | Print the token stream, one token per line: `<line>:<column> <KIND> <lexeme as JSON string>` followed by ` lang=<en\|hi>` for keywords |
| `tokens --canonical <file>` | One token per line: `<KIND>` only, plus ` <value as JSON>` for `NUMBER`, `TEXT`, `IDENT` |
| `ast [--spans] <file>` | Print the AST as JSON (§6) |
| `--version` | Print `agamlang <version>` |

Options for `run`: `--messages en|hi|auto`, `--max-steps N`.

| Exit status | Meaning |
|---|---|
| 0 | Success |
| 1 | The program has an error (any E01xx–E04xx) |
| 2 | Command-line usage error, or the file cannot be read |
| 70 | Internal error (E0900) |

---

## 10. Implementation structure (P-20, P-21)

```
agam-lang/
├── pyproject.toml              name "agamlang", requires-python ">=3.11", no runtime dependencies
└── src/agamlang/
    ├── __init__.py             __version__
    ├── __main__.py  cli.py     command line (§9)
    ├── source.py               SourceFile (text, line table), Span
    ├── tokens.py               TokenKind, Token
    ├── keywords.py             alias tables (§3.2) — single source of truth
    ├── lexer.py                raw tokens, indentation, keyword normaliser
    ├── nodes.py                AST records (§6) and JSON serialisation
    ├── parser.py               recursive descent (§5)
    ├── values.py               runtime values, display forms, type names
    ├── stdlib.py               built-in functions (§7.6)
    ├── interpreter.py          evaluator, scopes, limits
    ├── diagnostics.py          Diagnostic, renderer, suggestions
    └── messages/en.py, hi.py   message catalogues
```

Module dependency direction (no cycles): `source` → `tokens` → `keywords` → `lexer` → `nodes` → `parser` → `values` → `stdlib` → `interpreter` → `cli`; `diagnostics` and `messages` may be imported by any stage.

Public Python API (stable for tests):

```python
agamlang.lexer.tokenize(source: SourceFile) -> list[Token]
agamlang.parser.parse(tokens: list[Token], source: SourceFile) -> Program
agamlang.interpreter.run(program: Program, *, stdin, stdout, max_steps=None) -> None
agamlang.diagnostics.AgamError            # exception carrying .diagnostic
agamlang.diagnostics.render(diagnostic, source, messages="auto") -> str
```

Control flow inside the interpreter (`return`, `stop`, `skip`) uses private Python exceptions that never escape `run`.

---

## 11. Future directions (not part of v0.1)

- v0.2 candidates: text interpolation (`"Hello {name}"`), dictionaries, natural-order Hinglish forms (Q-07), a REPL, `agam translate --to en|hi` (re-emit a program in one language from its AST), Devanagari keyword aliases (Q-09).
- Later: bytecode compiler and VM, then native code generation; an AOS port once AOS has user mode and a C runtime. The canonical token stream and AST are the stable interfaces intended to survive those changes.

---

## 12. Known limitations and risks specific to the language design

| Topic | Limitation |
|---|---|
| Hinglish word order | v0.1 is keyword-first; some lines read less naturally than spoken Hindi (for example `har fal mein fruits:`). Mixed style (`har fal in fruits:`) is always available. |
| Reserved words | The union of all aliases is reserved, so common English words such as `to`, `each`, `say`, `skip` cannot be variable names. E0208 explains this. |
| Romanisation variety | People spell Hinglish differently. Only listed spellings are keywords; near-misses are caught by "did you mean". |
| Indentation | The most common beginner error class; E0103, E0104, E0202 and E0203 messages must be especially clear. |
