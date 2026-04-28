# Grammar: CLASS DEFINITION DEFERRED and LOCAL FRIENDS

**Date:** 2026-04-28

## Problem

`CLASS foo DEFINITION DEFERRED.` and `CLASS foo DEFINITION LOCAL FRIENDS bar.` produce error nodes in the current tree-sitter grammar. Both forms are valid ABAP header-only statements that terminate with `.` and have no body or `ENDCLASS.`.

Root cause: `class_definition` always requires `ENDCLASS.`. Although `DEFERRED` and `LOCAL FRIENDS` appear in `_class_def_additions`, the outer rule structure forces `ENDCLASS.` even when they are used — making a correct parse impossible.

## Decision

Add two separate named grammar rules and remove `DEFERRED`/`LOCAL FRIENDS` from `_class_def_additions`.

### Grammar changes (`grammar.js`)

**New rules:**

```
class_deferred_definition
  → CLASS <identifier> DEFINITION DEFERRED .

class_local_friends_definition
  → CLASS <identifier> DEFINITION LOCAL FRIENDS <identifier>+ .
```

**Remove from `_class_def_additions`:**
- `kwSeq("DEFERRED")`
- `seq(kwSeq("LOCAL FRIENDS"), repeat1($.identifier))`

These never appear in a full class definition; their presence was a latent bug.

**Add to `_statement` choice** alongside the existing `class_definition` entry.

### Test changes (`test/corpus/class_definition.txt`)

Two new corpus entries:

```
CLASS zcl_foo DEFINITION DEFERRED.
→ (class_deferred_definition (identifier))

CLASS zcl_foo DEFINITION LOCAL FRIENDS zcl_bar zcl_baz.
→ (class_local_friends_definition (identifier) (identifier) (identifier))
```

## Alternatives considered

**Single `class_definition` as a `choice`:** Restructure the existing rule to handle all three forms. Rejected because the three forms have meaningfully different semantics and distinct structure; sharing one AST node type would make LSP queries harder.

## Scope

Grammar and corpus tests only. No LSP server changes required.
