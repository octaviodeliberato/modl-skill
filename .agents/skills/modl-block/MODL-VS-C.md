# ModL vs C/C++

Disclosed reference for [`modl-block`](SKILL.md) — consult when doubling logic in C (write the double so it survives translation) and during the final syntax pass (catch what a compiler would have). ModL is close enough to C that these are the differences that actually bite, not a full language comparison.

| In C, you'd write | In ModL, write instead |
|---|---|
| `i++` used as an expression (`a[++i] = 5;`) | Statement only: `i++;` then `a[i] = 5;` on its own line |
| `for (expr; expr; expr)` | `for (statement; boolean; statement)` — same shape, but the first and third clauses are statements, not arbitrary expressions |
| `^` for exclusive-or | `^` is exponentiation in ModL; there's no bitwise XOR operator — use the bit-handling functions (e.g. `BitAnd`) for bit operations, none for XOR by operator |
| `strcat(dest, "abc")` | `stringVar = stringVar + "abc";` — `+` concatenates strings directly |
| `sprintf`/`ftoa` to stringify a number | `stringVar = x;` — assignment does the conversion |
| `strcmp(a, b) < 0` | `if (a < b)` — string relational operators work directly |
| `#define` as a text macro | `#define` only declares a symbol for `#ifdef`/`#ifndef`; there is no macro substitution |
| Case-sensitive identifiers | ModL is case-insensitive — don't rely on case to distinguish two names |
| Pointers into a growable buffer | Resizable dynamic arrays (one dimension left undeclared) |
| Structs | Linked lists, for complex data structures |
| Unchecked array access | ModL checks array bounds and aborts on overrun — a bug the C double will not catch by itself; check bounds explicitly in the double if the logic relies on them |
| `!=` only | `!=` or `<>`, both work |

Function declarations are ANSI-style only (no K&R). Message handlers and user-defined functions can be overridden by re-declaring them further down the file — there's no equivalent redeclaration-is-an-error rule to port from C.
