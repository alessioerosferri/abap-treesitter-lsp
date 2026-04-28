# CLASS DEFINITION DEFERRED and LOCAL FRIENDS Grammar Support

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add two new named grammar rules so that `CLASS foo DEFINITION DEFERRED.` and `CLASS foo DEFINITION LOCAL FRIENDS bar.` parse without error nodes.

**Architecture:** Two new top-level rules (`class_deferred_definition`, `class_local_friends_definition`) are added to `grammar.js` alongside the existing `class_definition`. Both rules parse a header-only class statement that ends with `.` and has no body or `ENDCLASS.`. `DEFERRED` and `LOCAL FRIENDS` are removed from `_class_def_additions` where they were unreachable. Both rules are added to the `_statement` choice. Tests are added to the existing corpus file.

**Tech Stack:** tree-sitter (grammar.js), `tree-sitter test` for corpus testing.

---

### Task 1: Add corpus tests (failing)

**Files:**
- Modify: `test/corpus/class_definition.txt`

- [ ] **Step 1: Append two new test cases to the corpus file**

Add to the end of `test/corpus/class_definition.txt`:

```
================================================================================
Class definition deferred
================================================================================
CLASS zcl_foo DEFINITION DEFERRED.
--------------------------------------------------------------------------------

(source_file
  (class_deferred_definition
    (identifier)))

================================================================================
Class definition local friends
================================================================================
CLASS zcl_foo DEFINITION LOCAL FRIENDS zcl_bar zcl_baz.
--------------------------------------------------------------------------------

(source_file
  (class_local_friends_definition
    (identifier)
    (identifier)
    (identifier)))
```

- [ ] **Step 2: Run the tests to confirm they fail**

Run: `npm test`

Expected: failures mentioning `class_deferred_definition` and `class_local_friends_definition` not found (or the input producing `ERROR` nodes instead of the expected tree).

- [ ] **Step 3: Commit the failing tests**

```bash
git add test/corpus/class_definition.txt
git commit -m "test: add failing corpus tests for DEFINITION DEFERRED and LOCAL FRIENDS"
```

---

### Task 2: Add grammar rules and wire them up

**Files:**
- Modify: `grammar.js`

- [ ] **Step 1: Remove DEFERRED and LOCAL FRIENDS from `_class_def_additions`**

In `grammar.js`, locate `_class_def_additions` (around line 765). Remove these two lines:

```js
        kwSeq("DEFERRED"),
        seq(kwSeq("LOCAL FRIENDS"), repeat1($.identifier)),
```

The resulting `_class_def_additions` body should be:

```js
    _class_def_additions: ($) =>
      choice(
        seq(kw("INHERITING"), kw("FROM"), $.identifier),
        kw("ABSTRACT"),
        kw("FINAL"),
        seq(kw("CREATE"), choice(kw("PUBLIC"), kw("PROTECTED"), kw("PRIVATE"))),
        kwSeq("FOR TESTING"),
        seq(kwSeq("RISK LEVEL"), choice(kw("HARMLESS"), kw("DANGEROUS"), kw("CRITICAL"))),
        seq(kw("DURATION"), choice(kw("SHORT"), kw("MEDIUM"), kw("LONG"))),
        seq(kw("FRIENDS"), repeat1($.identifier)),
        kw("PUBLIC"),
      ),
```

- [ ] **Step 2: Add the two new rules after `class_definition`**

After the closing `,` of `class_definition` (around line 763), insert:

```js
    class_deferred_definition: ($) =>
      seq(
        kw("CLASS"), $.identifier, kw("DEFINITION"), kw("DEFERRED"), ".",
      ),

    class_local_friends_definition: ($) =>
      seq(
        kw("CLASS"), $.identifier, kw("DEFINITION"), kwSeq("LOCAL FRIENDS"), repeat1($.identifier), ".",
      ),
```

- [ ] **Step 3: Add both new rules to the `_statement` choice**

Locate the `_statement` choice (around line 261) where `$.class_definition` is listed. Add the two new rules immediately after it:

```js
        $.class_definition,
        $.class_deferred_definition,
        $.class_local_friends_definition,
        $.class_implementation,
```

- [ ] **Step 4: Run the tests to confirm they pass**

Run: `npm test`

Expected: all tests pass, including the two new corpus cases.

- [ ] **Step 5: Commit**

```bash
git add grammar.js
git commit -m "feat: add class_deferred_definition and class_local_friends_definition grammar rules"
```
