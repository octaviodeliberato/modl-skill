# Dialog items and connectors — the build sheet

Disclosed reference for [`modl-block`](SKILL.md). Connectors are placed with icon-pane tools; dialog items are placed with the dialog editor's New Dialog Item dialog. Both are drawn, not written — the build sheet is how that drawing gets specified precisely enough that a human can do it without a follow-up question, and the ModL code can then reference the result as an ordinary static variable.

## Connector rows

Each row: **name** (must start with a letter, alphanumeric only, under 32 characters; must end in `In` or `Out` — that suffix is what makes it an input or output), **type** (Value, Item, Flow, Universal, Array, or User-Defined/diamond), **normal or variable** (a variable connector is an expandable row of connectors, managed in code with `ConArrayGetValue`, `ConArraySetValue`, `ConArraySetNumCons`, `ConArrayGetConNumber`).

Reading and writing: a normal input connector is read with `x = firstConIn;`; a normal output connector is set with `totalOut = 1.0;`. All connector values are real; assigning an integer costs a silent conversion.

## Dialog item rows

Each row: **name** (required for most types; optional and usually omitted for Static Text and Text Frame — an unnamed item can't be read or set from code, and won't appear in the block's Variables pane), **type** (one of the table below), **text/title** if the type has one, **tab** it lives on, and any of: Display only, Hide item, Visible in all tabs, radio group number, number format (General/Currency/Integer/Scientific/Percentage).

| Type | Code-facing shape | Notes |
|---|---|---|
| Parameter (Number) | real-valued variable | can be Display only |
| Editable Text / Editable Text 31 | string variable (255 / 31 chars) | can be Display only |
| Dynamic Text | string dynamic array, up to 32,000 chars | needs setup — see below |
| Check Box | boolean (true/false) variable, and a message of the same name | |
| Radio Button | boolean variable, one group of buttons; a message of the same name | see gotcha below |
| Popup Menu | integer variable, **1-based** position of the selected item | a message of the same name fires on selection |
| Button | fires a message of the same name when clicked; title is a string variable | title is not persisted, see below |
| Static Text (Label) | string variable if named | not persisted, see below |
| Text Frame | string variable if named (its title) | groups other items visually only |
| Switch | integer variable, 0 or 1 | |
| Slider | 3-element real array: `[0]` min, `[1]` max, `[2]` current | |
| Meter | 3-element real array: `[0]` min, `[1]` max, `[2]` current | output only |
| Data Table / Text Table | 2D array, 0-based, e.g. `table[row][col]` | text table entries are strings; convert with `StrToReal` |
| Calendar | ExtendSim date value | |
| Embedded Object (Windows only) | OLE/ActiveX placeholder | |

### Radio button gotcha

Selecting one button in a group does **not** clear the others in code — you must set the chosen one to `TRUE` explicitly (`Med = TRUE;`), which clears the rest of the group automatically. Setting a button to `FALSE` alone leaves the group's state ambiguous, possibly all-false.

### Titles and labels are not persisted

A button's title, or a static-text label's text, read back in code is always what was typed into the New Dialog Item dialog when the item was created — even after code has changed it on screen. If the code changes a title or label, it must set it again every time in `On DialogOpen`, or the modeler will see the original text the next time the block's dialog opens.

### Dynamic text setup

Unlike Editable Text, a Dynamic Text item needs to be wired to a string dynamic array before it's usable, typically in `CreateBlock`:

```
string aStringDynamicArray[];   // declared at top of code
...
on CreateBlock
{
    myDynamicTextItem = DynamicTextArrayNumber(aStringDynamicArray);
}
```

## Not on the build sheet

Icon artwork and 3D (E3D) object models are drawn or authored visually, not specified as text — flag in the hand-off where an icon or animation-object placeholder is expected, but don't attempt to describe the artwork itself. Help text has no such restriction: it's plain (optionally formatted) text and can be written directly, same as the ModL code.
