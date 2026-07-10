---
name: modl-block
description: Write ExtendSim 9 ModL code for a custom block from a plain-language behavior spec. Use when the user wants ModL code, an ExtendSim block, or custom block/dialog/connector logic, or asks to implement or modify block behavior for ExtendSim.
---

ModL only compiles and runs inside the ExtendSim IDE, and dialog items and connectors are drawn with the dialog editor and icon pane, not written as text. Neither constraint is a reason to guess: it's a reason to hand the GUI-only parts to the human as a **build sheet**, and to give the runnable parts a **double** — a stand-in that takes the risk of execution so the ModL, which can't run here, doesn't have to.

## Steps

1. **Pin the contract.** From the spec, decide before writing anything: the model type (continuous, discrete event, discrete rate, or an equation-block snippet — this fixes which lifecycle message handlers apply), every connector the block needs (name, In/Out, type), and every dialog item it needs (name, type). Nothing in later steps invents a name that wasn't decided here.

2. **Write the build sheet.** One row per connector and per dialog item from step 1, each carrying what a human needs to create it by hand in ExtendSim — see [DIALOG-ITEMS.md](DIALOG-ITEMS.md) for the row fields and the mechanics of each item type. Every identifier the code in step 3 references must trace back to a row here; there is no other way these names come into existence.

3. **Write the ModL code.** Lay it out the way ModL expects: type declarations and constants, then user-defined functions, then message handlers (`on MessageName { ... }`). Connector and dialog-item names from the build sheet are already static variables — reference them directly, no declaration. Any dialog item whose title or text your code sets programmatically (a button title, a static-text label) is not persisted — re-set it every time in `On DialogOpen`, or the modeler sees the original text the item was created with.

4. **Double the portable logic in C.** Find the piece of the requested logic that touches nothing ExtendSim-specific — no connector, no dialog item, no message-handler state, no ModL-only function, just math, strings, arrays, and control flow. Write that piece as standalone C, respecting the differences in [MODL-VS-C.md](MODL-VS-C.md), compile it (`cc`/`gcc`) and run it against representative inputs from the spec. Only once the double's output matches expectations, transliterate it into the ModL in step 3. If the requested logic can't be separated from ExtendSim state — it depends on connector timing, item messages, or the simulation clock — say so plainly in step 6 rather than leaving it silently unverified.

5. **Confirm every name against the reference.** List every distinct ModL function call and every `on X` message handler in the code. For each one, grep `assets/developer-reference.txt` at the repo root by name alone (no trailing `(` — the manual doesn't always print one) and confirm it turns up as a real function or message, not just an incidental substring hit — the function-reference index entries (`Name NNN`, a page number) and the message-handler tables are the reliable confirmations. ModL reads like C but is not C's library: a plausible-looking name (`atof`, `strcat`, `sprintf`) is exactly the failure mode this step exists to catch. None left unconfirmed. Then do one pass of the code against the ModL/C++ differences in [MODL-VS-C.md](MODL-VS-C.md) — this table is what a compiler would have caught, and nothing here compiles it for you.

6. **Hand off.** Deliver the build sheet, the ModL code ready to paste into the block's Code pane, and a short note: what the double verified, what's ExtendSim-only and still unverified, and one or two `DebugMsg(...)` or `comments = ...;` lines the human can leave in for the first run inside ExtendSim (ModL has no other test harness — see the manual's "Debugging block code without the Source Code Debugger").
