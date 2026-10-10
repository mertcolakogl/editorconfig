# .editorconfig Reference

Every key in this project's `.editorconfig`, in the order the file declares them, with what the value does and — where a snippet earns its place — a labelled example. The 126 naming keys are grouped into the 19 rules they belong to rather than one entry each, and the file-type sections keep to a description because their indent settings are self-evident. Built for **Unity projects edited in Visual Studio Code**, enforced by Roslyn.

Copy the file into the root of a Unity project — the folder holding `Assets/`, `Packages/` and `ProjectSettings/` — and the editor picks it up with no plugin. The `.vscode/` folder beside it is what makes an editor actually apply what the file asks for; both files are described under **Editor integration** at the end.

```
your-unity-project/
├── .vscode/
│   ├── settings.json
│   └── tasks.json
├── Assets/
├── Packages/
├── ProjectSettings/
└── .editorconfig
```

Four things are worth knowing before reading the keys:

- **Sections are globs, and later sections win.** The file runs broad → narrow: `[*]`, file types, `[*.{cs,vb}]`, `[*.cs]`, then the vendor carve-out.
- **There are no inline comments.** A `#` starts a comment only as the first non-blank character of a line; anywhere else it is part of the value, so `severity = warning # cast` silently sets a value nothing recognises. Every comment in this file sits on its own line.
- **Severity suffixes.** `csharp_*` and `dotnet_style_*` values can end in `:error`, `:warning`, `:suggestion`, `:silent` or `:none`. `dotnet_diagnostic.<ID>.severity` takes the same list plus `default`. When an option carries its own suffix, that suffix wins over a `dotnet_diagnostic` line on .NET 9 and later; on older SDKs the suffix counts only inside the IDE and the explicit line is what a build sees. Code-style analysis is off during command-line builds unless `EnforceCodeStyleInBuild` is set to `true`.
- **Every example is labelled.** `Correct` is the code the setting asks for; the label under it names what the other group is. A plain value and a formatting key use `Correct` / `Wrong`; an option with a severity uses `Preferred` / `Suggestion`, `Correct` / `Warning` or `Correct` / `Error`; an option set to `none` or `silent` gets a `Preferred` example only, because a disabled rule reports nothing.

## `root = true`

Marks this as the top-most configuration file: editors stop searching parent directories for another `.editorconfig`.

```ini
# Correct
root = true

[*]
indent_style = space

# Wrong
# read as a property of the section above, not as the root marker
[*]
indent_style = space
root = true
```

## `[*]`

### `charset = utf-8`

Files are written as UTF-8 without a BOM, so non-ASCII text round-trips identically everywhere.

### `end_of_line = lf`

Line endings are `LF` only; a Windows editor cannot rewrite the whole file with `CRLF` and bury the real change in a diff.

### `indent_style = space`

Indentation uses spaces. A tab-indented file and a four-space file are otherwise indistinguishable in a diff.

### `indent_size = 4`

One indentation level is four columns wide.

### `tab_width = 4`

A literal tab character, if one ever appears, renders four columns wide — matching `indent_size` so alignment does not drift.

### `insert_final_newline = false`

Editors must not add a trailing newline on save. Deliberate: it keeps a tool from touching a byte the author did not.

### `trim_trailing_whitespace = false`

Editors must not strip trailing whitespace on save, for the same reason.

## `[*.{yml,yaml}]`

### `indent_style = space`

YAML is indentation-sensitive, so tabs are explicitly excluded.

### `indent_size = 2`

YAML nesting is two spaces per level.

## `[*.{json,jsonc,asmdef,asmref,inputactions}]`

### `indent_style = space`

Spaces again, for the JSON-shaped files Unity produces.

### `indent_size = 2`

Two spaces per level in JSON, assembly definitions, assembly references and Input System action maps.

## `[*.{meta,prefab,unity,asset,preset}]`

### `indent_style = space`

Unity's serialized YAML uses spaces; the setting spells it out so an editor cannot decide otherwise.

### `indent_size = 2`

Two spaces, matching what Unity itself writes. Anything else means every save produces a diff.

## `[*.{uss,uxml}]`

### `indent_style = space`

UI Toolkit stylesheets and markup are indented with spaces.

### `indent_size = 2`

Two spaces, matching the UI Builder's output.

## `[*.{shader,hlsl,hlslinc,cginc,glsl,glslinc,compute}]`

### `indent_style = space`

Shader source is code, so it follows the code convention.

### `indent_size = 4`

Four spaces, the same width as C# — a shader and the script that drives it read at the same depth.

## `[*.{cs,vb}]`

### `dotnet_code_quality_unused_parameters = all:suggestion`

Reports unused parameters on every method, not only on non-public ones. `all` is also why Unity message signatures that ignore an argument produce a hint.

```csharp
// Preferred
void Handle(int a) { Foo(a); }

// Suggestion
void Handle(int a, int b) { Foo(a); }   // IDE0060
```

### `dotnet_remove_unnecessary_suppression_exclusions = none`

Nothing is excluded from the "this suppression suppresses nothing" analysis, so every dead `#pragma` is reported. The option cannot carry a `:severity` suffix, which is why `IDE0079` gets a line of its own below.

```csharp
// Correct
#pragma warning disable IDE0051
private void Foo() { }   // the pragma still suppresses a real finding

// Wrong
#pragma warning disable IDE0051
public void Foo() { }   // IDE0079: nothing in scope is unused
```

### `dotnet_diagnostic.IDE0079.severity = warning`

Sets the severity for `IDE0079` (Remove unnecessary suppression). It flags `#pragma warning disable` directives and `SuppressMessage` attributes that no longer suppress anything — and it never runs on build, only in the IDE.

```csharp
// Correct
public void Foo() { }   // the unnecessary pragma is gone

// Warning
#pragma warning disable CS0168   // IDE0079: no unused local here
public void Foo() { }
```

### `dotnet_separate_import_directive_groups = false`

Usings form one block; no blank line is inserted between groups.

```csharp
// Correct
using System;
using Foo.Bar;

// Wrong
using System;

using Foo.Bar;
```

### `dotnet_sort_system_directives_first = true`

`System.*` usings come first whenever imports are sorted.

```csharp
// Correct
using System;
using Foo.Bar;

// Wrong
using Foo.Bar;
using System;
```

### `dotnet_diagnostic.IDE0055.severity = warning`

The single rule behind every formatting option in the `[*.cs]` section. Formatting options cannot carry a severity suffix of their own, so without this line the formatter preferences have no lever at all — including the two import options above.

```csharp
// Correct
public void Foo()
{
    Bar();
}

// Warning
public void Foo() { Bar(); }   // IDE0055
```

### `dotnet_style_allow_multiple_blank_lines_experimental = false:suggestion`

Two or more blank lines in a row are reported.

```csharp
// Preferred
public void Foo() { }

public void Bar() { }

// Suggestion
public void Foo() { }


public void Bar() { }   // two blank lines above
```

### `dotnet_style_allow_statement_immediately_after_block_experimental = false:suggestion`

A statement may not follow a closing brace without a blank line between them.

```csharp
// Preferred
if (flag) { Foo(); }

Console.WriteLine("text");

// Suggestion
if (flag) { Foo(); }
Console.WriteLine("text");   // no blank line above
```

### `dotnet_style_parentheses_in_arithmetic_binary_operators = never_if_unnecessary:suggestion`

Do not parenthesise arithmetic that precedence already handles.

```csharp
// Preferred
var result = a + b * c;

// Suggestion
var result = (a) + (b * c);
```

### `dotnet_style_parentheses_in_other_binary_operators = always_for_clarity:suggestion`

Keep parentheses when `&&` and `||` are mixed, so the reader does not have to recall precedence.

```csharp
// Preferred
var flag = (a && !b) || c;

// Suggestion
var flag = a && !b || c;
```

### `dotnet_style_parentheses_in_other_operators = never_if_unnecessary:suggestion`

Drop parentheses around operators that are not binary arithmetic or logic.

```csharp
// Preferred
var x = -value;

// Suggestion
var x = -(value);
```

### `dotnet_style_parentheses_in_relational_binary_operators = never_if_unnecessary:suggestion`

Comparison operators do not need help.

```csharp
// Preferred
if (x > 0 && y > 0) { }

// Suggestion
if ((x > 0) && (y > 0)) { }
```

### `dotnet_style_predefined_type_for_locals_parameters_members = true:suggestion`

Use the language keywords for types in declarations.

```csharp
// Preferred
int count = 5;
string name = "text";

// Suggestion
Int32 count = 5;
String name = "text";
```

### `dotnet_style_predefined_type_for_member_access = true:suggestion`

The same for static member access.

```csharp
// Preferred
var x = int.MaxValue;
var y = string.Empty;

// Suggestion
var x = Int32.MaxValue;
var y = String.Empty;
```

### `dotnet_style_prefer_collection_expression = when_types_loosely_match:suggestion`

Array and list creation can become a collection expression when the target type allows it.

```csharp
// Preferred
int[] items = [1, 2];
List<string> list = ["foo", "bar"];

// Suggestion
int[] items = new int[] { 1, 2 };
List<string> list = new List<string> { "foo", "bar" };
```

### `dotnet_style_prefer_foreach_explicit_cast_in_source = when_strongly_typed:suggestion`

When a `foreach` element type is strongly typed, write the cast rather than relying on an implicit conversion.

```csharp
// Preferred
foreach (Foo foo in items) { foo.Bar(); }

// Suggestion
foreach (var raw in items) { raw.Bar(); }
```

### `dotnet_style_prefer_auto_properties = true:suggestion`

A property with a trivial backing field becomes an auto-property.

```csharp
// Preferred
public int Value { get; set; }

// Suggestion
private int _value;
public int Value { get { return _value; } set { _value = value; } }
```

### `dotnet_style_prefer_compound_assignment = true:suggestion`

Compound assignment instead of repeating the target on the right.

```csharp
// Preferred
count += 10;
flag |= b;

// Suggestion
count = count + 10;
flag = flag | b;
```

### `dotnet_style_prefer_conditional_expression_over_assignment = true:suggestion`

An assignment that only branches between two values becomes a conditional expression.

```csharp
// Preferred
var name = flag ? "foo" : "bar";

// Suggestion
string name;
if (flag) { name = "foo"; } else { name = "bar"; }
```

### `dotnet_style_prefer_conditional_expression_over_return = true:suggestion`

A method that only branches between two returns becomes one return.

```csharp
// Preferred
public string Foo() => flag ? "foo" : "bar";

// Suggestion
public string Foo() { if (flag) { return "foo"; } else { return "bar"; } }
```

### `dotnet_style_prefer_non_hidden_explicit_cast_in_source = true:suggestion`

When a conversion happens without a visible cast, write the cast out.

```csharp
// Preferred
double x = (double)a / b;

// Suggestion
double x = a / b;
```

### `dotnet_style_prefer_inferred_anonymous_type_member_names = true:warning`

Anonymous types infer member names instead of repeating them.

```csharp
// Correct
var row = new { value.Name, value.Count };

// Warning
var row = new { Name = value.Name, Count = value.Count };
```

### `dotnet_style_prefer_inferred_tuple_names = true:warning`

The same for tuple elements.

```csharp
// Correct
var pair = (value.Name, value.Count);

// Warning
var pair = (Name: value.Name, Count: value.Count);
```

### `dotnet_style_prefer_is_null_check_over_reference_equality_method = true:warning`

`is null` instead of a call to `ReferenceEquals`.

```csharp
// Correct
if (value is null) return;

// Warning
if (ReferenceEquals(value, null)) return;
```

For `UnityEngine.Object` this suggestion is suppressed by `USP0021`: `is null` bypasses the overloaded equality operator, so a destroyed object would read as alive.

### `dotnet_style_prefer_simplified_boolean_expressions = true:suggestion`

Drop comparisons against boolean literals.

```csharp
// Preferred
if (flag && other) { }

// Suggestion
if (flag == true && other == true) { }
```

### `dotnet_style_prefer_simplified_interpolation = true:suggestion`

Interpolation does not need an explicit `ToString()`.

```csharp
// Preferred
var text = $"foo: {value}";

// Suggestion
var text = $"foo: {value.ToString()}";
```

### `dotnet_style_readonly_field = true:suggestion`

A private field that is only assigned in its declaration or a constructor should be `readonly`.

```csharp
// Preferred
private readonly int _count;

// Suggestion
private int _count;   // only ever assigned in the constructor
```

On serialized fields `USP0004` suppresses this: Unity assigns them from the Inspector, and a `readonly` field disappears from it.

### `dotnet_style_require_accessibility_modifiers = always:suggestion`

Every member states its accessibility, including interface members and explicit implementations.

```csharp
// Preferred
public void Foo() { }

// Suggestion
void Foo() { }   // IDE0040
```

### `dotnet_style_qualification_for_event = false:suggestion`

Events are not qualified with `this.`.

```csharp
// Preferred
Foo?.Invoke();

// Suggestion
this.Foo?.Invoke();
```

### `dotnet_style_qualification_for_field = false:suggestion`

Fields are not qualified with `this.`.

```csharp
// Preferred
_count++;

// Suggestion
this._count++;
```

### `dotnet_style_qualification_for_method = false:suggestion`

Methods are not qualified with `this.`.

```csharp
// Preferred
Foo();

// Suggestion
this.Foo();
```

### `dotnet_style_qualification_for_property = false:suggestion`

Properties are not qualified with `this.`.

```csharp
// Preferred
Name = "foo";

// Suggestion
this.Name = "foo";
```

### `dotnet_style_coalesce_expression = true:suggestion`

A null test that picks between two values becomes `??`.

```csharp
// Preferred
var name = input ?? "foo";

// Suggestion
var name = input != null ? input : "foo";
```

Suppressed for Unity objects by `USP0001` and `USP0022`, for the same equality-operator reason as above.

### `dotnet_style_collection_initializer = true:suggestion`

Collections are filled in the initializer rather than by repeated `Add` calls.

```csharp
// Preferred
var items = new List<string> { "foo", "bar" };

// Suggestion
var items = new List<string>();
items.Add("foo");
items.Add("bar");
```

### `dotnet_style_explicit_tuple_names = true:suggestion`

Tuple elements are named, so call sites do not read `Item1`.

```csharp
// Preferred
var row = (Name: "foo", Count: 1);
Foo(row.Name);

// Suggestion
var row = ("foo", 1);
Foo(row.Item1);
```

### `dotnet_style_namespace_match_folder = true:suggestion`

The namespace mirrors the folder path.

```csharp
// Preferred
// File: Foo/Bar/Baz.cs
namespace Foo.Bar
{
    class Baz { }
}

// Suggestion
// File: Foo/Bar/Baz.cs
namespace Foo
{
    class Baz { }
}
```

### `dotnet_style_null_propagation = true:suggestion`

A null check guarding a member access becomes `?.`.

```csharp
// Preferred
var text = input?.Name;

// Suggestion
var text = input != null ? input.Name : null;
```

Suppressed for Unity objects by `USP0002`.

### `dotnet_style_object_initializer = true:suggestion`

Object properties are set in the initializer.

```csharp
// Preferred
var foo = new Foo { Name = "value", Count = 1 };

// Suggestion
var foo = new Foo();
foo.Name = "value";
foo.Count = 1;
```

### `dotnet_style_operator_placement_when_wrapping = beginning_of_line`

When an expression wraps, the operator starts the continuation line. This one has no diagnostic behind it: it is a Visual Studio/Rider refactoring preference and neither analyzers nor `dotnet format` enforce it.

```csharp
// Correct
var flag = a
            && !b
            && !c;

// Wrong
var flag = a &&
            !b &&
            !c;
```

### `dotnet_prefer_system_hash_code = true:suggestion`

`HashCode.Combine` instead of hand-rolled hash arithmetic.

```csharp
// Preferred
public override int GetHashCode() => HashCode.Combine(Name, Count);

// Suggestion
public override int GetHashCode() => Name.GetHashCode() ^ Count.GetHashCode();
```

## Naming — `[*.{cs,vb}]`

Each entry below is one naming rule from the file, in declaration order, with the keys that make it up and one example. A rule has three parts: a **symbol group** that selects the symbols, a **naming style** that says what shape they take, and a **rule** that links the two and sets a severity. Rules are not applied in file order — **the more specific rule wins**, where specificity comes from modifiers, accessibility and kind, which is why the `const` groups list `const,static` together: a constant is implicitly static and would otherwise be caught by the static-field rules.

Every rule is at `severity = warning`, and a rule's own severity counts only inside the IDE — the global `IDE1006` line in the Diagnostics section below is what makes a naming violation appear on build and in `dotnet format`. Roslyn has no `enum_member` symbol kind, so no rule covers enum members; the `field` + `const` rules do not match them reliably either, so enforce enum member naming with an analyzer or suppressor if it matters.

### Constants (non-private) — `PascalCase`

Non-private `const` fields: `public`, `internal`, `protected`, `protected internal` and `private protected`. The `const,static` pair is deliberate — a `const` field is implicitly `static`, so requiring both makes this an intersection rule that outranks the non-private static-field rule.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.constants.symbols` | `constants` | Selects the `constants` symbol group. |
| `dotnet_naming_rule.constants.style` | `constant_style` | Applies the `constant_style` naming style. |
| `dotnet_naming_rule.constants.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.constants.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.constants.applicable_accessibilities` | `public,internal,protected,protected_internal,private_protected` | Every accessibility except `private` and `private_protected`; private constants have their own rule. |
| `dotnet_naming_symbols.constants.required_modifiers` | `const,static` | Both must be present. `const` implies `static`, and listing both is what makes this rule outrank the plain static-field rule. |
| `dotnet_naming_style.constant_style.capitalization` | `pascal_case` | PascalCase, and no prefix. |

```csharp
// Correct
public const int MaxValue = 1;

// Warning
public const int maxValue = 1;   // PascalCase expected
```

### Private constants — `PascalCase`

Private `const` fields, treated like public ones: PascalCase and no `_`. The same `const,static` intersection keeps a private constant out of the private field and private static field rules.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.private_constants.symbols` | `private_constants` | Selects the `private_constants` symbol group. |
| `dotnet_naming_rule.private_constants.style` | `private_constant_style` | Applies the `private_constant_style` naming style. |
| `dotnet_naming_rule.private_constants.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.private_constants.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.private_constants.applicable_accessibilities` | `private` | Private members only; the non-private constants rule covers the rest. |
| `dotnet_naming_symbols.private_constants.required_modifiers` | `const,static` | The `const,static` intersection again, so a private constant is not mistaken for a private field or a private static field. |
| `dotnet_naming_style.private_constant_style.capitalization` | `pascal_case` | PascalCase and no prefix — private constants are treated like public ones. |

```csharp
// Correct
private const int Count = 1;

// Warning
private const int _count = 1;   // PascalCase expected, and no `_` prefix
```

### Private static readonly fields — `PascalCase`

Private fields that are both `static` and `readonly`. This is what separates them from plain private static fields — those take `_`, these do not. Deliberate: a static readonly value is a constant in all but keyword.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.private_static_readonly_fields.symbols` | `private_static_readonly_fields` | Selects the `private_static_readonly_fields` symbol group. |
| `dotnet_naming_rule.private_static_readonly_fields.style` | `private_static_readonly_field_style` | Applies the `private_static_readonly_field_style` naming style. |
| `dotnet_naming_rule.private_static_readonly_fields.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.private_static_readonly_fields.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.private_static_readonly_fields.applicable_accessibilities` | `private` | Private members only. |
| `dotnet_naming_symbols.private_static_readonly_fields.required_modifiers` | `readonly,static` | Both `readonly` and `static`, which is what separates this group from plain private static fields — those take `_`, these do not. |
| `dotnet_naming_style.private_static_readonly_field_style.capitalization` | `pascal_case` | PascalCase, even though the field is private. Deliberate: a static readonly value is a constant in all but keyword. |

```csharp
// Correct
private static readonly string Name = "value";

// Warning
private static readonly string _name = "value";   // PascalCase expected
```

### Non-private static readonly fields — `PascalCase`

Static readonly fields that are `public`, `internal`, `protected`, `protected internal` or `private protected`.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.non_private_static_readonly_fields.symbols` | `non_private_static_readonly_fields` | Selects the `non_private_static_readonly_fields` symbol group. |
| `dotnet_naming_rule.non_private_static_readonly_fields.style` | `non_private_static_readonly_field_style` | Applies the `non_private_static_readonly_field_style` naming style. |
| `dotnet_naming_rule.non_private_static_readonly_fields.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.non_private_static_readonly_fields.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.non_private_static_readonly_fields.applicable_accessibilities` | `public,internal,protected,protected_internal,private_protected` | Every accessibility except the two private ones. |
| `dotnet_naming_symbols.non_private_static_readonly_fields.required_modifiers` | `readonly,static` | Both `readonly` and `static` must be present. |
| `dotnet_naming_style.non_private_static_readonly_field_style.capitalization` | `pascal_case` | PascalCase, and no prefix. |

```csharp
// Correct
public static readonly int MaxValue = 1;

// Warning
public static readonly int maxValue = 1;   // PascalCase expected
```

### Private static fields — `_camelCase`

Private static fields that are not `readonly`. Same `_` prefix as private instance fields, rather than the `s_` prefix other style guides use.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.private_static_fields.symbols` | `private_static_fields` | Selects the `private_static_fields` symbol group. |
| `dotnet_naming_rule.private_static_fields.style` | `private_static_field_style` | Applies the `private_static_field_style` naming style. |
| `dotnet_naming_rule.private_static_fields.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.private_static_fields.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.private_static_fields.applicable_accessibilities` | `private` | Private members only. |
| `dotnet_naming_symbols.private_static_fields.required_modifiers` | `static` | `static` must be present; a `readonly` static field is caught by the more specific rule above instead. |
| `dotnet_naming_style.private_static_field_style.capitalization` | `camel_case` | camelCase. |
| `dotnet_naming_style.private_static_field_style.required_prefix` | `_` | The name must start with `_` — the same prefix as private instance fields, rather than the `s_` other guides use. |

```csharp
// Correct
private static int _count;

// Warning
private static int count;   // the `_` prefix is missing
```

### Non-private static fields — `PascalCase`

Static fields that are `public`, `internal`, `protected`, `protected internal` or `private protected`. Unlike non-private instance fields, these stay PascalCase.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.non_private_static_fields.symbols` | `non_private_static_fields` | Selects the `non_private_static_fields` symbol group. |
| `dotnet_naming_rule.non_private_static_fields.style` | `non_private_static_field_style` | Applies the `non_private_static_field_style` naming style. |
| `dotnet_naming_rule.non_private_static_fields.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.non_private_static_fields.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.non_private_static_fields.applicable_accessibilities` | `public,internal,protected,protected_internal,private_protected` | Every accessibility except the two private ones. |
| `dotnet_naming_symbols.non_private_static_fields.required_modifiers` | `static` | `static` must be present and `readonly` must not — a `static readonly` field is PascalCase too, but through its own rule. |
| `dotnet_naming_style.non_private_static_field_style.capitalization` | `pascal_case` | PascalCase. Unlike non-private instance fields, static fields stay PascalCase. |

```csharp
// Correct
public static int Count;

// Warning
public static int count;   // PascalCase expected
```

### Private instance fields — `_camelCase`

Private fields that belong to an object — not static, not const. This is the rule most often met when a serialized field is named without the underscore.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.private_instance_fields.symbols` | `private_instance_fields` | Selects the `private_instance_fields` symbol group. |
| `dotnet_naming_rule.private_instance_fields.style` | `private_instance_field_style` | Applies the `private_instance_field_style` naming style. |
| `dotnet_naming_rule.private_instance_fields.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.private_instance_fields.applicable_kinds` | `field` | Fields only; properties, locals and parameters have their own rules. |
| `dotnet_naming_symbols.private_instance_fields.applicable_accessibilities` | `private` | Private members only. |
| `dotnet_naming_style.private_instance_field_style.capitalization` | `camel_case` | camelCase. |
| `dotnet_naming_style.private_instance_field_style.required_prefix` | `_` | The name must start with `_` — including serialized fields, which are still private and still in this group. |

```csharp
// Correct
private int _value;

// Warning
private int value;   // the `_` prefix is missing
```

### Non-private readonly instance fields — `PascalCase`

Instance fields that are `public`, `internal`, `protected`, `protected internal` or `private protected` and `readonly`. Deliberate: Unity never serializes a `readonly` field, so the Inspector convention behind the camelCase rule below does not apply — the value is fixed at construction, which is the same reasoning the static `readonly` rules above use. Requiring `readonly` is more specific than requiring nothing, so this rule outranks the plain non-private field rule.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.non_private_readonly_fields.symbols` | `non_private_readonly_fields` | Selects the `non_private_readonly_fields` symbol group. |
| `dotnet_naming_rule.non_private_readonly_fields.style` | `non_private_readonly_field_style` | Applies the `non_private_readonly_field_style` naming style. |
| `dotnet_naming_rule.non_private_readonly_fields.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.non_private_readonly_fields.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.non_private_readonly_fields.applicable_accessibilities` | `public,internal,protected,protected_internal,private_protected` | Every accessibility except the two private ones; a private field stays with `private_instance_fields`. |
| `dotnet_naming_symbols.non_private_readonly_fields.required_modifiers` | `readonly` | `readonly` must be present. A `static readonly` field is caught by the more specific static rules above instead. |
| `dotnet_naming_style.non_private_readonly_field_style.capitalization` | `pascal_case` | PascalCase, unlike the mutable non-private fields below. |

```csharp
// Correct
public readonly float Value;

// Warning
public readonly float value;   // PascalCase expected
```

### Non-private fields — `camelCase`

Instance fields that are `public`, `internal`, `protected`, `protected internal` or `private protected`. Deliberate: Microsoft's guidance is PascalCase, but these fields are visible in the Unity Inspector, and the serialized-field convention there is camelCase.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.non_private_fields.symbols` | `non_private_fields` | Selects the `non_private_fields` symbol group. |
| `dotnet_naming_rule.non_private_fields.style` | `non_private_field_style` | Applies the `non_private_field_style` naming style. |
| `dotnet_naming_rule.non_private_fields.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.non_private_fields.applicable_kinds` | `field` | Fields only. |
| `dotnet_naming_symbols.non_private_fields.applicable_accessibilities` | `public,internal,protected,protected_internal,private_protected` | Every accessibility except the two private ones, which fall to `private_instance_fields`. |
| `dotnet_naming_style.non_private_field_style.capitalization` | `camel_case` | camelCase rather than Microsoft's PascalCase. Deliberate: these fields are visible in the Unity Inspector, and the serialized-field convention there is camelCase. |

```csharp
// Correct
public float value;

// Warning
public float Value;   // camelCase expected
```

### Types and namespaces — `PascalCase`

Classes, structs, enums, delegates and namespaces. `interface` is deliberately absent — it has its own rule because of the `I` prefix.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.types_and_namespaces.symbols` | `types_and_namespaces` | Selects the `types_and_namespaces` symbol group. |
| `dotnet_naming_rule.types_and_namespaces.style` | `type_and_namespace_style` | Applies the `type_and_namespace_style` naming style. |
| `dotnet_naming_rule.types_and_namespaces.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.types_and_namespaces.applicable_kinds` | `class,delegate,enum,namespace,struct` | Classes, structs, enums, delegates and namespaces; `interface` is deliberately absent. |
| `dotnet_naming_symbols.types_and_namespaces.applicable_accessibilities` | `*` | `*` — the group is not filtered by accessibility at all. |
| `dotnet_naming_style.type_and_namespace_style.capitalization` | `pascal_case` | PascalCase for types and namespaces alike. |

```csharp
// Correct
public class Foo { }

// Warning
public class foo { }   // PascalCase expected
```

### Interfaces — `IPascalCase`

Interfaces only: a capital `I` followed by PascalCase.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.interfaces.symbols` | `interfaces` | Selects the `interfaces` symbol group. |
| `dotnet_naming_rule.interfaces.style` | `interface_style` | Applies the `interface_style` naming style. |
| `dotnet_naming_rule.interfaces.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.interfaces.applicable_kinds` | `interface` | Interfaces only. |
| `dotnet_naming_symbols.interfaces.applicable_accessibilities` | `*` | `*` — every accessibility. |
| `dotnet_naming_style.interface_style.capitalization` | `pascal_case` | PascalCase after the prefix. |
| `dotnet_naming_style.interface_style.required_prefix` | `I` | The name must start with a capital `I`. |

```csharp
// Correct
public interface IFoo { }

// Warning
public interface Foo { }   // the `I` prefix is missing
```

### Type parameters — `TPascalCase`

Generic type parameters: a capital `T` followed by PascalCase.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.type_parameters.symbols` | `type_parameters` | Selects the `type_parameters` symbol group. |
| `dotnet_naming_rule.type_parameters.style` | `type_parameter_style` | Applies the `type_parameter_style` naming style. |
| `dotnet_naming_rule.type_parameters.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.type_parameters.applicable_kinds` | `type_parameter` | Type parameters only. |
| `dotnet_naming_symbols.type_parameters.applicable_accessibilities` | `*` | `*` — every accessibility. |
| `dotnet_naming_style.type_parameter_style.capitalization` | `pascal_case` | PascalCase after the prefix. |
| `dotnet_naming_style.type_parameter_style.required_prefix` | `T` | The name must start with a capital `T`. |

```csharp
// Correct
public class Foo<TBar> { }

// Warning
public class Foo<Bar> { }   // the `T` prefix is missing
```

### Methods — `PascalCase`

Methods with any accessibility. Local functions are a separate group.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.methods.symbols` | `methods` | Selects the `methods` symbol group. |
| `dotnet_naming_rule.methods.style` | `method_style` | Applies the `method_style` naming style. |
| `dotnet_naming_rule.methods.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.methods.applicable_kinds` | `method` | Methods only; local functions are a separate group. |
| `dotnet_naming_symbols.methods.applicable_accessibilities` | `*` | `*` — every accessibility, including `private` and the implicit accessibility of interface members. |
| `dotnet_naming_style.method_style.capitalization` | `pascal_case` | PascalCase. |

```csharp
// Correct
public void Foo(int a) { }

// Warning
public void foo(int a) { }   // PascalCase expected
```

### Local functions — `PascalCase`

Functions declared inside another method, which are PascalCase even though the parameters they take are camelCase.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.local_functions.symbols` | `local_functions` | Selects the `local_functions` symbol group. |
| `dotnet_naming_rule.local_functions.style` | `local_function_style` | Applies the `local_function_style` naming style. |
| `dotnet_naming_rule.local_functions.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.local_functions.applicable_kinds` | `local_function` | Local functions only. |
| `dotnet_naming_symbols.local_functions.applicable_accessibilities` | `*` | `*` — a local function has no declared accessibility. |
| `dotnet_naming_style.local_function_style.capitalization` | `pascal_case` | PascalCase, unlike the parameters it takes. |

```csharp
// Correct
int Foo(int x) => x * 2;

// Warning
int foo(int x) => x * 2;   // PascalCase expected
```

### Properties — `PascalCase`

Properties. Fields keep their own rules.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.properties.symbols` | `properties` | Selects the `properties` symbol group. |
| `dotnet_naming_rule.properties.style` | `property_style` | Applies the `property_style` naming style. |
| `dotnet_naming_rule.properties.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.properties.applicable_kinds` | `property` | Properties only; fields keep their own rules. |
| `dotnet_naming_symbols.properties.applicable_accessibilities` | `*` | `*` — every accessibility. |
| `dotnet_naming_style.property_style.capitalization` | `pascal_case` | PascalCase. |

```csharp
// Correct
public int Value { get; set; }

// Warning
public int value { get; set; }   // PascalCase expected
```

### Events — `PascalCase`

Events.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.events.symbols` | `events` | Selects the `events` symbol group. |
| `dotnet_naming_rule.events.style` | `event_style` | Applies the `event_style` naming style. |
| `dotnet_naming_rule.events.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.events.applicable_kinds` | `event` | Events only. |
| `dotnet_naming_symbols.events.applicable_accessibilities` | `*` | `*` — every accessibility. |
| `dotnet_naming_style.event_style.capitalization` | `pascal_case` | PascalCase. |

```csharp
// Correct
public event Action Bar;

// Warning
public event Action bar;   // PascalCase expected
```

### Locals — `camelCase`

Local variables. A local has no declared accessibility, so no accessibility filter applies.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.locals.symbols` | `locals` | Selects the `locals` symbol group. |
| `dotnet_naming_rule.locals.style` | `local_style` | Applies the `local_style` naming style. |
| `dotnet_naming_rule.locals.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.locals.applicable_kinds` | `local` | Locals only. |
| `dotnet_naming_symbols.locals.applicable_accessibilities` | `*` | `*` — locals have none of their own. |
| `dotnet_naming_style.local_style.capitalization` | `camel_case` | camelCase. |

```csharp
// Correct
var name = "foo";

// Warning
var Name = "foo";   // camelCase expected
```

### Local constants — `camelCase`

Locals with `const`, which is what makes this rule outrank the plain `locals` rule. Deliberate: a local constant reads better alongside the ordinary locals it sits between.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.local_constants.symbols` | `local_constants` | Selects the `local_constants` symbol group. |
| `dotnet_naming_rule.local_constants.style` | `local_constant_style` | Applies the `local_constant_style` naming style. |
| `dotnet_naming_rule.local_constants.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.local_constants.applicable_kinds` | `local` | Locals — the `const` modifier below is what narrows it to constants. |
| `dotnet_naming_symbols.local_constants.applicable_accessibilities` | `*` | `*` — every accessibility. |
| `dotnet_naming_symbols.local_constants.required_modifiers` | `const` | `const` must be present, which is what makes this rule outrank the plain `locals` rule for a local constant. |
| `dotnet_naming_style.local_constant_style.capitalization` | `camel_case` | camelCase rather than PascalCase. Deliberate: a local constant reads better alongside the ordinary locals it sits between. |

```csharp
// Correct
const int maxValue = 3;

// Warning
const int MaxValue = 3;   // camelCase expected
```

### Parameters — `camelCase`

Method parameters.

| Key | Value | Effect |
|---|---|---|
| `dotnet_naming_rule.parameters.symbols` | `parameters` | Selects the `parameters` symbol group. |
| `dotnet_naming_rule.parameters.style` | `parameter_style` | Applies the `parameter_style` naming style. |
| `dotnet_naming_rule.parameters.severity` | `warning` | A violation is a warning. |
| `dotnet_naming_symbols.parameters.applicable_kinds` | `parameter` | Parameters only. |
| `dotnet_naming_symbols.parameters.applicable_accessibilities` | `*` | `*` — every accessibility. |
| `dotnet_naming_style.parameter_style.capitalization` | `camel_case` | camelCase. |

```csharp
// Correct
public void Foo(string name) { }

// Warning
public void Foo(string Name) { }   // camelCase expected
```

## Diagnostics — `[*.{cs,vb}]`

An `IDE*` rule takes its severity from the option that configures it — except these. The ones below either have no option at all (Roslyn ships none, so `dotnet_diagnostic.<ID>.severity` is the only lever) or have an option that cannot carry a suffix. They are listed in the order the file declares them.

### `dotnet_diagnostic.IDE1006.severity = warning`

The global severity for every naming rule above. A rule's own `severity` counts only inside an IDE; this line is what makes naming violations appear on build and in `dotnet format`.

```csharp
// Correct
private int _count;

// Warning
private int count;   // IDE1006
```

### `dotnet_diagnostic.IDE0001.severity = suggestion`

Shortens a type name that qualifies something already in scope. IDE-only: it never runs on build.

```csharp
// Preferred
List<int> items = new();

// Suggestion
System.Collections.Generic.List<int> items = new();
```

### `dotnet_diagnostic.IDE0002.severity = suggestion`

The same for member access — no need to name the type you are already inside. IDE-only.

```csharp
// Preferred
Bar();

// Suggestion
Foo.Bar();
```

### `dotnet_diagnostic.IDE0004.severity = warning`

A cast that changes nothing; removing it leaves the IL identical.

```csharp
// Correct
int count = list.Count;

// Warning
int count = (int)list.Count;   // IDE0004
```

### `dotnet_diagnostic.IDE0005.severity = warning`

A `using` directive nothing in the file needs. On a build it only appears when `<GenerateDocumentationFile>true</GenerateDocumentationFile>` is set; in the editor it works regardless.

```csharp
// Correct
using System;

// Warning
using System.Text;   // IDE0005 — nothing below touches System.Text
```

### `dotnet_diagnostic.IDE0010.severity = none`

Would demand every enum member of a `switch` statement. Off on purpose: enums in a game project keep growing.

```csharp
// Preferred
switch (state) { case Foo.Bar: return; }   // the remaining members are not demanded
```

### `dotnet_diagnostic.IDE0035.severity = none`

Unreachable code, which the compiler already reports as CS0162. Off to avoid saying it twice, and it is IDE-only anyway.

```csharp
// Preferred
return;

Foo("text");   // CS0162; IDE0035 stays off
```

### `dotnet_diagnostic.IDE0043.severity = warning`

A format placeholder with no matching argument, which would throw at run time.

```csharp
// Correct
string.Format("{0} {1}", a, b);

// Warning
string.Format("{0} {1}", a);   // IDE0043 — {1} has no argument
```

### `dotnet_diagnostic.IDE0050.severity = none`

An anonymous type with two or more fields that could be a tuple. IDE-only and off by default in Roslyn itself.

```csharp
// Preferred
var row = (a, b);
```

### `dotnet_diagnostic.IDE0051.severity = warning`

A private member nothing uses. Unity calls `Awake`, `Start`, `Update` and friends by reflection, which is why Microsoft.Unity.Analyzers ships `USP0003`, `USP0006`, `USP0008`, `USP0009`, `USP0010`, `USP0012` and `USP0019` to keep them out of this rule.

```csharp
// Correct
private int _value;

int GetFoo() { return _value; }

// Warning
private int _count;   // IDE0051
```

### `dotnet_diagnostic.IDE0052.severity = warning`

A private member that is written but never read. `USP0020` covers the members Unity reads for you.

```csharp
// Correct
private int _value;

int GetFoo() { return _value; }

// Warning
private int _value;

void Bar() { _value = 1; }   // IDE0052 — written, never read
```

### `dotnet_diagnostic.IDE0064.severity = suggestion`

A `readonly` field in a struct that assigns to `this`, which cannot work while the field stays readonly.

```csharp
// Preferred
public void Foo() { _value = 0; }

// Suggestion
public void Foo() { this = default; }   // IDE0064
```

### `dotnet_diagnostic.IDE0072.severity = suggestion`

A `switch` expression that does not handle every value of its input.

```csharp
// Preferred
var name = state switch { Foo.Bar => "foo", _ => "bar" };

// Suggestion
var name = state switch { Foo.Bar => "foo" };   // IDE0072 — other members unhandled
```

### `dotnet_diagnostic.IDE0076.severity = warning`

A global `SuppressMessage` whose `Scope` or `Target` no longer resolves — a suppression that suppresses nothing, usually after a rename.

```csharp
// Correct
[assembly: SuppressMessage("Style", "IDE0004", Scope = "member",
    Target = "~M:Foo.Bar(System.Single)")]

// Warning
[assembly: SuppressMessage("Style", "IDE0004", Scope = "member",
    Target = "~M:Foo.Baz(System.Single)")]   // IDE0076 — renamed to Bar
```

### `dotnet_diagnostic.IDE0077.severity = suggestion`

A global suppression still using the legacy FxCop target format. It works, but the legacy form is slow to process; the recommended form is the documentation ID.

```csharp
// Preferred
Target = "~F:Foo._value"

// Suggestion
Target = "Foo.#_value"        // IDE0077 — legacy
```

### `dotnet_diagnostic.IDE0080.severity = warning`

The null-forgiving `!` applied to something that is not nullable anyway.

```csharp
// Correct
var name = value.ToString();

// Warning
var name = value!.ToString();   // IDE0080
```

### `dotnet_diagnostic.IDE0081.severity = none`

An explicit `ByVal` on a parameter. Visual Basic only; a C# project never sees it.

```vb
' Preferred
Sub Foo(a As Single)
End Sub
```

### `dotnet_diagnostic.IDE0082.severity = warning`

`typeof(X).Name` where `nameof(X)` says the same thing and survives a rename.

```csharp
// Correct
Console.WriteLine(nameof(Foo));

// Warning
Console.WriteLine(typeof(Foo).Name);   // IDE0082
```

### `dotnet_diagnostic.IDE0100.severity = warning`

A comparison against a boolean literal.

```csharp
// Correct
if (flag) { }

// Warning
if (flag == true) { }   // IDE0100
```

### `dotnet_diagnostic.IDE0110.severity = warning`

A discard pattern that matches everything, so the check around it does nothing.

```csharp
// Correct
Foo(value);

// Warning
if (value is _) { Foo(value); }   // IDE0110
```

### `dotnet_diagnostic.IDE0120.severity = suggestion`

LINQ that walks further than it needs to.

```csharp
// Preferred
items.Any(x => x.Foo);

// Suggestion
items.Where(x => x.Foo).Any();
```

### `dotnet_diagnostic.IDE0121.severity = suggestion`

A LINQ type test followed by a cast, which `OfType<T>` does in one pass.

```csharp
// Preferred
items.OfType<Foo>();

// Suggestion
items.Where(x => x is Foo).Cast<Foo>();
```

### `dotnet_diagnostic.IDE0240.severity = suggestion`

A `#nullable` directive that restates the context it is already in.

```csharp
// Preferred
#nullable disable
string name;

// Suggestion
#nullable enable   // IDE0240
string name;
```

### `dotnet_diagnostic.IDE0241.severity = suggestion`

A `#nullable` directive that changes nothing, because no reference type is in its scope.

```csharp
// Preferred
enum Foo
{
    Bar,
    Baz
}

// Suggestion
#nullable disable   // IDE0241
enum Foo
{
    Bar,
    Baz
}
```

### `dotnet_diagnostic.IDE0280.severity = warning`

A member name written as a string literal where `nameof` would do — the literal goes stale on rename without a compiler error.

```csharp
// Correct
[NotNullIfNotNull(nameof(other))]
public string? Value { get; }

// Warning
[NotNullIfNotNull("other")]
public string? Value { get; }   // IDE0280
```

### `dotnet_diagnostic.IDE0370.severity = suggestion`

A `!` suppressing a nullable warning that is not being produced.

```csharp
// Preferred
var text = value.ToString();

// Suggestion
var text = value!.ToString();   // IDE0370
```

### `dotnet_diagnostic.IDE0380.severity = suggestion`

An `unsafe` modifier on a declaration that contains no unsafe operations.

```csharp
// Preferred
void Foo() { }

// Suggestion
unsafe void Foo() { }   // IDE0380
```

### `dotnet_diagnostic.IDE0390.severity = suggestion`

An `async` member with no `await`, which pays for a state machine it never uses.

```csharp
// Preferred
Task Foo() => Task.CompletedTask;

// Suggestion
async Task Foo() { return; }   // IDE0390
```

### `dotnet_diagnostic.IDE0391.severity = suggestion`

The same rule aimed at overrides and interface implementations, which `IDE0390` deliberately leaves alone.

```csharp
// Preferred
public override Task Foo() => Task.CompletedTask;

// Suggestion
public override async Task Foo() { return; }   // IDE0391
```

### `dotnet_diagnostic.IDE3000.severity = none`

The light-bulb suggestion on a method that throws `NotImplementedException`. An editor feature rather than a style finding, so it is off.

```csharp
// Preferred
public void Foo() => throw new NotImplementedException();
```

## `[*.cs]`

### `file_header_template = {fileName}\n\nCopyright (c) 2026 Mert Colakoglu. All rights reserved.`

Every C# file starts with that header. Line breaks are `\n` escapes because an `.editorconfig` value is a single line, and `{fileName}` is the only placeholder Roslyn supports — there are no date or author macros, so anything that changes per file cannot live in the template.

```csharp
// Correct
// Foo.cs
//
// Copyright (c) 2026 Mert Colakoglu. All rights reserved.

// Wrong
public class Foo { }
```

### `dotnet_diagnostic.IDE0073.severity = warning`

Severity for `IDE0073` (Require file header). The option above is an arbitrary string and cannot carry a suffix, so this line is the only lever. Seeding an existing project is a bulk job: `dotnet format style --diagnostics IDE0073` or "Fix all" in the editor.

```csharp
// Correct
// Foo.cs
//
// Copyright (c) 2026 Mert Colakoglu. All rights reserved.

// Warning
public class Foo { }   // IDE0073
```

### `csharp_indent_block_contents = true`

Statements inside a block are indented.

```csharp
// Correct
void Foo()
{
    Bar();
}

// Wrong
void Foo()
{
Bar();
}
```

### `csharp_indent_braces = false`

Braces sit at the level of the construct they belong to, not one level in.

```csharp
// Correct
void Foo()
{ Bar(); }

// Wrong
void Foo()
    { Bar(); }
```

### `csharp_indent_case_contents = true`

Statements under a `case` label are indented one level.

```csharp
// Correct
switch (state)
{
    case Foo.Bar:
        Bar();
        break;
}

// Wrong
switch (state)
{
    case Foo.Bar:
    Bar();
    break;
}
```

### `csharp_indent_case_contents_when_block = true`

A `case` that opens a block indents the block as well.

```csharp
// Correct
switch (state)
{
    case Foo.Bar:
        { Bar(); break; }
}

// Wrong
switch (state)
{
    case Foo.Bar:
    { Bar(); break; }
}
```

### `csharp_indent_switch_labels = true`

`case` and `default` are indented inside `switch`.

```csharp
// Correct
switch (state)
{
    case Foo.Bar: break;
    default: break;
}

// Wrong
switch (state)
{
case Foo.Bar: break;
default: break;
}
```

### `csharp_indent_labels = one_less_than_current`

A `goto` target sits one level to the left of the code around it.

```csharp
// Correct
void Foo()
{
    int i = 0;

retry:
    if (i++ < 3) goto retry;
}

// Wrong
void Foo()
{
    int i = 0;

    retry:
    if (i++ < 3) goto retry;
}
```

### `csharp_new_line_before_catch = true`

`catch` starts its own line.

```csharp
// Correct
try
{ Foo(); }
catch
{ Bar(); }

// Wrong
try
{ Foo(); } catch
{ Bar(); }
```

### `csharp_new_line_before_else = true`

`else` starts its own line.

```csharp
// Correct
if (flag)
{ Foo(); }
else
{ Bar(); }

// Wrong
if (flag)
{ Foo(); } else
{ Bar(); }
```

### `csharp_new_line_before_finally = true`

`finally` starts its own line.

```csharp
// Correct
try
{ Foo(); }
finally
{ Bar(); }

// Wrong
try
{ Foo(); } finally
{ Bar(); }
```

### `csharp_new_line_before_members_in_anonymous_types = true`

Members of an anonymous type each get their own line.

```csharp
// Correct
var row = new {
    Name = "foo"
};

// Wrong
var row = new { Name = "foo" };
```

### `csharp_new_line_before_members_in_object_initializers = false`

Initializer members may share a line when the initializer is short.

```csharp
// Correct
var obj = new Foo { Name = "foo" };

// Wrong
var obj = new Foo {
    Name = "foo"
};
```

### `csharp_new_line_before_open_brace = all`

Allman braces: every opening brace starts its own line.

```csharp
// Correct
void Foo()
{
    Bar();
}

// Wrong
void Foo() {
    Bar();
}
```

### `csharp_new_line_between_query_expression_clauses = true`

Each clause of a LINQ query expression goes on its own line.

```csharp
// Correct
var result = from x in items
    where flag
    select x;

// Wrong
var result = from x in items where flag select x;
```

### `csharp_prefer_braces = when_multiline:warning`

Braces are required only when the body spans more than one line, so a one-line guard clause stays legal.

```csharp
// Correct
if (flag) return;

// Warning
if (flag)
    Foo();   // IDE0011 — a multi-line body needs braces
```

### `dotnet_diagnostic.IDE0011.severity = warning`

Severity for `IDE0011` (Add braces). The option above does take a suffix, but an explicit line is required as a workaround for build enforcement ([roslyn#44201](https://github.com/dotnet/roslyn/issues/44201)); both are kept at `warning` so they never disagree.

```csharp
// Correct
if (flag)
{ Foo(); }

// Warning
if (flag)
    Foo();   // IDE0011
```

### `csharp_prefer_simple_default_expression = true:suggestion`

`default` instead of `default(T)` when the type is obvious.

```csharp
// Preferred
int count = default;

// Suggestion
int count = default(int);
```

### `csharp_prefer_simple_using_statement = true:suggestion`

A `using` declaration when the scope is the rest of the method.

```csharp
// Preferred
using var s = new Bar();

// Suggestion
using (var s = new Bar())
{ s.Baz(); }
```

### `csharp_prefer_static_anonymous_function = true:none`

Records that static lambdas are preferred, then silences the rule — worth doing, not worth nagging about.

```csharp
// Preferred
var result = static () => "foo";
```

### `csharp_prefer_static_local_function = true:none`

The same for local functions.

```csharp
// Preferred
static int Foo(int x) => x * 2;
```

### `csharp_prefer_system_threading_lock = true:suggestion`

Prefers `System.Threading.Lock` over locking on a bare object. .NET 9 added the type; Unity's runtime does not ship it, so the rule stays quiet until a project targets something that has it.

```csharp
// Preferred
private readonly Lock _value = new();   // .NET 9+

// Suggestion
private readonly object _value = new();
```

### `csharp_preferred_modifier_order = public, private, protected, internal, file, new, static, abstract, virtual, sealed, readonly, override, unsafe, extern, volatile, async, required:suggestion`

Modifiers are written in that order, most significant first. This is a custom order rather than the documented default; what matters is that it is a valid permutation of real C# modifiers.

```csharp
// Preferred
public static readonly int MaxValue = 100;

// Suggestion
public readonly static int MaxValue = 100;
```

### `csharp_preserve_single_line_blocks = true`

The formatter leaves a single-line block alone.

```csharp
// Correct
public void Foo() { }

// Wrong
public void Foo()
{
}
```

### `csharp_preserve_single_line_statements = true`

The formatter leaves a single-line statement alone.

```csharp
// Correct
if (flag) return;

// Wrong
if (flag)
    return;
```

### `csharp_space_after_cast = false`

No space between a cast and its operand.

```csharp
// Correct
var value = (int)source;

// Wrong
var value = (int) source;
```

### `csharp_space_after_colon_in_inheritance_clause = true`

One space after the colon of a base list.

```csharp
// Correct
public class Foo : Bar { }

// Wrong
public class Foo :Bar { }
```

### `csharp_space_after_comma = true`

One space after a comma.

```csharp
// Correct
Foo(x, y, z);

// Wrong
Foo(x , y , z);
```

### `csharp_space_after_dot = false`

No space after the member-access dot.

```csharp
// Correct
var name = obj.Name;

// Wrong
var name = obj . Name;
```

### `csharp_space_after_keywords_in_control_flow_statements = true`

One space between a control-flow keyword and its parenthesis.

```csharp
// Correct
if (flag) { }
for (int i = 0; i < n; i++) { }

// Wrong
if(flag) { }
for(int i = 0; i < n; i++) { }
```

### `csharp_space_after_semicolon_in_for_statement = true`

One space after each semicolon of a `for` header.

```csharp
// Correct
for (int i = 0; i < n; i++) { }

// Wrong
for (int i = 0;i < n;i++) { }
```

### `csharp_space_around_binary_operators = before_and_after`

A space on both sides of every binary operator.

```csharp
// Correct
var result = a + b * c;
if (x == y) { }

// Wrong
var result = a+b*c;
if (x==y) { }
```

### `csharp_space_around_declaration_statements = false`

No space before the semicolon of a declaration.

```csharp
// Correct
int x;

// Wrong
int x ;
```

### `csharp_space_before_colon_in_inheritance_clause = true`

One space before the colon of a base list.

```csharp
// Correct
public class Foo : Bar { }

// Wrong
public class Foo: Bar { }
```

### `csharp_space_before_comma = false`

No space before a comma.

```csharp
// Correct
Foo(x, y);

// Wrong
Foo(x , y);
```

### `csharp_space_before_dot = false`

No space before the member-access dot.

```csharp
// Correct
var name = obj.Name;

// Wrong
var name = obj. Name;
```

### `csharp_space_before_open_square_brackets = false`

No space before an indexer.

```csharp
// Correct
var value = items[0];

// Wrong
var value = items [0];
```

### `csharp_space_before_semicolon_in_for_statement = false`

No space before the semicolons of a `for` header.

```csharp
// Correct
for (int i = 0; i < n; i++) { }

// Wrong
for (int i = 0 ; i < n ; i++) { }
```

### `csharp_space_between_empty_square_brackets = false`

No space inside an empty pair of brackets.

```csharp
// Correct
int[] items;

// Wrong
int [] items;
```

### `csharp_space_between_method_call_empty_parameter_list_parentheses = false`

No space inside an empty argument list.

```csharp
// Correct
Foo();

// Wrong
Foo( );
```

### `csharp_space_between_method_call_name_and_opening_parenthesis = false`

No space between a method name and its opening parenthesis.

```csharp
// Correct
Foo(x);

// Wrong
Foo (x);
```

### `csharp_space_between_method_call_parameter_list_parentheses = false`

No space inside an argument list.

```csharp
// Correct
Foo(x, y);

// Wrong
Foo( x, y );
```

### `csharp_space_between_method_declaration_empty_parameter_list_parentheses = false`

No space inside an empty parameter list.

```csharp
// Correct
public void Foo() { }

// Wrong
public void Foo( ) { }
```

### `csharp_space_between_method_declaration_name_and_open_parenthesis = false`

No space between a method name and its opening parenthesis in a declaration.

```csharp
// Correct
public void Foo() { }

// Wrong
public void Foo () { }
```

### `csharp_space_between_method_declaration_parameter_list_parentheses = false`

No space inside a parameter list.

```csharp
// Correct
public void Foo(string name) { }

// Wrong
public void Foo( string name ) { }
```

### `csharp_space_between_parentheses = false`

No padding inside parentheses.

```csharp
// Correct
var value = (a + b) * c;

// Wrong
var value = ( a + b ) * c;
```

### `csharp_space_between_square_brackets = false`

No padding inside square brackets.

```csharp
// Correct
var x = items[0];

// Wrong
var x = items[ 0 ];
```

### `csharp_style_allow_embedded_statements_on_same_line_experimental = true:suggestion`

Allows a guard clause on one line, which is what `csharp_prefer_braces = when_multiline` already permits.

```csharp
// Preferred
if (flag) return;

// Suggestion
if (flag)
    return;
```

### `csharp_style_allow_blank_lines_between_consecutive_braces_experimental = false:suggestion`

Forbids a blank line between a closing and an opening brace.

```csharp
// Preferred
if (flag) { Foo(); }
else { Bar(); }

// Suggestion
if (flag) { Foo(); }

else { Bar(); }
```

### `csharp_style_allow_blank_line_after_colon_in_constructor_initializer_experimental = false:suggestion`

Forbids a blank line after the colon of a constructor initializer.

```csharp
// Preferred
public Foo(int a)
    : base(a)
{
}

// Suggestion
public Foo(int a)
    : base(a)

{
}
```

### `csharp_style_allow_blank_line_after_token_in_conditional_expression_experimental = false:suggestion`

Forbids a blank line after `?` or `:` in a conditional expression.

```csharp
// Preferred
var text = flag
    ? "foo"
    : "bar";

// Suggestion
var text = flag
    ? "foo"

    : "bar";
```

### `csharp_style_allow_blank_line_after_token_in_arrow_expression_clause_experimental = false:suggestion`

Forbids a blank line after `=>`.

```csharp
// Preferred
public int Value =>
    _value;

// Suggestion
public int Value =>

    _value;
```

### `csharp_style_conditional_delegate_call = true:suggestion`

A null check followed by a delegate call becomes a conditional call.

```csharp
// Preferred
Foo?.Invoke();

// Suggestion
if (Foo != null) { Foo(); }
```

### `csharp_style_deconstructed_variable_declaration = true:suggestion`

Deconstruct in the declaration rather than assigning each part afterwards.

```csharp
// Preferred
var (name, count) = GetFoo();

// Suggestion
string name;
int count;
(name, count) = GetFoo();
```

### `csharp_style_expression_bodied_accessors = false:suggestion`

Property accessors keep block bodies.

```csharp
// Preferred
public int Value { get { return _value; } set { _value = value; } }

// Suggestion
public int Value { get => _value; set => _value = value; }
```

### `csharp_style_expression_bodied_constructors = false:suggestion`

Constructors keep block bodies.

```csharp
// Preferred
public Foo(int a) { _value = a; }

// Suggestion
public Foo(int a) => _value = a;
```

### `csharp_style_expression_bodied_indexers = false:suggestion`

Indexers keep block bodies.

```csharp
// Preferred
public Foo this[int x] { get { return items[x]; } }

// Suggestion
public Foo this[int x] => items[x];
```

### `csharp_style_expression_bodied_lambdas = false:suggestion`

Lambdas keep block bodies, like every other member kind in the family.

```csharp
// Preferred
Func<int, int> square = x => { return x * x; };

// Suggestion
Func<int, int> square = x => x * x;
```

### `csharp_style_expression_bodied_local_functions = false:suggestion`

Local functions keep block bodies.

```csharp
// Preferred
int Foo(int x) { return x * 2; }

// Suggestion
int Foo(int x) => x * 2;
```

### `csharp_style_expression_bodied_methods = false:suggestion`

Methods keep block bodies.

```csharp
// Preferred
public int Add(int a, int b) { return a + b; }

// Suggestion
public int Add(int a, int b) => a + b;
```

### `csharp_style_expression_bodied_operators = false:suggestion`

Operators keep block bodies.

```csharp
// Preferred
public static Foo operator +(Foo a, Foo b) { return new Foo(); }

// Suggestion
public static Foo operator +(Foo a, Foo b) => new Foo();
```

### `csharp_style_expression_bodied_properties = false:suggestion`

Properties keep block bodies.

```csharp
// Preferred
public int Value { get { return _value; } }

// Suggestion
public int Value => _value;
```

### `csharp_style_implicit_object_creation_when_type_is_apparent = true:suggestion`

Target-typed `new()` where the type is already written on the left.

```csharp
// Preferred
List<int> list = new();

// Suggestion
List<int> list = new List<int>();
```

### `csharp_style_inlined_variable_declaration = true:suggestion`

Declare the variable inside the `out` or `is` expression rather than before it.

```csharp
// Preferred
if (int.TryParse(input, out int result)) { }

// Suggestion
int result;
if (int.TryParse(input, out result)) { }
```

### `csharp_style_namespace_declarations = block_scoped:suggestion`

Braced namespaces, not file-scoped ones — the whole file keeps one indentation level.

```csharp
// Preferred
namespace Foo
{
    class Bar { }
}

// Suggestion
namespace Foo;

class Bar { }
```

### `csharp_style_pattern_matching_over_as_with_null_check = true:suggestion`

Replace `as` followed by a null check with a pattern.

```csharp
// Preferred
if (obj is string s) { Bar(s); }

// Suggestion
var s = obj as string;
if (s != null) { Bar(s); }
```

### `csharp_style_pattern_matching_over_is_with_cast_check = true:suggestion`

Replace an `is` test followed by a cast with a pattern.

```csharp
// Preferred
if (obj is string s) { Bar(s); }

// Suggestion
if (obj is string) { Bar((string)obj); }
```

### `csharp_style_prefer_extended_property_pattern = true:suggestion`

Nested property patterns collapse into a dotted one.

```csharp
// Preferred
if (foo is { Bar.Baz: "x" }) { }

// Suggestion
if (foo is { Bar: { Baz: "x" } }) { }
```

### `csharp_style_prefer_implicitly_typed_lambda_expression = true:suggestion`

Drop a lambda parameter type the compiler can infer.

```csharp
// Preferred
Action<int> a = x => { };

// Suggestion
Action<int> a = (int x) => { };
```

### `csharp_style_prefer_index_operator = true:suggestion`

Index from the end with `^` rather than subtracting.

```csharp
// Preferred
var x = buffer[^1];

// Suggestion
var x = buffer[buffer.Length - 1];
```

### `csharp_style_prefer_labeled_jump_statements = true:suggestion`

Replaces a `goto` or a boolean flag used to break out of nested loops with a labeled `break`/`continue`. Labeled jumps are C# 15, so the rule stays quiet in a Unity project on an older language version.

```csharp
// Preferred
Foo: for (int x = 0; x < 10; x++)
    for (int y = 0; y < 10; y++)
        if (x * y > 20) break Foo;

// Suggestion
bool flag = false;
for (int x = 0; x < 10 && !flag; x++)
    for (int y = 0; y < 10 && !flag; y++) { if (x * y > 20) flag = true; }
```

### `csharp_style_prefer_local_over_anonymous_function = true:suggestion`

A local function instead of a lambda stored in a delegate.

```csharp
// Preferred
int Foo(int x) => x * 2;

// Suggestion
Func<int, int> a = x => x * 2;
```

### `csharp_style_prefer_method_group_conversion = true:suggestion`

Pass the method group rather than wrapping it in a lambda.

```csharp
// Preferred
items.ForEach(Handle);

// Suggestion
items.ForEach(x => Handle(x));
```

### `csharp_style_prefer_not_pattern = true:suggestion`

`is not null` instead of negating an `is` expression.

```csharp
// Preferred
if (value is not null) { }

// Suggestion
if (!(value is null)) { }
```

### `csharp_style_prefer_null_check_over_type_check = true:warning`

Use a null check where a type check was standing in for one.

```csharp
// Correct
if (value is null) return;

// Warning
if (value is object) return;
```

### `csharp_style_prefer_pattern_matching = true:suggestion`

Prefer pattern syntax wherever it fits.

```csharp
// Preferred
if (x is > 0 and < 100) { }

// Suggestion
if (x > 0 && x < 100) { }
```

### `csharp_style_prefer_primary_constructors = true:suggestion`

A class whose constructor only assigns fields can take a primary constructor.

```csharp
// Preferred
public class Foo(int value) { public int Value { get; } = value; }

// Suggestion
public class Foo
{
    public Foo(int value) => Value = value;
    public int Value { get; }
}
```

### `csharp_style_prefer_range_operator = true:suggestion`

A slice becomes a range expression.

```csharp
// Preferred
var slice = items[1..4];

// Suggestion
var slice = items.Skip(1).Take(3);
```

### `csharp_style_prefer_readonly_struct = true:suggestion`

A struct that never mutates is declared `readonly`.

```csharp
// Preferred
public readonly struct Foo { public readonly int Value; }

// Suggestion
public struct Foo { public int Value; }

```

### `csharp_style_prefer_readonly_struct_member = true:suggestion`

A member of a non-readonly struct that never mutates is marked `readonly`.

```csharp
// Preferred
public readonly int Value => Count * 2;

// Suggestion
public int Value => Count * 2;
```

### `csharp_style_prefer_simple_property_accessors = true:suggestion`

An accessor that only returns or assigns the `field` keyword becomes a simple auto-accessor. C# 13 and later, so it does not fire in a Unity project on an older language version.

```csharp
// Preferred
public int Value
{
    get;
    set { field = value; }
}

// Suggestion
private int _value;
public int Value
{
    get => _value;
    set => _value = value;
}
```

### `csharp_style_prefer_switch_expression = true:suggestion`

A `switch` statement that only assigns becomes a switch expression.

```csharp
// Preferred
var name = state switch { Foo.Bar => "foo", _ => "bar" };

// Suggestion
string name;
switch (state)
{
    case Foo.Bar: name = "foo"; break;
    default: name = "bar"; break;
}
```

### `csharp_style_prefer_top_level_statements = true:silent`

Records that top-level statements are preferred, then silences the rule: every Unity script is a type.

```csharp
// Preferred
// entry point style code — not used in a library
Console.WriteLine("foo");
```

### `csharp_style_prefer_tuple_swap = true:suggestion`

Swap through a tuple instead of a temporary.

```csharp
// Preferred
(a, b) = (b, a);

// Suggestion
var t = a; a = b; b = t;
```

### `csharp_style_prefer_unbound_generic_type_in_nameof = true:suggestion`

An unbound generic survives a change of type argument.

```csharp
// Preferred
var name = nameof(List<>);

// Suggestion
var name = nameof(List<int>);
```

### `csharp_style_prefer_utf8_string_literals = true:suggestion`

A UTF-8 literal where bytes are what is wanted.

```csharp
// Preferred
ReadOnlySpan<byte> text = "foo"u8;

// Suggestion
ReadOnlySpan<byte> text = Encoding.UTF8.GetBytes("foo");
```

### `csharp_style_throw_expression = true:suggestion`

Throw as part of an expression instead of inside a guarded block.

```csharp
// Preferred
_value = value ?? throw new ArgumentNullException(nameof(value));

// Suggestion
if (value == null) throw new ArgumentNullException(nameof(value));
_value = value;
```

### `csharp_style_unused_value_assignment_preference = discard_variable:suggestion`

An assignment whose value is never read is rewritten to a discard.

```csharp
// Preferred
_ = Foo();

// Suggestion
var unused = Foo();
```

### `csharp_style_unused_value_expression_statement_preference = discard_variable:warning`

The same for a bare expression used as a statement.

```csharp
// Correct
_ = TryGet(out var ignored);

// Warning
TryGet(out var ignored);
```

### `csharp_style_var_elsewhere = true:suggestion`

`var` is preferred wherever the type is not obvious from the right-hand side.

```csharp
// Preferred
var obj = Foo();

// Suggestion
Foo obj = Foo();
```

### `csharp_style_var_for_built_in_types = true:suggestion`

`var` for built-in types.

```csharp
// Preferred
var count = 42;

// Suggestion
int count = 42;
```

### `csharp_style_var_when_type_is_apparent = true:suggestion`

`var` where the right-hand side states the type.

```csharp
// Preferred
var list = new List<string>();

// Suggestion
List<string> list = new List<string>();
```

### `csharp_using_directive_placement = outside_namespace:error`

Usings go above the namespace, not inside it. This is the only style rule in the file at `error`, so a misplaced using fails the build.

```csharp
// Correct
using System;

namespace Bar
{
    public class Foo { }
}

// Error
namespace Bar
{
    using System;
    public class Foo { }
}
```

## `[Assets/Plugins/Sirenix/**/*.cs]`

### `generated_code = true`

Marks third-party code as generated, which removes it from **all** analyzer diagnostics — `IDE*`, `CA*` and Unity's own `UNT*` — while compiler diagnostics (`CS*`) still run, so a package that breaks compilation is still visible. Roslyn already detects files named `*.designer.cs`, `*.generated.cs`, `*.g.cs` or carrying an `<auto-generated>` header; this line covers vendored assets that carry no such marker. Use `generated_code = false` for the opposite case, a file that looks generated but is maintained by hand.

```csharp
// Correct
// Assets/Plugins/Sirenix/**/*.cs — out of IDE*, CA* and UNT*, CS* still runs
public class Foo { }

// Wrong
// hand-maintained sibling without the marker — restyled and reported
public class Foo { public int value; }
```

## Recommended tooling

| Component | Why it is needed |
|---|---|
| **VS Code** | Reads `[*]` and the file-type sections natively: charset, line endings, indentation. No plugin required. |
| **[C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)** and the **[C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)** extension | Run the Roslyn language server, which is what applies the `IDE*` rules, the naming rules and the `dotnet_style_*` preferences inside the editor. Without it only the formatting sections do anything. |
| **[Unity](https://marketplace.visualstudio.com/items?itemName=VisualStudioToolsForUnity.vstuc)** extension | Unity integration, the debugger, and the `Microsoft.Unity.Analyzers.dll` that ships in its `Analyzers/` folder. |
| **.NET SDK 10** (or any SDK 9.0.200 or later) | `dotnet format`, `dotnet build`, and support for the `.slnx` solution files Unity 6 generates. |
| **Unity `Visual Studio Editor` package 2.0.20 or later** | Generates the project files and wires the Unity analyzers into them with an `<Analyzer Include>` entry. |

The suppressors that keep `IDE0044`, `IDE0051`, `IDE0052`, `IDE0060`, `IDE0029`, `IDE0031`, `IDE0041` and `IDE1006` quiet on Unity's reflection-driven code come from that analyzer DLL. If one of those rules starts firing on a serialized field or an `Awake` method, the analyzer reference is what to check: regenerate the project files from Unity's External Tools preferences.

On an existing project that lights up in the hundreds — `IDE0055` for formatting, `IDE0073` for the missing header, `IDE1006` for naming. Fixing it is a command-line job:

```bash
dotnet format whitespace                       # the IDE0055 family
dotnet format style --diagnostics IDE0073      # add the file header everywhere
dotnet build -p:EnforceCodeStyleInBuild=true   # see style rules as build output
dotnet format --verify-no-changes              # CI gate
```

Two facts answer most "why is this not firing" questions. Code-style analysis is **disabled during command-line builds** unless `EnforceCodeStyleInBuild` is `true`, so `IDE0051` and friends are effectively editor-only; and Unity's own compilation never sets that property, so `IDE*` warnings reach the VS Code Problems panel but never the Unity Console. **Compiler diagnostics are the other way round.** `dotnet_diagnostic.CSxxxx.severity` controls them, they have no `csharp_*` or `dotnet_*` counterpart, they already default to `warning` — so writing `= warning` for one is a no-op — and they do appear in both places. The only ones worth an entry in a Unity project are the unused-code warnings, and they belong in the project copy of the file rather than a shared template:

```ini
# Locals — no IDE rule covers these, and they reach the Unity Console
dotnet_diagnostic.CS0168.severity = none   # declared, never used
dotnet_diagnostic.CS0219.severity = none   # assigned, never used
# Fields — IDE0051/IDE0052 cover these in the editor; add these only if the
# Unity Console should be quiet too
dotnet_diagnostic.CS0414.severity = none   # assigned, never read
dotnet_diagnostic.CS0169.severity = none   # never used
```

## Editor integration — `.vscode/`

A `.editorconfig` states rules; it does not run them and it cannot apply them. Both halves of that live in editor settings, so they ship as a folder meant to be copied next to the `.editorconfig`. Copy the folder as a whole rather than retyping the keys — the labels in `tasks.json` are referenced by name from the composite task, so renaming one breaks the chain.

### `settings.json`

| Key | Value | Effect |
|---|---|---|
| `dotnet.backgroundAnalysis.compilerDiagnosticsScope` | `fullSolution` | Compiler diagnostics (`CS*`) run over the whole solution instead of only the files you have open, which is the default. A `CS0168` in a file you never opened is otherwise invisible until you open it. The cost is background CPU and memory on a large project. |
| `dotnet.backgroundAnalysis.analyzerDiagnosticsScope` | `fullSolution` | The same for analyzer diagnostics — every `IDE*`, `CA*` and `UNT*` rule this `.editorconfig` configures. `openFiles` is the usual reason a violation "does not appear" at all. |
| `dotnet.diagnostics.reportInformationAsHint` | `false` | Reports information-level diagnostics as information instead of demoting them to hints, which appear only inside the code-action popup. This is what puts the 78 `:suggestion` options in this `.editorconfig` in front of you, as information rather than a yellow warning — the contract `.editorconfig` writes for `suggestion`. |
| `dotnet.formatting.organizeImportsOnFormat` | `true` | A format also groups and sorts `using` directives, which is what carries `dotnet_sort_system_directives_first` and `dotnet_separate_import_directive_groups` into a manual **Format Document**, not only into a save. |
| `[csharp].editor.codeActionsOnSave.source.fixAll` | `explicit` | On save, run every fix the language server offers — the `IDE*` rules set to `warning`, `IDE0005` and `IDE0055` among them. `explicit` waits for a save you trigger yourself (`Ctrl`+`S`) and stays out of an auto-save, so a background save never rewrites the file mid-keystroke. |
| `[csharp].editor.codeActionsOnSave.source.organizeImports` | `explicit` | The same for `using` directives. On a save this repeats `organizeImportsOnFormat` harmlessly; on a manual **Format Document** the two are the only reason the directives move at all. |
| `[csharp].editor.defaultFormatter` | `ms-dotnettools.csharp` | The Roslyn language server formats the file rather than a second formatter with its own idea of the style. |
| `[csharp].editor.formatOnSave` | `true` | Saving formats. |
| `[csharp].editor.formatOnSaveMode` | `file` | Formats the whole file rather than only the lines that changed. Under a shared `.editorconfig` the untouched lines should already conform, so reformatting them costs nothing and repairs drift. |

```json
{
  "dotnet.backgroundAnalysis.compilerDiagnosticsScope": "fullSolution",
  "dotnet.backgroundAnalysis.analyzerDiagnosticsScope": "fullSolution",
  "dotnet.diagnostics.reportInformationAsHint": false,
  "dotnet.formatting.organizeImportsOnFormat": true,
  "[csharp]": {
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit",
      "source.organizeImports": "explicit"
    },
    "editor.defaultFormatter": "ms-dotnettools.csharp",
    "editor.formatOnSave": true,
    "editor.formatOnSaveMode": "file"
  }
}
```

`dotnet.defaultSolution` is deliberately absent: it names one solution, and this file is meant to be copied into a project whose solution has a different name — or which has no solution until Unity generates one.

### `tasks.json`

`dotnet format` is the command-line twin of the formatter and the only way to apply the rules the language server merely reports. Five tasks wrap it — **Terminal → Run Task**, or `Tasks: Run Task` in the command palette:

| Task | Runs | Needs a solution |
|---|---|---|
| **Task Runner: Reformat Code** | `dotnet format whitespace --folder .` | no — `--folder` needs neither `.sln` nor `.csproj`, which is what makes it usable in a Unity project whose project files have not been generated |
| **Task Runner: Apply Syntax Style** | `dotnet format style <solution> --severity info` | yes |
| **Task Runner: Reformat & Apply Syntax Style** | the two above, in order | no for the first half, yes for the second |
| **Task Runner: Full Cleanup** | `dotnet format <solution> --severity info` | yes |
| **Task Runner: Full Cleanup (verify)** | the same with `--verify-no-changes` | yes |

`--severity info` is the part that matters here: 78 of the options in this `.editorconfig` carry `:suggestion`, and anything below `info` is filtered out of the run. `whitespace` is the `IDE0055` family, and every task passes the same `--exclude` list so that `Library/`, `Temp/`, `Logs/`, `obj/`, `Build/` and `UserSettings/` stay out of the sweep. The verify task writes nothing and exits non-zero when a change is needed, which is what a pre-commit hook or a CI step wants.

`<solution>` is `${workspaceFolder}/${workspaceFolderBasename}.slnx`, the name Unity 6 gives the solution it generates. The extension is the one thing to check when copying these tasks: a project whose solution is still a plain `.sln` — an older Unity, or a hand-written .NET solution — needs that string changed back to `.sln`. `dotnet format` reads either form on SDK 9.0.200 and later, and `Reformat Code` makes no assumption at all, because `--folder` works without a solution.

## References

- [editorconfig.org](https://editorconfig.org) and the [EditorConfig specification](https://spec.editorconfig.org/) — file format, globs, and the "no inline comments" rule
- [Code-style rules overview](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/) — every `IDE*` rule and the option behind it
- [Language rules](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/language-rules) — the `value:severity` option format used throughout this file
- [Configuration options](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/configuration-options) — severity levels, `dotnet_diagnostic` precedence, `generated_code`
- [Code-style naming rules](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/naming-rules) — symbol groups, naming styles, and why specific rules outrank general ones
- [IDE0055: Fix formatting](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0055) — the one rule behind every formatting option
- [IDE0073: Require file header](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0073) — `file_header_template` and `{fileName}`
- [IDE0079: Remove unnecessary suppression](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0079) — why the exclusion option needs a separate severity line
- [roslyn#44201](https://github.com/dotnet/roslyn/issues/44201) — why `IDE0011` needs an explicit severity line for build enforcement
- [roslyn#72238](https://github.com/dotnet/roslyn/pull/72238) — from .NET 9 on, an option's own severity wins over a `dotnet_diagnostic` line
- [roslyn#44362](https://github.com/dotnet/roslyn/issues/44362) — the performance problem behind the legacy `SuppressMessage` target format
- [Microsoft.Unity.Analyzers](https://github.com/microsoft/Microsoft.Unity.Analyzers) and its [diagnostic index](https://github.com/microsoft/Microsoft.Unity.Analyzers/blob/main/doc/index.md) — the `UNT*` analyzers and `USP*` suppressors named above
- [Unity: Serialization rules](https://docs.unity3d.com/Manual/script-serialization-rules.html) — why a serialized field must not be `readonly`, and why renaming one needs `[FormerlySerializedAs]`