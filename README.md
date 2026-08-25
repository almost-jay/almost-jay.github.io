For each position, the current behaviour is described, followed by the open question about intended behaviour.

---

## 1. Binding

### `#name = expr`

Currently: binds a value (number, string, sequence, table). Cannot bind a function.

```
#x = 42          -- valid
#x = "hello"     -- valid
#x = $trim       -- INVALID: $trim is a function, not a #value
#x = #y => #y*2  -- INVALID: inline lambda produces a function
```

**Question:** Should `#name` be able to hold a function?

---

### `$name = funcExpr`

Currently: binds a function. `funcExpr` accepts an inline lambda, a pipeline fragment (`^ op`), or any expression.

```
$f = #x => #x * 2       -- valid: inline lambda
$f = $trim              -- valid: funcref as value
$f = ^ trim => upper    -- valid: pipeline fragment
$f = 42                 -- technically valid (42 is an expr), but useless
```

**Question:** Should `$name = 42` (binding a plain value to a function name) be valid?

---

### `$name : params -> body` (named lambda)

Currently: defines a named multi-arg lambda. Params are `#name` or `_`.

```
$f : #x -> #x * 2           -- valid
$f : #x, #y -> #x + #y      -- valid
$f : _ -> 42                 -- valid: ignored param
$f : $g -> ...               -- INVALID: param sigil must be #
```

**Question:** Should params be able to be functions (e.g. `$f : $g -> $g(42)`)?

---

## 2. Pipeline positions

### Input (`expr =>`)

Currently: the pipeline input is a `pipelineHead` expression — any expr except a bare inline lambda.

```
#data => $trim           -- valid: #value as input
"hello" => $trim         -- valid: literal as input
$trim => ...             -- valid: function as input (unusual but parseable)
(#x => #x + 1) => $map  -- valid: parenthesised lambda as input
#x => #x + 1 => $map    -- INVALID: ambiguous, must parenthesise
```

**Question:** What should `$trim => $upper` mean? Apply `$trim` to `$upper`? Compose them?

---

### Operation (`=> $name` / `=> name(args)`)

Currently: operations in a pipeline are either `$name`, `$name(args)`, `name`, or `name(args)`.

```
#data => $trim           -- valid: bare funcref
#data => $map($trim)     -- valid: funcref with funcref arg
#data => $map(#x => #x)  -- valid: funcref with inline lambda arg
#data => $map($f)        -- valid: funcref with funcref arg
#data => trim            -- valid: bare name (looked up in fns then defs)
```

**Question:** Can an operation be a `#value` that happens to hold a function? Currently no.

---

### `-> &name` (fragment binding)

Currently: only valid at end of a pipeline in a `::fragment::` block. Binds result to fragment namespace.

```
#data => $pre -> &output    -- valid
#data => $trim -> &output   -- valid (binds string result as fragment? questionable)
```

**Question:** Should `-> &name` be valid for any pipeline result, or only ones that produce a renderable?

---

## 3. Argument positions

### Function call args: `$f(arg, arg)`

Currently: args are `expr` — any expression including funcrefs, inline lambdas, values.

```
$map($trim)              -- valid: funcref as arg
$map(#x => #x * 2)      -- valid: inline lambda as arg
$map(42)                 -- valid: value as arg (useless but parseable)
$reduce(#a, #b -> #a + #b, 0)  -- INVALID: multi-arg inline lambda not supported
```

**Question:** Should multi-arg inline lambdas be valid in arg position?

---

### Named lambda params: `$f : #x, #y -> body`

Currently: params must use `#` sigil or `_`. Cannot accept function params.

```
$f : #x -> #x * 2           -- valid
$f : #f, #x -> #f(#x)       -- INVALID: #f is a value, not callable
$f : $g, #x -> $g(#x)       -- INVALID: param sigil must be #
```

**Question:** How do you pass a function as a named lambda parameter? Currently impossible without storing it in a `$` binding first.

---

### Capture arm transform: `^1 => $fn`

Currently: transform must be a bare `$name` with optional args.

```
^1 => $trim              -- valid
^1 => $map($upper)       -- valid: partially applied
^1 => #f                 -- INVALID: #value, not $fn
^1 => (#x => #x + 1)    -- INVALID: inline lambda not accepted here
```

**Question:** Should capture transforms accept any callable expression?

---

## 4. Match pattern positions

### `funcpred` pattern: `| $pred ->`

Currently: accepts only a bare `$name`. No args, no expressions.

```
| $is-positive -> ...    -- valid
| $gt(2) -> ...          -- INVALID: call expression not accepted
| $not($is-empty) -> ... -- INVALID: composed predicate not accepted
| (#x => #x > 0) -> ... -- INVALID: inline lambda not accepted
```

**Question:** Should `funcpred` accept any callable expression, including partial application and inline lambdas?

---

### Match arm result: `-> expr`

Currently: result is any `expr`. If it evaluates to a function, it is applied to the original input.

```
| _ -> "string"          -- valid: returns string directly
| _ -> 42               -- valid: returns number directly
| _ -> $trim            -- valid: $trim applied to input
| _ -> #x => #x * 2     -- INVALID: inline lambda in result position (ambiguous with ->)
| _ -> ($x => #x * 2)   -- valid if parenthesised? unclear
```

**Question:** Should inline lambdas be valid in arm result position (parenthesised)?

---

## 5. Operator positions

### Left operand

Currently: any expression. A `$name` as left operand is unusual — the operator semantics are left-type–driven, and a function as left operand has no defined behaviour for most operators.

```
#seq + #val              -- valid
"foo" * 3               -- valid
$trim + "foo"           -- undefined: function + string
$trim * #seq            -- undefined
```

**Question:** What should happen when a function is a left operand to an arithmetic operator?

---

### Right operand

Currently: as discussed, `$name` on the right triggers predicate/transformer mode (proposed, not yet implemented). Bare `$name` works; expressions don't.

```
#seq / $even            -- proposed: group-by
#seq / ?$even           -- proposed: partition
#seq / $gt(2)           -- INVALID currently: only bare $name
#seq / (#x => #x > 0)  -- INVALID: inline lambda
```

**Question:** Should the right-operand function position accept any callable expression?

---

## 6. Sequence and dict positions

### Sequence literal: `[a, b, c]`

Currently: elements are `expr`. A `$name` (funcref) is a valid expr and would be stored — but since `#` values can't hold functions, retrieving it later is impossible.

```
[1, 2, 3]               -- valid: sequence of numbers
[$trim, $upper]         -- parseable: sequence of funcrefs
                        -- but #seq[0] would give a funcref with no way to call it
```

**Question:** Should sequences be allowed to contain functions? If so, how do you call an element?

---

### Dict literal: `{ key: val }`

Currently: values are `expr`. Same issue as sequences.

```
{ a: 1, b: "hello" }        -- valid
{ trim: $trim, up: $upper } -- parseable: dict of functions
                             -- #dict[trim] gives a funcref, uncallable via #
```

**Question:** Should dicts be allowed to contain functions as values?

---

## 7. Template literal positions

### `\`...``interpolation:`${#expr}`

Currently: proposed syntax. `${}` evaluates an expression.

```
`hello ${#name}`         -- valid: #value interpolated
`result: ${$trim(#x)}`   -- valid: function call interpolated
`fn: ${$trim}`           -- valid: would interpolate the string representation of $trim?
```

**Question:** What is `${$trim}` — the string representation of a function? An error? Should functions be interpolatable?

---

## 8. Environment variable positions

### `%VAR% = expr`

Currently: proposed. Value can be any expression.

```
%PAD% = 5               -- valid: number
%FONT% = "monospace"    -- valid: string
%SCALE% = $double       -- valid?: function as env var
%OUT% => file.svg       -- special: output declaration
```

**Question:** Should env vars be able to hold functions?

---

## 9. `::plot::` block positions

### Function definition in plot context

Proposed: in `::plot::`, functions are written as `f(x) = expr` and all values are numeric.

```
f(x) = x**2 + 1         -- valid: polynomial
f(x) = $sin(x)          -- valid: stdlib math fn (no $ needed in ::plot::)
f(x) = #a * x + #b      -- valid: uses #values from ::def::
f(x) = $my-fn(x)        -- valid?: user-defined function used in plot
```

**Question:** Can user-defined `$fn` from `::def::` be used in `::plot::` context?

---

## 10. Continuation line positions

### `^ op` (buffer operation)

Currently: continuation lines apply operations to the preceding element. Operation is `$name(args)` or `name(args)`.

```
photo fill=blue
^ blur(4)                -- valid: named operation
^ $my-op(2)             -- valid?: user-defined operation
^ #f(4)                 -- INVALID: # not valid in op position
```

**Question:** Can user-defined `$fn` from `::def::` be used as a buffer operation?

---

## Summary of open questions

|Position|Currently accepts|Open question|
|---|---|---|
|`#name =`|values only|can `#` hold a function?|
|`$name =`|functions + any expr|should values be valid?|
|named lambda params|`#param` / `_` only|can params be functions (`$g`)?|
|pipeline input|any expr except bare lambda|what does `$fn => $fn` mean?|
|pipeline operation|`$name` / `name` / calls|can a `#value` holding a fn be an operation?|
|function call args|any expr|multi-arg inline lambdas?|
|capture transform|bare `$name` / `$name(args)`|any callable expr?|
|match `funcpred`|bare `$name` only|any callable expr?|
|match arm result|any expr|inline lambda (parenthesised)?|
|operator left|any expr|what does `$fn op val` mean?|
|operator right|value or bare `$name`|any callable expr?|
|sequence element|any expr (funcref parseable)|intended? callable elements?|
|dict value|any expr (funcref parseable)|intended? callable values?|
|template `${}`|any expr|what is `${$fn}` — string rep or error?|
|env var value|any expr|can env vars hold functions?|
|`::plot::` fn body|expr|user-defined `$fn` usable?|
|continuation `^` op|named operations|user-defined `$fn` as buffer op?|