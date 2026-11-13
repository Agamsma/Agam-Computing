# AgamLang Specification — v0.1

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 4 — NOT APPROVED.** Corrects the points the second independent review left open: AR-11/AR-29 (literal limits), AR-30 (traversal budgets), AR-14/AR-31 (diagnostic and API contracts), AR-32 (`DECISIONS.md` §10). All are *addressed in draft — pending independent re-review*. No interpreter code may be written until the owner opens gate G-LANG. |
| Date | 2026-11-12 (revision 4) |
| Location | Implementation in `agam-lang/` (Claude); tests in `agam-lang/tests/` (Codex) |
| Owner decisions applied | Fusion default (O-07); indentation blocks (O-05); 0-based lists (O-06); cycles rejected (O-13); provisional limits (O-14); Python-compatible evaluation order (O-15); mixed else-if phrases (O-16) |
| **Not approved** | All Hinglish keywords and all Hinglish message wording are **proposals pending the owner's review** (O-20, B-06, Q-07). Q-08 and Q-09 are open. |
| Out of scope (v0.1) | Dictionaries, string interpolation, modules, classes, file and network I/O, REPL, native compilation, running on AOS |

Sections marked *normative* define behaviour that independent tests depend on. Section 10 says which implementation details are deliberately **not** normative. Nothing here has been implemented or run.

The choices that revision 4 first drafted as provisional — the stages of the literal limits, the display and comparison limits, and the treatment of deep source nesting — were decided by the owner on 2026-11-13 (`DECISIONS.md` §3.4: O-24, O-25, O-28). One choice remains **provisional**: the floor for long operator sequences in §7.10 (`DECISIONS.md` §11, Q-24).

---

## 1. Design goals

1. **A first program reads like a sentence**: `show "Hello"` / `dikhao "Namaste"`.
2. **Fusion is the default and only mode** (C-08, O-07): English, Hinglish (Hindi in Latin letters) and any mixture are valid in the same `.agam` file, on the same line, and inside the same two-word phrase. No flag, setting or file header selects a language. There is one lexer, one grammar, one AST and one interpreter.
3. **Errors teach.** Every error says what went wrong, where, and what to try, in plain English or Hinglish.
4. **Python-compatible habits** (O-05, O-06, O-15): indentation, colons, operators, 0-based lists and evaluation order behave as in Python wherever this document does not say otherwise.
5. **Closed and fully specified.** Every construct has defined behaviour, including every error. No operation can produce a value outside §7.1, and no user program can cause a host traceback.

---

## 2. Source text (normative)

- File extension `.agam`; encoding UTF-8 (a leading byte-order mark is ignored). Invalid UTF-8 is E0100.
- Line endings `\n` and `\r\n` are equivalent.
- A comment starts with `#` outside a text literal and runs to the end of the line.
- Blank lines and comment-only lines are ignored, including for indentation.
- A file is a sequence of statements. There is no `main`.
- Source text is not Unicode-normalised, except identifiers (§4.3).
- **Lines.** The text is split into lines at each line ending. A line ending at the very end of the text does not start a further, empty line. Line and column numbers are 1-based; columns count Unicode code points; a leading byte-order mark is not a column.

---

## 3. Fusion keyword system (P-22, P-23, P-24)

### 3.1 Mechanism (normative)

1. The lexer splits each physical line into raw tokens. A **word** is an identifier-shaped token (§4.3).
2. The **keyword normaliser**, which is part of the lexer, processes the words of each physical line from left to right. A word is looked up by its **ASCII-lower-cased** spelling; words containing non-ASCII characters never match. At each word it tries, in order:
   a. a **two-word phrase** (§3.2): this word and the next raw token, if that token is a word on the same physical line and only spaces or tabs lie between them;
   b. a **single word** (§3.2).
   The first match wins. A match becomes **one** keyword token.
3. A word that matches nothing but is a **reserved component** (§3.2) becomes a `RESERVED_WORD` token. The lexer never raises an error for it; the parser reports E0208 (§5.2).
4. Any other word becomes `IDENT`.
5. **Else-if composition** (§3.3) is then applied.
6. The parser receives only token kinds, values and spans. It MUST NOT read a keyword's spelling or language.

Phrases never join across a line break, a comment or any punctuation, including inside brackets.

**Reserved words.** Every single word, every phrase component and every reserved component is reserved in every file, whatever languages the file uses.

**Case.** Keywords are ASCII case-insensitive (`If`, `IF`, `aGaR`). Identifiers are case-sensitive.

**Two tables, deliberately.** The implementation has exactly one runtime alias table, `agam-lang/src/agamlang/keywords.py`, read by the lexer and by diagnostics. **Tests do not read it to learn what is expected.** Codex hand-writes an independent fixture from §3.2 (`agam-lang/tests/fixtures/keywords_expected.json`) and asserts that the runtime table equals it exactly: same entries, kinds, languages, phrases and reserved components (AR-08).

### 3.2 Keyword tables (normative in structure; **Hinglish column is a proposal pending owner review**)

**Single words — 52 entries**

| Canonical token | English (`en`) | Hinglish (`hi`) |
|---|---|---|
| `KW_SHOW` | `show`, `print`, `say` | `dikhao`, `bolo`, `likho` |
| `KW_IF` | `if` | `agar` |
| `KW_ELIF` | `elif` | — |
| `KW_ELSE` | `else`, `otherwise` | `warna`, `varna` |
| `KW_WHILE` | `while` | `jabtak` |
| `KW_REPEAT` | `repeat` | `dohrao` |
| `KW_TIMES` | `times` | `baar` |
| `KW_FOR` | `for` | `har` |
| `KW_IN` | `in` | `mein` |
| `KW_DEFINE` | `define`, `function` | `kaam` |
| `KW_RETURN` | `return` | `wapas`, `lautao` |
| `KW_STOP` | `stop`, `break` | `ruko`, `bas` |
| `KW_SKIP` | `skip`, `continue` | `agla`, `chhodo` |
| `KW_AND` | `and` | `aur` |
| `KW_OR` | `or` | `ya` |
| `KW_NOT` | `not` | `nahi`, `nahin` |
| `KW_TRUE` | `true` | `sach` |
| `KW_FALSE` | `false` | `jhooth`, `galat` |
| `KW_NOTHING` | `nothing`, `none` | `khaali` |

**Two-word phrases — 5 entries**

| Phrase | Canonical token | Component languages |
|---|---|---|
| `for each` | `KW_FOR` | `en`, `en` |
| `har ek` | `KW_FOR` | `hi`, `hi` |
| `jab tak` | `KW_WHILE` | `hi`, `hi` |
| `nahi to` | `KW_ELSE` | `hi`, `hi` |
| `kuch nahi` | `KW_NOTHING` | `hi`, `hi` |

**Reserved components — 6 entries** (words that are reserved but are not keywords on their own): `each` (`en`), `ek`, `jab`, `tak`, `to`, `kuch` (`hi`).

Deliberately not aliases, because they are common names in English code: `bar`, `me`. Implementations MUST NOT add aliases that are not in these tables.

### 3.3 Else-if composition (normative; O-16)

After step 2 of §3.1, wherever a `KW_ELSE` token is immediately followed by a `KW_IF` token on the same physical line with only spaces or tabs between them, the normaliser replaces the two tokens with **one** `KW_ELIF` token.

Consequently every combination of an else-alias and an if-alias is an else-if, including mixed-language ones:

| Source | Tokens after composition |
|---|---|
| `else if`, `otherwise if` | `KW_ELIF` |
| `warna agar`, `varna agar` | `KW_ELIF` |
| `warna if`, `else agar` | `KW_ELIF` (mixed) |
| `nahi to agar`, `nahi to if` | `KW_ELIF` (phrase plus word) |
| `else:` then `if` on the next line | `KW_ELSE`, …, `KW_IF` (not composed) |

The grammar never allows `KW_ELSE` directly followed by `KW_IF`, so composition cannot change the meaning of a valid program.

### 3.4 Keyword token metadata (normative)

Every keyword token and `RESERVED_WORD` token records:

| Field | Content |
|---|---|
| `lexeme` | The exact source text from the first character of its first word to the last character of its last word, including interior spaces or tabs exactly as typed |
| `span` | From the start of the first word to the end of the last word |
| `parts` | One entry per component word, in order: the word as typed, its language, and its own span |
| `lang` | `en` if every part is `en`; `hi` if every part is `hi`; otherwise `mixed` |

Examples: `Agar` → lexeme `Agar`, `lang=hi`, one part. `nahi  to agar` (two spaces) → one `KW_ELIF`, lexeme `nahi  to agar`, parts `nahi`/`hi`, `to`/`hi`, `agar`/`hi`, `lang=hi`. `warna if` → `KW_ELIF`, parts `warna`/`hi`, `if`/`en`, `lang=mixed`.

**Language counts** used by diagnostics (§8.5) count parts, not tokens: `warna if` adds one to `hi` and one to `en`.

This metadata is used only by diagnostics and tooling. It never affects parsing or execution.

### 3.5 Built-in function names

Built-ins are ordinary names in the global scope, not keywords. Each is bound under an English and a Hinglish name that refer to **the same function value** (§7.11). Users may shadow them.

### 3.6 Example (one file, three styles)

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

# Mixed, including a mixed else-if
kaam grade(marks):
    agar marks >= 90:
        return "A"
    warna if marks >= 60:
        wapas "B"
    else:
        return "C"

repeat 2 baar:
    dikhao grade(75)
```

---

## 4. Lexical structure (normative)

### 4.1 Token kinds

| Group | Kinds |
|---|---|
| Structure | `NEWLINE`, `INDENT`, `DEDENT`, `EOF` |
| Literals and names | `NUMBER`, `TEXT`, `IDENT` |
| Keywords | the 19 `KW_*` kinds of §3.2, and `RESERVED_WORD` |
| Operators | `PLUS +`, `MINUS -`, `STAR *`, `SLASH /`, `DSLASH //`, `PERCENT %`, `DSTAR **`, `EQ ==`, `NE !=`, `LT <`, `LE <=`, `GT >`, `GE >=`, `ASSIGN =`, `PLUS_ASSIGN +=`, `MINUS_ASSIGN -=`, `STAR_ASSIGN *=`, `SLASH_ASSIGN /=` |
| Punctuation | `LPAREN (`, `RPAREN )`, `LBRACKET [`, `RBRACKET ]`, `COMMA ,`, `COLON :` |

A token has `kind`, `lexeme`, `value`, `span`, and for keyword tokens the fields of §3.4.

- `span` is `(line, column, end_line, end_column)`, 1-based, columns counted in Unicode code points of the original line, end exclusive.
- `value`: the parsed number for `NUMBER`; the unescaped text for `TEXT`; **the identifier's normalised spelling (§4.3) for `IDENT`**; nothing for other kinds.

### 4.2 Lines and indentation

- Blocks are delimited by indentation only (O-05). `end` and `khatam` are ordinary names.
- A logical line ends with `NEWLINE`. Inside `( )` or `[ ]`, line breaks and indentation are ignored. There is no backslash continuation.
- Indentation is the run of spaces and tabs at the start of a logical line. The lexer keeps a stack of indentation strings, starting with the empty string:
  - equal to the top: no token;
  - longer, with the top as a prefix: push, emit `INDENT`;
  - shorter and equal to an earlier entry: pop to it, one `DEDENT` per pop;
  - shorter and equal to no entry: E0104;
  - neither string is a prefix of the other: E0103.
- Spaces and tabs may both be used as long as these rules hold. Example: a block indented with one tab may contain a block indented with one tab followed by four spaces; a sibling line indented with eight spaces is E0103.
- At end of file: a `NEWLINE` if the last line lacks one, then one `DEDENT` per remaining level, then `EOF`.

### 4.3 Identifiers

A word is a maximal run of characters that, as a whole, satisfies Python's `str.isidentifier()` (letters of any script, digits, underscore; not starting with a digit). Two identifiers are the same name if their NFKC-normalised spellings are equal, as in Python; the `IDENT` token's `value` is that normalised spelling and its `lexeme` is the text as typed. The set of letters is that of the host Python's Unicode version.

### 4.4 Numbers

```
whole    = digit { digit }
decimal  = digit { digit } "." digit { digit } [ exponent ]
         | digit { digit } exponent
exponent = ( "e" | "E" ) [ "+" | "-" ] digit { digit }
```

- Digits are ASCII `0`–`9` only. Leading zeros are allowed (`007` is 7). `1e3` is the decimal 1000.0.
- There is no negative literal; `-5` is unary minus applied to `5`.
- `.5` and `5.` are E0105 (tips: write `0.5` / `5.0`). A number immediately followed by an identifier character (`12abc`) is E0105.
- **Limits (O-14), checked before conversion:** a whole literal with more than 1,000 significant digits (after removing leading zeros) is E0107. A decimal literal whose lexeme is longer than 1,000 characters, or whose value is not a finite decimal, is E0107. A decimal literal too small to represent becomes `0.0`.

### 4.5 Text

- Delimited by `"` or `'`; the literal ends at the next unescaped occurrence of its opening quote on the same line.
- Escapes: `\n`, `\t`, `\\`, `\"`, `\'`.
- **Limit (O-14; stage decided by O-24):** the decoded value may have at most `MAX_ITEMS` (1,000,000) code points.
- The lexer scans a literal from left to right and reports **the first problem it reaches**:
  - a backslash followed by any other character on the same line: E0106;
  - the 1,000,001st decoded character: E0108, decided while scanning and before the value is built;
  - the end of the line or of the file, including a backslash that is the last character of the line: E0102.

### 4.6 Other characters

A character that cannot start a token is E0101, with a specific tip for curly quotes, `;`, `{` and `}` (Appendix A).

---

## 5. Grammar (normative)

### 5.1 EBNF over canonical tokens

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

### 5.2 Parser rules

- Hand-written recursive descent, one token of look-ahead, stops at the first error.
- `**` is right-associative and binds tighter than a unary minus on its left: `-2 ** 2` is `-(2 ** 2)`.
- Comparisons do not chain: `1 < x < 10` is E0207.
- A trailing comma is allowed only in a list literal, not in call arguments or parameters.
- A block needs at least one statement; the expression statement `nothing` is a valid no-op.
- The left side of an assignment must be `IDENT` or an index expression, else E0212.
- **Function boundaries.** `KW_RETURN` outside a function body is E0209. `KW_STOP`/`KW_SKIP` is valid only inside a loop **of the same function body** (or of the top level): the loop context is reset when a `func_def` body starts, so a `stop` inside a function nested in a loop is E0210.
- Duplicate parameter names (compared as normalised identifiers) are E0213.
- **Reserved words (E0208, parser only).** E0208 is reported when (a) a `RESERVED_WORD` token appears anywhere; (b) any keyword token appears where the grammar requires `IDENT` (function name, parameter, for-loop variable); or (c) a statement begins with a keyword token that is immediately followed by an assignment operator (`show = 3`, `if = 1`).
- **Error precedence** when several codes could apply at one token: E0214 (§5.3), E0208, E0206, E0207, E0211, E0201, E0205, E0212, E0213, E0209, E0210, E0202, E0203, and E0204 only as the fallback.

### 5.3 Syntactic nesting limit (normative; O-28)

**Measurement.** Two constructs nest:

- a **block** — the statements between one `INDENT` and its `DEDENT`. The header of a compound statement is not inside its own block, and the `else if` and `else` branches of one `if` are separate blocks side by side.
- a **bracket pair** — an opening `(` or `[` that the parser consumes, together with its matching closing bracket. There are five uses: a parenthesised expression, a list literal, a call's argument list, an index, and a function definition's parameter list. Both brackets belong to their pair.

The **level** of a token is the number of blocks that contain it plus the number of bracket pairs that contain it. Tokens of top-level statements outside any bracket are at level 0. **No token may have a level above `MAX_SOURCE_NESTING` (100).**

Nothing else adds a level: not a binary operator, a prefix operator, a comparison or `**`; not a longer chain of them (`a + b + c + …` stays at one level, however long); not calls or indexes that follow one another (`a[0][1][2]` is three pairs side by side, each at the same level).

**Detection and diagnostic.** The parser keeps one count: open blocks plus open bracket pairs. The check is made at exactly two moments, and each is decided before the parser examines anything further:

1. it consumes an opening bracket while the count is already 100 → **E0214**; the span is that bracket;
2. it enters a block while the count is already 100 → **E0214**; the span is the first token of the block's first statement.

E0214 has no variant and no arguments. Like every parse error it is the first error in token order that stops the parser, and lexical errors come before any parse error. It is first in the precedence list of §5.2: at the first token of a block it is reported in place of any other code for that token. A token that the grammar does not accept where it stands is reported with its own code and is never counted as an opening bracket.

**Boundaries.** A program exactly at the limit is valid: 100 nested bracket pairs, 100 nested blocks, or any mixture that adds up to 100. A statement inside 100 nested blocks cannot contain a bracket. 101 opening brackets that are never closed give E0214 at the 101st bracket, not E0205a, because the parser stops there. Worked boundaries are in Appendix C.11.

---

## 6. Abstract syntax tree (normative)

### 6.1 Nodes

Nodes are immutable records. Keyword spelling and language are not part of a node's value.

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
| `Name` | `id: str` (normalised spelling) |
| `Unary` | `op: "-" \| "not"`, `operand: expr` |
| `Binary` | `op`: one of `+ - * / // % **`; `left`, `right` |
| `Compare` | `op`: one of `== != < <= > >=`; `left`, `right` |
| `Logical` | `op: "and" \| "or"`, `left`, `right` |
| `Call` | `callee: expr`, `args: [expr]` |
| `Index` | `target: expr`, `index: expr` |

### 6.2 JSON form

Used by `agam ast` and by tests: each node is an object with `"type"` and one key per field, in the order above; lists become arrays; `None` becomes `null`; whole numbers are JSON integers and decimals are JSON numbers with a decimal point or exponent; two-space indentation. With `--spans`, each node also has, as its last key, `"span": [line, column, end_line, end_column]`, written on one line (Appendix C.10). Nothing else is ever emitted.

### 6.3 Source metadata (normative; AR-14, AR-31)

Every node carries, **outside** its value:

- `span` — the source range of the construct (§6.5);
- `origin` — for each keyword that formed the node, the keyword token's metadata of §3.4.

The `Program` node additionally carries `lang_counts` (§8.5).

`span`, `origin` and `lang_counts` are excluded from node equality and from the default JSON. They exist so that messages can quote the user's own word, point at the right place and choose a language. Nothing in the parser or interpreter may branch on them.

### 6.4 Fusion equivalence (normative; AR-10)

- **Keyword substitution.** Two programs that differ only in which alias, phrase or else-if form is used for each keyword produce identical `tokens --canonical` output, identical default AST JSON, identical standard output and identical exit status.
- **Built-in names are ordinary names.** Replacing `length` by `lambai` changes `IDENT` values and `Name` nodes, so token and AST identity is **not** required. Required instead: in a program that does not rebind either name, the two programs produce identical output and exit status, and `length == lambai` is `true` (§7.5).
- Identifiers are never canonicalised or translated.

### 6.5 Node spans (normative; used by `ast --spans` and by diagnostics)

- A node's span runs from the start of the **first token consumed by its grammar production** to the end of the **last token consumed by it**. `NEWLINE`, `INDENT`, `DEDENT` and `EOF` are never part of a span.
- Parentheses create no node: `( expression )` yields the inner node with its own span unchanged. A node whose production consumed the parentheses includes them. In `x = (1 + 2) * 3`, the `+` node spans `1 + 2` and the `*` node spans `(1 + 2) * 3`.
- Simple statements: first token to last token.
- Compound statements (`If`, `While`, `Repeat`, `ForEach`, `FunctionDef`): from the header keyword to the end of the last token of the last statement of the last block.
- `Call`: from the start of the callee to the closing `)`. `Index`: from the start of the target to the closing `]`. `ListLit`: `[` to `]`. `Unary`: operator to the end of the operand.
- `Program`: starts at line 1, column 1 and ends at the end of the last token; for a program with no tokens it is `[1, 1, 1, 1]`.
- Branch tuples of `If` and parameter names are not nodes and have no span.

A complete example is in Appendix C.9.

---

## 7. Semantics (normative)

### 7.1 Values and provisional limits

| Type name | Examples | Host representation (informative) |
|---|---|---|
| `number` — whole | `42` | Python `int` |
| `number` — decimal | `3.14` | Python `float`, always finite |
| `text` | `"hi"` | Python `str` |
| `boolean` | `true`, `false` | Python `bool` |
| `nothing` | `nothing` | Python `None` |
| `list` | `[1, "a", [2]]` | Python `list`; mutable; shared by reference; never cyclic |
| `function` | user-defined or built-in | internal object |

A boolean is never a number. There are no other values: no complex numbers, infinities or NaN.

**Limits (O-14, O-25, O-28).** Provisional v0.1 policy, not measured safety guarantees. All are owner decisions: the first three by O-14; `MAX_NESTING` and `MAX_COMPARE`, which bound display and comparison, by O-25; `MAX_SOURCE_NESTING` by O-28.

| Limit | Value | What it bounds |
|---|---|---|
| `MAX_WHOLE_DIGITS` | 1,000 | The magnitude of every whole number: at most 10^1000 − 1 |
| Decimal range | finite IEEE-754 double | Every decimal |
| `MAX_ITEMS` | 1,000,000 | The length of every list (elements) and of every text (code points) — text literals, computed text, display forms and input lines alike |
| `MAX_NESTING` | 100 | The **depth** of a list that is displayed or compared |
| `MAX_COMPARE` | 1,000,000 | The **expanded element count** of a list that is compared |
| `MAX_SOURCE_NESTING` | 100 | The **syntactic nesting level** of every token of a program: enclosing blocks plus enclosing bracket pairs (§5.3) |

**Depth and expanded element count are properties of a value, not of an algorithm:**

- `depth(v)` is 0 if `v` is not a list. For a list it is 1 plus the largest depth of its elements (1 for an empty list). So `[]` and `[1, 2]` have depth 1, and `[[1]]` has depth 2.
- `count(v)` is 0 if `v` is not a list. For a list it is the sum, over its elements `e`, of `1 + count(e)`. An element that is reachable several times is counted **once per occurrence**. So `[1, 2, 3]` has count 3 and `[[1, 2], [3]]` has count 5.

Lists are never cyclic (§7.8), so both are finite. An implementation may compute them in any way — stopping early once a limit is known to be exceeded, or sharing work between shared sub-lists — but the outcome MUST be the one these definitions give.

**Boundaries and codes.** A value exactly at a limit is valid; one step beyond is an error:

| Situation | Code |
|---|---|
| Numeric literal beyond its limit (§4.4) | E0107 |
| Text literal longer than `MAX_ITEMS` (§4.5) | E0108 |
| Computed number beyond its limit (§7.4, §7.11) | E0407 |
| Everything else: list or text length, display length, depth, expanded count | E0408 |

The source nesting limit is not a value limit: a program above `MAX_SOURCE_NESTING` is the parse error E0214, whatever it would compute (§5.3).

**Checks come first.** An operation whose result could exceed a limit MUST decide that from its operands, or stop as soon as the limit is crossed, without first building the oversized value:

- concatenation and `append` compare lengths before allocating;
- `range` computes its count before allocating;
- a **list literal** with more than `MAX_ITEMS` item expressions is E0408 when the literal is evaluated, **before any of its items is evaluated** (O-24);
- display and `text()` stop as soon as the output would pass `MAX_ITEMS`;
- `**` on whole numbers follows §7.4.

Operations on operands that are themselves within limits and whose result is at most twice their size (whole `+`, `-`, `*`, `//`, `%`) may compute first and check afterwards.

A failed operation never leaves a partly changed value behind.

### 7.2 Variables and scope

- Assignment creates the variable in the current scope if absent, otherwise rebinds it.
- One global scope; each call creates a local scope whose parent is the scope where the function was **defined** (lexical scoping; closures work).
- Reading a name searches the current scope, then outward; not found is E0301.
- Assignment inside a function always targets that function's local scope. A function cannot rebind an outer variable in v0.1, but it can change an outer list in place.
- A for-loop variable is an ordinary variable of the current scope and keeps its last value.
- A `func_def` binds the function to its name in the current scope when the statement executes.

### 7.3 Evaluation order (Python-compatible; O-15)

Every rule below is observable through calls with side effects, and tests rely on it.

1. **Expressions** evaluate their operands left to right. A binary operation or comparison evaluates the left operand, then the right operand, then applies the operation.
2. **`and` / `or`**: evaluate the left operand; it must be a boolean (E0302c) — this is checked before anything else; if it decides the result, the right operand is not evaluated; otherwise evaluate the right operand, which must be a boolean.
3. **Calls**: evaluate the callee, then each argument left to right, then check that the callee is a function (E0303), then check the argument count (E0304), then run the body.
4. **Indexing** `t[i]`: evaluate `t`, then `i`, then check and fetch.
5. **List literals**: if the literal has more than `MAX_ITEMS` item expressions, E0408 is raised before any item is evaluated; otherwise the items are evaluated left to right.
6. **Assignment** `target = value`: evaluate `value` **first**. Then, for a name target, bind it; for an index target `t[i]`, evaluate `t`, then `i`, then store (§7.8).
7. **Augmented assignment** `target op= value`: the target is evaluated **once, before** `value`. For a name: read its current value (E0301 if unbound). For `t[i]`: evaluate `t`, then `i`, then fetch the current element. Then evaluate `value`, apply the operator, and store to the same name or the same container and index.
8. **`show a, b, c`**: evaluate all values left to right, then build all display forms left to right, then write one line. If anything fails, nothing is written.
9. **`if`**: conditions are evaluated in order until one is true. **`repeat`** evaluates its count once. **`for`** evaluates its iterable once and iterates over a snapshot.
10. **Errors within one operation** are checked in this order after its operands are evaluated: types (E0302, E0305), then domain (E0401, E0406, E0306, E0308), then limits (E0407, E0408). Equality and display have their own fixed order (§7.5, §7.6).

### 7.4 Operators and numeric rules (AR-11)

| Operator | Operand types | Result |
|---|---|---|
| `+` | number, number | number |
| `+` | text, text | new text (length limit) |
| `+` | list, list | new list (length limit) |
| `-`, `*` | number, number | number |
| `/` | number, number | always a decimal |
| `//`, `%` | number, number | floor division and remainder, Python sign rules |
| `**` | number, number | number |
| unary `-` | number | number |
| `==`, `!=` | any, any | boolean (§7.5) |
| `<`, `<=`, `>`, `>=` | number with number, or text with text | boolean; text compares by code point |
| `and`, `or`, `not` | boolean | boolean |

Any other combination is E0302. Numeric rules:

- **Whole with whole** (`+ - * // %`, and `**` with exponent ≥ 0) gives a whole number, exact. If its magnitude exceeds `MAX_WHOLE_DIGITS`, E0407.
- **Whole `**` whole, exponent ≥ 0**: the result is exact. The implementation MUST determine that the result is too large without computing a number of more than 6,644 bits (twice the limit). A base of 0, 1 or −1 never overflows.
- **Whole `**` whole, exponent < 0**: base 0 is E0401; otherwise both operands are converted to decimals and the decimal rule applies.
- **Mixed whole and decimal** operands of `+ - * // % **`: the whole operand is first converted to a decimal. If it is too large to be a finite decimal, E0407. Comparisons are exempt (see below).
- **`/`**: divisor zero is E0401. For two whole numbers the result is the exact quotient rounded to the nearest decimal; otherwise operands are converted as above. A non-finite result is E0407.
- **`//` and `%`**: divisor zero (whole or decimal) is E0401.
- **Decimal results** must be finite. A result that would be infinite is E0407. A result too small to represent becomes `0.0`.
- **`**` with a decimal operand**: base 0 with a negative exponent is E0401; a negative base with an exponent that is not a whole value is E0406; a non-finite result is E0407.
- **Comparisons** between a whole number and a decimal compare exact mathematical values and never fail.
- `text + number` (in either order) is E0302f, whose tip is to use `show "Age:", age` or `text(age)`.
- Unary `-` on a non-number is E0302b. `not` on a non-boolean is E0302c.
- Any host exception caused by a user program's arithmetic is reported as one of these codes, never as E0900.

### 7.5 Equality (AR-12, AR-30)

`a == b` never fails for type reasons. After both operands are evaluated, the result is decided in this order:

1. If `a` and `b` are **the same list value** (the same object), the result is `true`. No limit applies.
2. If they are not both lists, the table below decides. No limit applies.
3. If they are two different list values: if `depth(a)` or `depth(b)` exceeds `MAX_NESTING`, or `count(a)` or `count(b)` exceeds `MAX_COMPARE`, the result is E0408. This depends only on the two values, not on where they first differ. The comparison MUST NOT return `false` or `true` in place of this error (O-25).
4. Otherwise the table below decides.

| Operands | Equal when |
|---|---|
| number, number | same mathematical value (whole and decimal compare exactly) |
| text, text | same code points |
| boolean, boolean | same truth value |
| nothing, nothing | always |
| list, list | same length and every pair of elements at the same position equal by this table |
| function, function | the same function value (identity). A built-in's two names are the same function. |
| different types | never — in particular `true == 1` and `[true] == [1]` are `false` |

`!=` is the negation, with the same errors.

Because step 3 looks only at the operands, the outcome cannot depend on comparison order, early exit or memoisation. Example: after `a = [0]` and twenty repetitions of `a = [a, a]`, `count(a)` is 3,145,726. Then `a == a` is `true` (step 1), while `a == b` for a separately built equal `b` is E0408 (step 3).

### 7.6 Display forms

Used by `show` and `text()`:

| Value | Display form |
|---|---|
| whole number | decimal digits, with `-` if negative |
| decimal | the shortest text that reads back as the same value, always with a `.` or an exponent (`2.0`, `0.1`, `1e+300`) — Python's `repr` |
| text | the text itself, unquoted |
| boolean | `true` or `false` (Q-08 open) |
| nothing | `nothing` |
| list | `[` items separated by `, ` `]`; a text item is written in double quotes with `\\`, `\"`, newline as `\n` and tab as `\t`; other items use their own display form |
| function | `<function NAME>`; a built-in shows its English name |

Limits (AR-30): displaying a list `v` is E0408 if `depth(v)` exceeds `MAX_NESTING`, or if its display form is longer than `MAX_ITEMS` code points. Both conditions are properties of the value; an element reachable several times contributes its text each time. The limit applies to each shown value separately, not to the whole output line.

### 7.7 Statements

- **`show`** writes the display forms separated by one space, then a newline. `show` alone writes an empty line.
- **`if` / `while`**: the condition must be a boolean (E0302c).
- **`repeat n times:`**: `n` must be a whole number ≥ 0 (E0404).
- **`for x in items:`**: `items` must be a list or text (E0305b); text iterates over characters.
- **`stop`** leaves the nearest enclosing loop; **`skip`** starts its next iteration.
- An expression statement evaluates and discards.

### 7.8 Lists, indexing and the no-cycle rule (O-13)

- Lists are 0-based (O-06). `t[i]`: `t` must be a list or text (E0305b); `i` must be a whole number (E0306c); it must satisfy `0 ≤ i < length` (E0306a, or E0306b when the container is empty). Negative indexes are not supported. Indexing text gives a one-character text.
- Text is immutable: storing into a text index is E0302d.
- **A list can never contain itself, directly or indirectly.** Lists start acyclic (literals and `+` build new lists), so only stores can create a cycle. The stores are `append(L, v)`, `L[i] = v`, and `L[i] op= …` (whose result is stored into `L[i]`).
- **Cycle rule (semantic).** Let `R(v)` be the set of list values reachable from `v`: `v` itself if it is a list, the lists among its elements, the lists among their elements, and so on. Storing `v` into `L` is E0308 exactly when `L` is in `R(v)`.
- **No depth, count or length limit takes part in this decision**, so it never depends on how the search is carried out: a store is rejected for a cycle if and only if there is one. An implementation examines each reachable list once.
- **Order of checks for a store**: (1) container type; (2) index validity; (3) the cycle rule; (4) for `append`, the length limit (E0408). If any check fails, **the list is unchanged**.
- Sharing is allowed: `b = [1]` then `a = [b, b]` is valid and displays `[[1], [1]]`.
- Deeply nested lists may be built; only displaying or comparing them is limited (§7.5, §7.6).

### 7.9 Functions

- Parameters are positional; the argument count must match exactly (E0304).
- A function that ends without `return`, or executes `return` alone, returns `nothing`.
- Functions are values: they can be stored, passed and compared (§7.5).
- Closures see the variables of their defining scope by reference. Reference example (tests use it):

```
define make_counter():
    count = [0]
    define next():
        count[0] = count[0] + 1
        return count[0]
    return next

c = make_counter()
show c(), c()        # 1 2
```

### 7.10 Execution limits

- **Call depth**: more than 200 nested calls is E0402. The implementation MUST ensure this limit is reached before any host recursion limit.
- **Step limit** (`max_steps`, optional): each statement executed and each loop iteration counts one step; exceeding it is E0403. Default: unlimited.
- **Deep values**: the cycle rule, `depth` and `count` MUST give their defined results for lists of any nesting depth; a host recursion failure is not an acceptable outcome.
- **Source nesting** (O-28): bounded exactly by `MAX_SOURCE_NESTING` (§5.3). E0214 MUST be reported, without a host recursion failure, for input of any nesting. A program within the limit MUST NOT fail in parsing, in `ast` output or in execution because of its block or bracket nesting.
- **Long operator sequences** (provisional, Q-24; not asserted by tests above the floor): operators add no nesting level. A simple statement, or the header of a compound statement, that contains at most 1,000 operations — binary operators, prefix operators, calls and index operations counted together — MUST be accepted and executed. With more than 1,000, the program MUST either work or be reported as E0408; it MUST NOT produce a traceback. The stage and span of that E0408 are not specified in v0.1.
- The step limit does not bound the cost of a single operation; the limits of §7.1 do. AgamLang v0.1 is not a sandbox for untrusted programs (R-24).

### 7.11 Built-in functions

Each row is one function value bound to two names.

| English | Hinglish (proposal) | Arguments | Behaviour and errors |
|---|---|---|---|
| `ask` | `pucho` | 0 or 1 | The argument, if given, must be text (E0302e). Writes it without a newline, reads one line from standard input, returns it without its line ending. End of input is E0405. A line longer than `MAX_ITEMS` is E0408. |
| `number` | `sankhya` | 1 | A number is returned unchanged. Text is accepted if, after removing leading and trailing spaces and tabs, it is an optional `+` or `-` followed by a literal of §4.4; the same limits apply (E0407). Anything else — including booleans, `inf`, `nan`, underscores, non-ASCII digits — is E0307. |
| `text` | `shabd` | 1 | The display form as text. |
| `length` | `lambai` | 1 | Element count of a list or code-point count of a text; other types E0302e. |
| `append` | `jodo` | 2 | The first argument must be a list (E0305a). Applies the store rules of §7.8, then the length limit (E0408). Returns `nothing`. |
| `range` | `ginti` | 1 or 2 | `range(stop)` or `range(start, stop)`; arguments must be whole numbers (E0302e). Returns a new list of the whole numbers from `start` (default 0) up to but not including `stop`; empty if `stop ≤ start`. A count above `MAX_ITEMS` is E0408, decided before allocation. |
| `type` | `prakar` | 1 | One of the texts `number`, `text`, `boolean`, `nothing`, `list`, `function`. |

A wrong argument count is E0304 for every built-in.

---

## 8. Diagnostics (normative; P-29, AR-14, AR-31)

### 8.1 Model

Every stage raises the same exception type, `AgamError`, whose `diagnostic` attribute has exactly these normative attributes:

| Attribute | Type | Content |
|---|---|---|
| `code` | `str` | `"E0306"` — never with the variant letter |
| `variant` | `str` | `""`, or one letter `"a"`…`"f"` selecting a row of Appendix A |
| `span` | tuple of four `int`, or `None` | `(line, column, end_line, end_column)` as in §4.1; `None` only for E0100 and E0900 |
| `lang_counts` | tuple of two `int` | (English parts, Hinglish parts), §8.5 |

How message arguments are stored inside the diagnostic is internal. Tests check `code`, `variant`, `span`, `lang_counts` and the rendered text.

One diagnostic ends the run. **No Python traceback is ever shown to a user.**

### 8.2 Rendering

```
Error E0201 in hello.agam, line 2:

    agar x > 5
              ^
I expected a colon ":" at the end of this line.
Tip: A line that starts with "agar" must end with ":".
```

Exact rules (complete worked outputs are in Appendix C):

1. Header: English `Error {code} in {file}, line {line}:`; Hinglish `Galti {code}, {file}, line {line}:`. `{code}` has no variant letter. `{file}` is the path exactly as given to the command line or API. `{line}` is the span's first line.
2. An empty line.
3. Four spaces, then the first source line of the span with its line ending removed and **each tab replaced by four spaces**.
4. The caret line: four spaces, then one space for every column before the span's start (measured in the tab-expanded line), then `^` once for every column of the span on that line, at least one. A span that continues past the end of the first line is marked only to the end of that line. A zero-width span gets one `^`.
5. The message line.
6. If the Appendix A row has a tip, or a suggestion exists (§8.4): `Tip: …` / `Sujhav: …`. A suggestion replaces the row's own tip.
7. Messages quote keywords **as the user typed them** (the `lexeme` of §3.4, including the inner spacing of a phrase). Where a message must name a keyword the user did not type, English messages use the first English alias of §3.2 and Hinglish messages the first Hinglish alias. For `KW_ELIF`, which has no single-word Hinglish alias, the names are `else if` and `warna agar`.
8. **Diagnostics without a span** (E0100, E0900): the header is `Error {code} in {file}:` / `Galti {code}, {file}:`, and steps 2–4 are omitted, so the message follows the header directly.
9. Diagnostics go to standard error, program output to standard output, both UTF-8. Every rendered line ends with `\n`.

### 8.3 Spans and arguments

General rules:

- A span refers to the original source (§2 line rule); rendering expands tabs afterwards.
- "Zero-width at a position" means `column == end_column`.
- **End of file**: zero-width after the last character of the last line; line 1, column 1 in an empty file.

The span, the variant and every message argument of **each** catalogue row are fixed by Appendix B. Nothing about them is left to the implementation.

### 8.4 Suggestions ("did you mean")

- Applies to E0301, and to E0204 when the unexpected token directly follows an `IDENT` that begins the statement (the typical misspelt keyword, as in `dikhoa "hi"`); in that case the suggestion is about that `IDENT`, while the span stays on the unexpected token.
- Candidates: for E0301, every name visible from the current scope (built-in names included), then every single-word alias of §3.2; for E0204, only the single-word aliases.
- Distance: Levenshtein distance (insertion, deletion, substitution, each cost 1) between the ASCII-lower-cased spellings. A candidate qualifies if the distance is at most 2, or at most 1 when the misspelt word has fewer than 5 characters. Distance 0 qualifies (a name that differs only in case).
- Choice: smallest distance; then visible names before keyword aliases; then the candidate whose spelling comes first in code-point order. Exactly one suggestion is shown.
- Text: English `Did you mean "{suggestion}"?`; Hinglish (proposal) `Kya aapka matlab "{suggestion}" tha?`.

### 8.5 Message language

- `--messages en|hi|auto` (default `auto`) is accepted by every command.
- `auto` uses Hinglish if the Hinglish part count is greater than the English part count, otherwise English.
- **How the counts reach the renderer without language-dependent parsing:** the lexer counts keyword parts by language (§3.4) while tokenising and returns the two totals with the token list. The parser copies them, unread, onto the `Program`. Whichever stage raises an error copies them into the diagnostic. For a lexical error the counts cover the tokens produced before the error. For E0100 they are (0, 0). For E0900 they are the totals of the program if lexing had finished, otherwise (0, 0).

### 8.6 Error catalogue (codes are stable and never reused)

| Code | Stage | Condition |
|---|---|---|
| E0100 | lex | File is not valid UTF-8 |
| E0101 | lex | Character cannot be used here (variants a–d) |
| E0102 | lex | Text literal not closed before end of line |
| E0103 | lex | Tabs and spaces mixed inconsistently |
| E0104 | lex | Indentation matches no earlier level |
| E0105 | lex | Malformed number (variants a, b) |
| E0106 | lex | Unknown escape sequence |
| E0107 | lex | **Numeric** literal exceeds a limit |
| E0108 | lex | Text literal longer than `MAX_ITEMS` |
| E0201 | parse | Missing `:` after a block header |
| E0202 | parse | Expected an indented block |
| E0203 | parse | Unexpected indentation |
| E0204 | parse | Unexpected token (fallback) |
| E0205 | parse | Bracket never closed (a) or closed with the wrong bracket (b) |
| E0206 | parse | `=` where a comparison was expected |
| E0207 | parse | Chained comparison |
| E0208 | parse | Reserved word used as a name |
| E0209 | parse | `return` outside a function |
| E0210 | parse | `stop`/`skip` outside a loop of the same function |
| E0211 | parse | `repeat` without `times` |
| E0212 | parse | Left side of an assignment cannot be assigned to |
| E0213 | parse | Duplicate parameter name |
| E0214 | parse | Blocks and brackets nested more than 100 levels deep |
| E0301 | run | Unknown name |
| E0302 | run | Operation not supported for these types (variants a–f) |
| E0303 | run | Called value is not a function |
| E0304 | run | Wrong number of arguments |
| E0305 | run | A list is required (a), or a list or text is required (b) |
| E0306 | run | Index out of range (a), container empty (b), index not a whole number (c) |
| E0307 | run | Cannot convert to a number |
| E0308 | run | Store would make a list contain itself |
| E0401 | run | Division by zero |
| E0402 | run | Calls nested too deeply |
| E0403 | run | Step limit exceeded |
| E0404 | run | Repeat count is not a whole number ≥ 0 |
| E0405 | run | Input ended while `ask` was waiting |
| E0406 | run | Result is not a real number |
| E0407 | run | Computed number exceeds a limit |
| E0408 | run | Length, display, depth or expanded-count limit exceeded (also an over-long operator sequence, §7.10) |
| E0900 | internal | Interpreter bug |

### 8.7 Internal errors and how they are tested

- Any exception that is not an `AgamError` and reaches the command-line entry point is reported as E0900, with exit status 70 and no traceback. E0900 has no span and is rendered by §8.2 rule 8.
- A correct interpreter cannot be made to raise E0900 by any user program, so **E0900 is tested by controlled injection, not by a program**: the in-process entry point `agamlang.cli.main` accepts a keyword-only test parameter, `_fault`, a callable invoked once, after the source has been lexed and parsed and before execution starts. A test passes a callable that raises, and checks the E0900 rendering and status 70.
- `_fault` is reachable only from Python code that calls `main` directly. There is **no** command-line option, environment variable or configuration file that triggers it.
- Fuzz tests treat status 70 as a failure.

---

## 9. Command line (normative)

Invocation: `python -m agamlang <command> [options] <file>`; the package also installs an `agam` script.

| Command | Behaviour |
|---|---|
| `run <file>` | Execute the program |
| `check <file>` | Lex and parse only; print nothing on success |
| `tokens <file>` | One token per line: `<line>:<column> <KIND> <lexeme as JSON string>`, then ` lang=<en\|hi\|mixed>` for keyword and `RESERVED_WORD` tokens |
| `tokens --canonical <file>` | One token per line: `<KIND>`, then ` <value as JSON>` for `NUMBER`, `TEXT` and `IDENT` |
| `ast [--spans] <file>` | The AST as JSON (§6.2) |
| `--version` | `agamlang <version>` |

Options: `--messages en|hi|auto` for every command; `--max-steps N` for `run`, where `N` is a whole number ≥ 1 (anything else is a usage error).

| Exit status | Meaning |
|---|---|
| 0 | Success |
| 1 | The program has an error (E01xx–E04xx) |
| 2 | Usage error, or the file cannot be read |
| 70 | Internal error (E0900) |

---

## 10. Implementation structure

**Normative** (tests and tools depend on these; all inputs and outputs are plain Python values, so no internal class is part of the contract):

```
agam-lang/
├── pyproject.toml              name "agamlang", requires-python ">=3.11", no runtime dependencies
└── src/agamlang/
    ├── keywords.py             the single runtime alias table (§3.2)
    ├── messages/en.py, hi.py   message catalogues (Appendix A)
    └── cli.py                  main(argv, *, stdin, stdout, stderr, _fault=None) -> int
```

```python
agamlang.lexer.decode(data: bytes) -> str                      # removes a BOM; raises AgamError E0100
agamlang.lexer.tokenize(text: str) -> tuple[list, tuple[int, int]]   # (tokens, lang_counts)
agamlang.parser.parse(tokens: list, lang_counts: tuple[int, int])    # -> Program
agamlang.interpreter.run(program, *, stdin, stdout, max_steps: int | None = None) -> None
agamlang.diagnostics.AgamError                                 # exception; .diagnostic as in §8.1
agamlang.diagnostics.render(diagnostic, text: str, filename: str, messages: str = "auto") -> str
agamlang.keywords.WORDS, PHRASES, RESERVED_COMPONENTS          # read-only views of §3.2
```

- `text` is the decoded source as one `str`; `filename` is used only in the header; for E0100, which has no text, `render` is called with `text=""`.
- Each token has the read-only attributes `kind` (the kind's name as a `str`, for example `"KW_IF"`), `lexeme`, `value`, `span`, `lang`, `parts` (§3.4, §4.1).
- Each node supports `==` (value equality, §6.3), has `span`, and has `to_json(spans: bool = False) -> dict` producing §6.2.
- `stdin` and `stdout` are text streams; `main` returns the exit status of §9 and never calls `sys.exit`.

**Not normative** (the implementer's choice; reviewers must not require a particular design): every other module name and split, internal class names, how tokens, nodes and diagnostics store their data, how scopes, control flow, the cycle rule, `depth`, `count` or the size checks are implemented, and how the suggestion distance is computed — provided the behaviour above holds.

---

## 11. Future directions (not part of v0.1)

Text interpolation, dictionaries, natural-order Hinglish forms, a REPL, `agam translate`, Devanagari aliases (Q-09); later a bytecode VM, native code and an AOS port. The canonical token stream and the AST are the interfaces meant to survive those changes.

---

## 12. Known limitations

| Topic | Limitation |
|---|---|
| Hinglish word order | v0.1 is keyword-first; some lines read less naturally than spoken Hindi. Mixed style is always available. |
| Reserved words | The union of all aliases and components is reserved, so `to`, `each`, `say`, `skip`, `ek`, `kuch` and similar cannot be names. E0208 explains this. |
| Romanisation | Only listed spellings are keywords; near-misses are caught by suggestions. |
| Limits | The limits of §7.1 are provisional policy. They bound single operations but do not make the interpreter safe for untrusted programs. A consequence of `MAX_COMPARE`: two different lists of lists whose elements total more than 1,000,000 cannot be compared with `==`. |
| Source nesting and long expressions | Blocks and brackets may nest at most 100 levels in total (E0214, §5.3). A statement with more than 1,000 operations is "works or E0408", not an exact rule (§7.10; Q-24). |
| Long lines in messages | A diagnostic shows the whole source line, however long. |
| Host dependence | The set of identifier letters follows the host Python's Unicode tables. |

---

## Appendix A. Message catalogue (normative for English; **Hinglish is a proposal pending owner review**)

Placeholders in braces are filled as Appendix B specifies. Codex derives its expected-message fixture by hand from this appendix, never from the implementation's catalogues. Both tables have the same 53 rows.

### A.1 English

| Code | Message | Tip |
|---|---|---|
| E0100 | This file is not valid UTF-8 text (problem at byte {offset}). | Save the file with UTF-8 encoding and try again. |
| E0101a | I do not understand the character {char} here. | — |
| E0101b | I do not understand the character {char} here. | Use straight quotes: " or '. |
| E0101c | I do not understand the character {char} here. | Write one statement per line; no semicolon is needed. |
| E0101d | I do not understand the character {char} here. | Blocks use a colon and indentation, not braces. |
| E0102 | This text starts with {quote} but never ends. | Add the closing {quote} before the end of the line. |
| E0103 | This line mixes tabs and spaces in a way that does not match the lines above it. | Indent with spaces only, or with tabs only. |
| E0104 | This line's indentation does not line up with any earlier block. | Indent it exactly like the line that started the block you want to continue. |
| E0105a | {text} is not a number I can read. | Put a space or an operator between the number and the name. |
| E0105b | {text} is not a number I can read. | Write a digit on both sides of the point, for example 0.5 or 5.0. |
| E0106 | I do not know the escape {escape} inside text. | Use \n, \t, \\, \" or \'. |
| E0107 | This number is too large. | Whole numbers can have at most 1000 digits; decimal numbers can go up to about 1.8e308. |
| E0108 | This text is too long. | Text can hold at most 1,000,000 characters. |
| E0201 | I expected a colon ":" at the end of this line. | A line that starts with "{kw}" must end with ":". |
| E0202 | I expected an indented block after this line. | Indent the next line to put it inside "{kw}". |
| E0203 | This line is indented, but nothing above it starts a block. | Remove the extra spaces at the start of the line. |
| E0204 | I did not expect {found} here. | I expected {expected}. |
| E0205a | This {open} is never closed. | Add the matching {close}. |
| E0205b | I found {found}, but the {open} on line {line} needs {close}. | — |
| E0206 | A single "=" stores a value; it cannot compare. | Use "==" to compare two values. |
| E0207 | I cannot compare three things in a row. | Split it in two, for example: a < b and b < c. |
| E0208 | "{word}" is a reserved word in AgamLang, so it cannot be used as a name. | Choose a different name. |
| E0209 | "{kw}" can only be used inside a function. | — |
| E0210 | "{kw}" can only be used inside a loop. | Loops start with "while", "repeat" or "for". |
| E0211 | I expected "times" after the repeat count. | Write it like this: {kw} 3 times: |
| E0212 | I cannot store a value into this. | The left side of "=" must be a name or a list position such as items[0]. |
| E0213 | The name "{name}" is used for two parameters. | Give each parameter a different name. |
| E0214 | This part of the program is nested too deeply. | Blocks and brackets can be nested at most 100 levels deep in total. Move the inner part into a function, or store part of the calculation in a variable first. |
| E0301 | I do not know the name "{name}". | Create it first, for example: {name} = 0 |
| E0302a | I cannot use {op} with {left} and {right}. | — |
| E0302b | I cannot use {op} with {operand}. | — |
| E0302c | I need a true/false value here, but got {type}. | Use a comparison such as x > 0. |
| E0302d | Text cannot be changed in place. | Build a new text with "+" instead. |
| E0302e | {function} needs {expected} here, but got {type}. | — |
| E0302f | I cannot use + with text and a number. | To show them together write: show "…", value — or convert the number with text(value). |
| E0303 | This value is {type}, not a function, so it cannot be called. | — |
| E0304 | {function} needs {expected} value(s), but got {got}. | — |
| E0305a | I can only add to a list, but got {type}. | — |
| E0305b | I need a list or text here, but got {type}. | — |
| E0306a | Position {index} does not exist. This {kind} has {length} item(s), so valid positions are 0 to {last}. | Counting starts at 0. |
| E0306b | This {kind} is empty, so it has no positions. | — |
| E0306c | A position must be a whole number, but got {type}. | — |
| E0307 | I cannot turn {value} into a number. | A number looks like 42, -7 or 3.14. |
| E0308 | A list cannot contain itself. | Store a value that does not contain this list, either directly or inside another list. |
| E0401 | I cannot divide by zero. | — |
| E0402 | Functions are calling each other too deeply (more than 200 calls inside calls). | Check that "{name}" stops calling itself at some point. |
| E0403 | The program ran for more than {limit} steps and was stopped. | — |
| E0404 | The repeat count must be a whole number that is 0 or more, but got {value}. | — |
| E0405 | The program asked for input, but there was none left. | — |
| E0406 | This calculation does not have a real-number answer. | A negative number cannot be raised to a fractional power. |
| E0407 | This number is too large for AgamLang. | Whole numbers can have at most 1000 digits; decimal numbers can go up to about 1.8e308. |
| E0408 | This value is too large. | Lists and text can hold at most 1,000,000 items. A list can be shown or compared only if it is nested at most 100 levels deep and holds at most 1,000,000 items in total. |
| E0900 | Something went wrong inside AgamLang itself. This is a bug in AgamLang, not in your program. | Please report it together with your program. |


### A.2 Hinglish (proposal — wording not approved)

The Hinglish forms of type names, kinds and expected-phrases are in Appendix B.2.

| Code | Message | Tip |
|---|---|---|
| E0100 | Yeh file sahi UTF-8 text nahi hai (byte {offset} par dikkat hai). | File ko UTF-8 encoding mein save karke dobara chalaiye. |
| E0101a | Mujhe yahan {char} character samajh nahi aaya. | — |
| E0101b | Mujhe yahan {char} character samajh nahi aaya. | Seedhe quotes use kijiye: " ya '. |
| E0101c | Mujhe yahan {char} character samajh nahi aaya. | Ek line mein ek hi statement likhiye; semicolon ki zaroorat nahi hai. |
| E0101d | Mujhe yahan {char} character samajh nahi aaya. | Block ke liye colon aur indentation use hota hai, braces nahi. |
| E0102 | Yeh text {quote} se shuru hua, par khatam nahi hua. | Line khatam hone se pehle {quote} lagaiye. |
| E0103 | Is line mein tab aur space aise mile hain jo upar ki lines se match nahi karte. | Indent ke liye sirf space, ya sirf tab use kijiye. |
| E0104 | Is line ka indentation kisi pehle wale block se nahi milta. | Ise usi line jitna indent kijiye jisne woh block shuru kiya tha. |
| E0105a | {text} ko main number ki tarah nahi padh paya. | Number aur naam ke beech space ya operator lagaiye. |
| E0105b | {text} ko main number ki tarah nahi padh paya. | Point ke dono taraf digit likhiye, jaise 0.5 ya 5.0. |
| E0106 | Text ke andar {escape} escape main nahi jaanta. | \n, \t, \\, \" ya \' use kijiye. |
| E0107 | Yeh number bahut bada hai. | Poore number mein zyada se zyada 1000 digit ho sakte hain; decimal number lagbhag 1.8e308 tak ja sakte hain. |
| E0108 | Yeh text bahut lamba hai. | Text mein zyada se zyada 1,000,000 character ho sakte hain. |
| E0201 | Is line ke end mein ":" hona chahiye. | "{kw}" se shuru hone wali line ":" par khatam hoti hai. |
| E0202 | Is line ke baad ek indented block hona chahiye. | Agli line ko indent kijiye taaki woh "{kw}" ke andar aaye. |
| E0203 | Yeh line indented hai, par upar koi block shuru nahi hua. | Line ke shuru ke extra space hata dijiye. |
| E0204 | Yahan {found} ki ummeed nahi thi. | Yahan {expected} hona chahiye tha. |
| E0205a | Yeh {open} kabhi band nahi hua. | Iske saath wala {close} lagaiye. |
| E0205b | Yahan {found} mila, par line {line} ke {open} ke liye {close} chahiye. | — |
| E0206 | Akela "=" value rakhta hai; tulna nahi karta. | Do values ki tulna ke liye "==" use kijiye. |
| E0207 | Main teen cheezon ki ek saath tulna nahi kar sakta. | Ise do hisson mein likhiye, jaise: a < b aur b < c. |
| E0208 | "{word}" AgamLang ka reserved shabd hai, isliye ise naam ki tarah use nahi kar sakte. | Koi doosra naam chuniye. |
| E0209 | "{kw}" sirf kaam (function) ke andar use ho sakta hai. | — |
| E0210 | "{kw}" sirf loop ke andar use ho sakta hai. | Loop "jab tak", "dohrao" ya "har" se shuru hote hain. |
| E0211 | Ginti ke baad "baar" hona chahiye. | Aise likhiye: {kw} 3 baar: |
| E0212 | Main ismein value nahi rakh sakta. | "=" ke left mein naam ya list ki position honi chahiye, jaise items[0]. |
| E0213 | "{name}" naam do parameters ke liye use hua hai. | Har parameter ko alag naam dijiye. |
| E0214 | Program ka yeh hissa bahut zyada nested hai. | Blocks aur brackets kul milakar zyada se zyada 100 level tak nested ho sakte hain. Andar wale hisse ko function mein le jaiye, ya calculation ka ek hissa pehle variable mein rakhiye. |
| E0301 | Main "{name}" naam ko nahi jaanta. | Pehle ise banaiye, jaise: {name} = 0 |
| E0302a | Main {op} ko {left} aur {right} ke saath use nahi kar sakta. | — |
| E0302b | Main {op} ko {operand} ke saath use nahi kar sakta. | — |
| E0302c | Yahan sach/jhooth value chahiye, par {type} mila. | Koi tulna likhiye, jaise x > 0. |
| E0302d | Text ko usi jagah badla nahi ja sakta. | "+" se naya text banaiye. |
| E0302e | {function} ko yahan {expected} chahiye, par {type} mila. | — |
| E0302f | Main + ko text aur number ke saath use nahi kar sakta. | Dono ko saath dikhane ke liye likhiye: dikhao "…", value — ya number ko shabd(value) se text banaiye. |
| E0303 | Yeh value {type} hai, kaam (function) nahi, isliye ise call nahi kar sakte. | — |
| E0304 | {function} ko {expected} value chahiye, par {got} mili. | — |
| E0305a | Main sirf list mein jod sakta hoon, par {type} mila. | — |
| E0305b | Yahan list ya text chahiye, par {type} mila. | — |
| E0306a | Position {index} maujood nahi hai. Is {kind} mein {length} item hain, isliye sahi positions 0 se {last} tak hain. | Ginti 0 se shuru hoti hai. |
| E0306b | Yeh {kind} khaali hai, isliye ismein koi position nahi hai. | — |
| E0306c | Position poora number hona chahiye, par {type} mila. | — |
| E0307 | Main {value} ko number mein nahi badal sakta. | Number aise dikhta hai: 42, -7 ya 3.14. |
| E0308 | Ek list apne aap ko apne andar nahi rakh sakti. | Aisi value rakhiye jismein yeh list na ho, na seedhe aur na kisi doosri list ke andar. |
| E0401 | Main zero se divide nahi kar sakta. | — |
| E0402 | Kaam ek doosre ko bahut gehrai tak call kar rahe hain (200 se zyada call ke andar call). | Dekhiye ki "{name}" kahin par khud ko call karna band karta hai. |
| E0403 | Program {limit} se zyada steps chala, isliye roka gaya. | — |
| E0404 | Dohrane ki ginti 0 ya usse bada poora number honi chahiye, par {value} mila. | — |
| E0405 | Program ne input maanga, par koi input bacha nahi tha. | — |
| E0406 | Is hisaab ka koi real-number jawab nahi hai. | Negative number ko fractional power nahi de sakte. |
| E0407 | Yeh number AgamLang ke liye bahut bada hai. | Poore number mein zyada se zyada 1000 digit ho sakte hain; decimal number lagbhag 1.8e308 tak ja sakte hain. |
| E0408 | Yeh value bahut badi hai. | List aur text mein zyada se zyada 1,000,000 item ho sakte hain. List tabhi dikhayi ya compare ki ja sakti hai jab woh 100 level se zyada nested na ho aur usmein kul 1,000,000 se zyada item na hon. |
| E0900 | AgamLang ke andar kuch gadbad ho gayi. Yeh AgamLang ka bug hai, aapke program ka nahi. | Kripya ise apne program ke saath report kijiye. |

---

## Appendix B. Variants, spans and arguments (normative; AR-31)

This appendix fixes, for every row of Appendix A, when it is used, what its span is, and exactly how each placeholder is filled. The Hinglish words in B.2 are proposals pending the owner's review, like Appendix A.2.

### B.1 Placeholder formats

| Placeholder | Exact form |
|---|---|
| `{char}` | The character in double quotes, one space, then its code point in parentheses: `"€" (U+20AC)`. Code points are upper-case hexadecimal with at least four digits. For a character of Unicode general category Cc, Cf, Zl or Zp only the code point is written: `U+0007`. |
| `{quote}` | The opening quote character itself: `"` or `'` |
| `{text}` | The source text of the span, unchanged |
| `{escape}` | The two source characters of the span, unchanged, for example `\q` |
| `{kw}`, `{word}` | The token's `lexeme`, unchanged (for a phrase, with its inner spacing) |
| `{name}` | E0213, E0301: the identifier as typed. E0402: the called function's name by rule N. |
| `{op}` | The operator's lexeme in double quotes: `"+"` |
| `{type}`, `{left}`, `{right}`, `{operand}` | A type name from B.2 |
| `{function}` | **Rule N**: a user-defined function's definition name; a built-in's English name in English messages and its Hinglish name in Hinglish messages |
| `{expected}` in E0304 | User-defined function: its parameter count in decimal. Built-in: the count of §7.11, written `1`, `2`, `0 or 1` or `1 or 2`, with the B.2 word for "or". |
| `{expected}` in E0302e | The B.2 phrase for what the argument must be: `text` (`ask`), `a whole number` (`range`), `a list or text` (`length`) |
| `{expected}` in E0204 | **Rule X** below |
| `{found}` | **Rule F** below |
| `{got}`, `{index}`, `{length}`, `{last}`, `{limit}`, `{line}`, `{offset}` | Decimal digits without separators; `{index}` keeps a leading `-` |
| `{kind}` | The B.2 word for `list` or `text` |
| `{value}` | **Rule V**: a number's display form; a text in double quotes with the escaping of §7.6, cut to its first 30 code points followed by `...` if it is longer; for any other type, its B.2 type name |
| `{open}`, `{close}` | The bracket character in double quotes: `"("`, `")"`, `"["`, `"]"` |
| `{suggestion}` | The candidate's spelling, unchanged |

**Rule F — `{found}`.** `IDENT`, keyword, `RESERVED_WORD`, operator or punctuation: the lexeme in double quotes. `NUMBER`: the B.2 phrase "the number", a space, the lexeme. `TEXT`: the B.2 phrase "the text", a space, the lexeme as typed with its quotes, cut to 30 code points followed by `...` if longer. `NEWLINE`: "the end of the line". `DEDENT`: "the end of the block". `EOF`: "the end of the program".

**Rule X — `{expected}` in E0204.** Exactly one phrase per parser position:

| Position of the unexpected token | `{expected}` |
|---|---|
| Where a statement must start | `a statement` |
| Where an expression must start (after an operator, an assignment operator, `(`, `[`, `,`, or a keyword that is followed by an expression) | `a value` |
| Where a name is required (function name, parameter, for-loop variable) and the token is not a keyword | `a name` |
| After the function name in a definition | `"("` |
| After a parameter, after a call argument, or after the expression inside parentheses | `")"` |
| After an item of a list literal, or after an index expression | `"]"` |
| After the loop variable of `for` | The `KW_IN` keyword in double quotes, named by §8.2 rule 7 (`"in"`, `"mein"`) |
| After a complete simple statement | `the end of the line` |

A closing bracket with no open bracket is the last case. An `INDENT` where no block is open is E0203, not E0204.

### B.2 Language-dependent words

| Meaning | English | Hinglish (proposal) |
|---|---|---|
| Type `number` | `a number` | `number` |
| Type `text` | `text` | `text` |
| Type `boolean` | `a true/false value` | `sach/jhooth value` |
| Type `nothing` | `nothing` | `khaali` |
| Type `list` | `a list` | `list` |
| Type `function` | `a function` | `kaam` |
| `{kind}` | `list`, `text` | `list`, `text` |
| "or" in argument counts | `or` | `ya` |
| Expected-argument phrases | `text`, `a whole number`, `a list or text` | `text`, `poora number`, `list ya text` |
| Rule X phrases | `a statement`, `a value`, `a name`, `the end of the line` | `ek statement`, `ek value`, `ek naam`, `line ka end` |
| Rule F phrases | `the number`, `the text`, `the end of the line`, `the end of the block`, `the end of the program` | `number`, `text`, `line ka end`, `block ka end`, `program ka end` |

### B.3 Row by row

"Node" spans follow §6.5. "Zero-width after X" means a zero-width span immediately after the last character of X.

| Row | Used when | Span | Arguments |
|---|---|---|---|
| E0100 | The bytes are not valid UTF-8 | None | `offset`: zero-based offset, counted from the first byte of the file (a byte-order mark counts), of the first byte that cannot be decoded |
| E0101a | A character cannot start a token, and b–d do not apply | That character | `char` |
| E0101b | The character is U+2018, U+2019, U+201C or U+201D | That character | `char` |
| E0101c | The character is `;` | That character | `char` |
| E0101d | The character is `{` or `}` | That character | `char` |
| E0102 | End of line or file inside a text literal (§4.5) | From the opening quote to the end of the line | `quote` |
| E0103 | §4.2 | The line's indentation | — |
| E0104 | §4.2 | The line's indentation | — |
| E0105a | A number is immediately followed by an identifier character (also `1e` with no exponent digits) | From the first digit to the last adjacent identifier character | `text` |
| E0105b | `.` followed by a digit where a token must start (`.5`), or digits followed by `.` and no digit (`5.`) | The `.` and its adjacent digits | `text` |
| E0106 | §4.5 | The backslash and the character after it | `escape` |
| E0107 | §4.4 | The whole literal | — |
| E0108 | §4.5 | The opening quote character only | — |
| E0201 | A block header's expression is complete and the next token is not `COLON` | Zero-width after the last token of the header | `kw`: the header's keyword |
| E0202 | After a header's `COLON`, the next token is not `NEWLINE`, or the `NEWLINE` is not followed by `INDENT` | The `COLON` token | `kw`: the header's keyword |
| E0203 | An `INDENT` where no block is open | The line's indentation | — |
| E0204 | No more specific code applies (§5.2) | The unexpected token; zero-width at its position for `NEWLINE`, `DEDENT`, `EOF` | `found`, `expected` |
| E0205a | End of file while `(` or `[` is open | The innermost open bracket | `open`, `close` |
| E0205b | A closing bracket that does not match the innermost open bracket | The closing token | `found`, `open`, `close`, `line` (of the open bracket) |
| E0206 | In the condition of `if`, else-if or `while`: an `ASSIGN` token directly after a complete expression | The `=` token | — |
| E0207 | A second comparison operator directly after a complete comparison | The second operator | — |
| E0208 | §5.2 cases (a)–(c) | The reserved or keyword token | `word` |
| E0209 | §5.2 | The keyword token | `kw` |
| E0210 | §5.2 | The keyword token | `kw` |
| E0211 | After `KW_REPEAT` and its expression, the next token is not `KW_TIMES` | Zero-width after the count expression | `kw`: the repeat keyword |
| E0212 | An assignment operator follows an expression that is neither a name nor an index expression | That whole left-hand expression | — |
| E0213 | §5.2 | The second occurrence of the name | `name` |
| E0214 | §5.3: an opening bracket is consumed, or a block is entered, while 100 blocks and bracket pairs are already open | The opening bracket token; for a block, the first token of its first statement | — |
| E0301 | An unbound name is read | The `Name` node | `name` |
| E0302a | A binary arithmetic or ordering operator with unsupported operand types, other than f | The `Binary` or `Compare` node | `op`, `left`, `right` |
| E0302b | Unary `-` on a non-number | The `Unary` node | `op`, `operand` |
| E0302c | A non-boolean condition of `if`, else-if or `while`; a non-boolean operand of `and`, `or`, `not` | That condition or operand expression | `type` |
| E0302d | A store into an index of a text | The `Index` node that is the target | — |
| E0302e | A built-in argument of the wrong type, other than `append`'s first | The `Call` node | `function`, `expected`, `type` |
| E0302f | `+` with one text operand and one number operand, in either order | The `Binary` node | — |
| E0303 | The callee is not a function | The `Call` node | `type` |
| E0304 | Wrong number of arguments | The `Call` node | `function`, `expected`, `got` |
| E0305a | `append`'s first argument is not a list | The `Call` node | `type` |
| E0305b | A for-loop iterable, or the target of an index expression, is neither a list nor a text | The iterable expression, or the target expression | `type` |
| E0306a | The index is a whole number outside `0 … length − 1` (including negative) and the container is not empty | The `Index` node | `index`, `kind`, `length`, `last` |
| E0306b | The index is a whole number and the container is empty | The `Index` node | `kind` |
| E0306c | The index is not a whole number (a decimal, a boolean, or another type) | The index expression | `type` |
| E0307 | `number()` cannot convert its argument | The `Call` node | `value` |
| E0308 | §7.8 | The target `Index` node, or the `append` `Call` node | — |
| E0401 | §7.4 | The `Binary` node | — |
| E0402 | A call would be the 201st nested call | That `Call` node | `name`: the called function |
| E0403 | §7.10 | The statement about to be executed | `limit` |
| E0404 | §7.7 | The count expression | `value` |
| E0405 | §7.11 | The `Call` node | — |
| E0406 | §7.4 | The `Binary` node | — |
| E0407 | §7.4, §7.11 | The `Binary` node, or the `Call` node of `number` | — |
| E0408 | Length limit in `+`; in `append`; count limit in `range`; input line in `ask`; item count of a list literal; display limit in `show` or `text()`; depth or count limit in `==`/`!=` | Respectively: the `Binary` node; the `Call` node; the `Call` node; the `Call` node; the `ListLit` node; the shown expression or the `text` `Call` node; the `Compare` node | — |
| E0900 | §8.7 | None | — |

---

## Appendix C. Worked examples (normative for English; AR-31)

Each example gives a complete source file and the exact text written to standard error. In sources, `→` stands for one tab character. They exist so that golden files can be checked against the specification instead of against an implementation. Hinglish output is shown once, as an illustration of the layout; its wording is a proposal.

### C.1 Missing colon; zero-width span; keyword quoted as typed

`hello.agam`, run with `--messages en`:

```
x = 7
agar x > 5
    dikhao "big"
```

```
Error E0201 in hello.agam, line 2:

    agar x > 5
              ^
I expected a colon ":" at the end of this line.
Tip: A line that starts with "agar" must end with ":".
```

### C.2 The same file with the default `--messages auto`

The lexer finished, with 0 English and 2 Hinglish keyword parts, so Hinglish is used (wording is a proposal):

```
Galti E0201, hello.agam, line 2:

    agar x > 5
              ^
Is line ke end mein ":" hona chahiye.
Sujhav: "agar" se shuru hone wali line ":" par khatam hoti hai.
```

### C.3 Unknown name with a suggestion

`sum.agam`:

```
total = 10
show totl
```

Distance from `totl` to `total` is 1; the word has four characters, so the threshold is 1.

```
Error E0301 in sum.agam, line 2:

    show totl
         ^^^^
I do not know the name "totl".
Tip: Did you mean "total"?
```

### C.4 Tab expansion and an index error

`list.agam`:

```
items = [1, 2, 3]
if true:
→show items[5]
```

The `Index` node occupies columns 7–14 of line 3. The tab becomes four spaces in the excerpt, and the carets move with it.

```
Error E0306 in list.agam, line 3:

        show items[5]
             ^^^^^^^^
Position 5 does not exist. This list has 3 item(s), so valid positions are 0 to 2.
Tip: Counting starts at 0.
```

### C.5 Wrong number of arguments to a user-defined function

`f.agam`, run with `--messages en`:

```
kaam jodo2(a, b, c):
    wapas a + b + c
dikhao jodo2(1)
```

```
Error E0304 in f.agam, line 3:

    dikhao jodo2(1)
           ^^^^^^^^
jodo2 needs 3 value(s), but got 1.
```

### C.6 Invalid UTF-8 after a byte-order mark

`bad.agam` consists of the bytes `EF BB BF 73 68 6F 77 20 FF`. The first undecodable byte is at offset 8. There is no span, so there is no excerpt, and the counts are (0, 0), so English is used:

```
Error E0100 in bad.agam:
This file is not valid UTF-8 text (problem at byte 8).
Tip: Save the file with UTF-8 encoding and try again.
```

### C.7 Internal error by injection

Any valid `prog.agam` with English keywords, with a raising `_fault` passed to `agamlang.cli.main`. Exit status 70:

```
Error E0900 in prog.agam:
Something went wrong inside AgamLang itself. This is a bug in AgamLang, not in your program.
Tip: Please report it together with your program.
```

### C.8 Bracket never closed; error found at end of file

`p.agam` (one line, ending with a line ending):

```
show (1 + 2
```

The header shows the code without its variant letter.

```
Error E0205 in p.agam, line 1:

    show (1 + 2
         ^
This "(" is never closed.
Tip: Add the matching ")".
```

### C.9 A mixed else-if phrase quoted as typed

`m.agam`:

```
agar x > 5:
    show 1
warna if x > 2
    show 2
```

Keyword parts: `agar` (hi), `show` (en), `warna` (hi), `if` (en), `show` (en) — 3 English, 2 Hinglish — so `auto` selects English.

```
Error E0201 in m.agam, line 3:

    warna if x > 2
                  ^
I expected a colon ":" at the end of this line.
Tip: A line that starts with "warna if" must end with ":".
```

### C.10 Node spans

`s.agam` contains the single line `x = (1 + 2) * 3`. `agam ast --spans s.agam` prints:

```
{
  "type": "Program",
  "body": [
    {
      "type": "Assign",
      "target": {
        "type": "Name",
        "id": "x",
        "span": [1, 1, 1, 2]
      },
      "op": "=",
      "value": {
        "type": "Binary",
        "op": "*",
        "left": {
          "type": "Binary",
          "op": "+",
          "left": {
            "type": "NumberLit",
            "value": 1,
            "span": [1, 6, 1, 7]
          },
          "right": {
            "type": "NumberLit",
            "value": 2,
            "span": [1, 10, 1, 11]
          },
          "span": [1, 6, 1, 11]
        },
        "right": {
          "type": "NumberLit",
          "value": 3,
          "span": [1, 15, 1, 16]
        },
        "span": [1, 5, 1, 16]
      },
      "span": [1, 1, 1, 16]
    }
  ],
  "span": [1, 1, 1, 16]
}
```

A span array is written on one line; everything else uses two-space indentation.

### C.11 Limits at their boundaries

| Program fragment | Result |
|---|---|
| A text literal with exactly 1,000,000 characters | Accepted |
| A text literal with 1,000,001 characters | E0108 at the opening quote |
| `range(1000000)` | A list of 1,000,000 items |
| `range(1000001)` | E0408; no list is built |
| A list literal with 1,000,001 items whose first item is a call | E0408; the call is not made |
| `10 ** 999` | A whole number with 1,000 digits |
| `10 ** 1000` | E0407 |
| A list of depth 100 (`[[…[0]…]]`), shown | Displayed |
| A list of depth 101, built with `append` (a literal that deep is E0214), shown | E0408 |
| A list of depth 101, appended to another list | Accepted |
| Two different lists with expanded count 1,000,000 each, compared | `true` or `false` |
| Two different lists, one with expanded count 1,000,001, compared | E0408, even if their lengths differ |
| A list with expanded count above the limit, compared with itself | `true` |
| `a = [0]`, `b = [a]`, `append(a, b)` | E0308; `a` still displays `[0]` |
| A list literal nested 100 deep, `[[…[]…]]`, as a top-level statement | Accepted; its value has depth 100 and can be shown |
| The same with 101 bracket pairs | E0214 at the 101st opening bracket |
| 101 opening brackets, none of them closed | E0214 at the 101st opening bracket, not E0205a |
| 100 nested blocks | Accepted |
| A 101st nested block | E0214 at the first token of its first statement, whatever that token is |
| Inside 60 nested blocks, an expression with 40 nested bracket pairs | Accepted |
| The same with 41 nested bracket pairs | E0214 at the 41st opening bracket |
| `1 + 1 + … + 1` with 1,000 `+` and no brackets | Accepted; operators add no level |
| `a[0][0]…[0]` with 200 index operations one after another | Accepted; the pairs are side by side, each at level 1 |
