# .editorconfig Reference

> **A complete rule-by-rule breakdown of the project's `.editorconfig` with C# examples.**  
> Developed for **Unity C# game development with Visual Studio Code**.  
> Covers formatting, naming conventions, style preferences, and IDE diagnostics.

The `.editorconfig` format is an open standard maintained by the EditorConfig project. For the full specification and a list of supported editors, see **[editorconfig.org](https://editorconfig.org)**.

---

## Installation

Copy `.editorconfig` to the root of your Unity project — the same folder that contains `Assets/`, `Packages/`, and `ProjectSettings/`:

```
your-unity-project/
├── Assets/
├── Packages/
├── ProjectSettings/
└── .editorconfig   ← here
```

VS Code picks it up automatically. No extension or additional configuration needed.

## Recommended Extensions

Install these two VS Code extensions for the full experience:

- **[C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)** — enables Roslyn diagnostics (`IDE0xxx` rules) and naming convention enforcement. Without it, only basic formatting rules apply.
- **[Unity for VS Code](https://marketplace.visualstudio.com/items?itemName=VisualStudioToolsForUnity.vstuc)** — Unity project integration and debugger support.

---

## Table of Contents

1. [What is `.editorconfig`?](#1-what-is-editorconfig)
2. [Root Declaration](#2-root-declaration)
3. [Global Rules (all files)](#3-global-rules-all-files)
4. [File-Type Overrides](#4-file-type-overrides)
5. [C# Indentation](#5-c-indentation)
6. [New Line Rules](#6-new-line-rules)
7. [Spacing Rules](#7-spacing-rules)
8. [Wrapping Rules](#8-wrapping-rules)
9. [Expression-Bodied Members](#9-expression-bodied-members)
10. [Modifier Order & Access](#10-modifier-order--access)
11. [Modern C# & Pattern Matching](#11-modern-c--pattern-matching)
12. [var Usage](#12-var-usage)
13. [Using Directives](#13-using-directives)
14. [Expression-Level Preferences](#14-expression-level-preferences)
15. [Parentheses Preferences](#15-parentheses-preferences)
16. [Predefined Type Keywords](#16-predefined-type-keywords)
17. [Type Qualification (this.)](#17-type-qualification-this)
18. [Naming Conventions](#18-naming-conventions)
19. [IDE Diagnostic Severities](#19-ide-diagnostic-severities)
20. [Known Inconsistency](#20-known-inconsistencies)

---

## 1. What is `.editorconfig`?

`.editorconfig` is a configuration file that enforces consistent coding style across editors and IDEs — regardless of who is working on the project or what machine they use.

This file's goals:

- Uniform indentation and line endings across the entire team
- Fewer noise diffs in code reviews caused by formatting differences
- Shared IDE warnings and suggestions
- Automated enforcement of C# style preferences
- Standardized naming conventions

---

## 2. Root Declaration

```ini
root = true
```

Tells editors that this is the top-level `.editorconfig`. No parent directories will be searched for additional config files.

---

## 3. Global Rules (all files)

```ini
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 4
trim_trailing_whitespace = true
```

### `charset = utf-8`
All files are saved as UTF-8. Special characters, symbols, and emoji work reliably across platforms.

### `end_of_line = lf`
Line endings use Unix-style `LF` (`\n`) rather than Windows `CRLF` (`\r\n`). Keeps Git diffs clean in cross-platform teams.

```
# Bad (Windows CRLF):  "var x = 5;\r\n"
# Good (LF):           "var x = 5;\n"
```

### `insert_final_newline = true`
Every file ends with a blank line. Prevents the "No newline at end of file" warning in Git diffs.

### `indent_style = space` / `indent_size = 4`
Indentation uses spaces, not tabs. Default width is 4 spaces.

```csharp
public class PlayerService
{
    public void Start()
    {
        Console.WriteLine("Started.");
    }
}
```

### `trim_trailing_whitespace = true`
Invisible trailing spaces at the end of lines are automatically removed on save.

---

## 4. File-Type Overrides

### YAML — 2 spaces

```ini
[*.{yml,yaml}]
indent_size = 2
```

```yaml
player:
  name: Ada
  stats:
    hp: 100
    mp: 30
```

### JSON, asmdef, asmref — 2 spaces

```ini
[*.{json,asmdef,asmref}]
indent_size = 2
```

```json
{
  "name": "Game.Core",
  "references": [
    "Game.Common"
  ]
}
```

### Shader files — 4 spaces

```ini
[*.{shader,hlsl,glsl,cginc}]
indent_size = 4
```

### Unity `.cs.meta` files — 2 spaces

```ini
[*.cs.meta]
indent_size = 2
```

---

## 5. C# Indentation

### `csharp_indent_block_contents = true`
Code inside blocks is indented.

```csharp
if (isReady)
{
    Run();  // indented
}
```

### `csharp_indent_braces = false`
Curly braces themselves do not receive extra indentation.

```csharp
// Correct
public class OrderService
{
    public void Process()
    {
        Save();
    }
}

// Wrong
public class OrderService
    {
        public void Process()
            {
            }
    }
```

### `csharp_indent_case_contents = true`
Code inside `case` blocks is indented.

```csharp
switch (command)
{
    case "start":
        StartGame();  // indented
        break;
}
```

### `csharp_indent_case_contents_when_block = true`
When a `case` contains a block, its contents are also indented.

```csharp
switch (command)
{
    case "start":
    {
        Log("Starting");
        StartGame();
        break;
    }
}
```

### `csharp_indent_switch_labels = true`
`case` and `default` labels are indented inside the `switch`.

```csharp
switch (state)
{
    case GameState.Menu:
        OpenMenu();
        break;

    default:
        break;
}
```

### `csharp_indent_labels = one_less_then_current`
Labels (e.g. `goto` targets) are indented one level less than the surrounding block.

```csharp
public void Run()
{
start:
    Console.WriteLine("Running");
}
```

> **Note:** The value `one_less_then_current` contains a likely typo — it should be `one_less_than_current`. Some tools may treat this as an unrecognized value.

### `csharp_prefer_braces = when_multiline:warning` ⚠️
Curly braces are required when a block spans multiple lines. Single-line guard clauses are allowed without braces. Violations produce a **warning**.

```csharp
// Allowed — single-line
if (x) return;

// Correct — multi-line requires braces
if (isReady)
{
    Log("Starting");
    Start();
}

// Warning ⚠️ — multi-line without braces
if (isReady)
    Start();
```

---

## 6. New Line Rules

### `csharp_new_line_before_open_brace = all`
Opening braces always go on a new line (Allman style).

```csharp
// Correct (Allman)
public class InventoryService
{
    public void Reset()
    {
        Clear();
    }
}

// Wrong (K&R)
public class InventoryService {
    public void Reset() {
        Clear();
    }
}
```

### `csharp_new_line_before_else = true`

```csharp
if (isValid)
{
    Save();
}
else
{
    Reject();
}
```

### `csharp_new_line_before_catch = true` / `csharp_new_line_before_finally = true`

```csharp
try
{
    Open();
}
catch (Exception ex)
{
    Log(ex);
}
finally
{
    Close();
}
```

### `csharp_new_line_before_members_in_object_initializers = false`
Object initializer members do not need to be on separate lines.

```csharp
// Both are allowed
var player = new Player { Name = "Ada", Level = 5 };

var player = new Player
{
    Name = "Ada",
    Level = 5,
    Health = 100
};
```

### `csharp_new_line_before_members_in_anonymous_types = false`
Same rule applies to anonymous types.

```csharp
var info = new { Name = "Ada", Score = 100 };
```

### `csharp_new_line_between_query_expression_clauses = true`
Each LINQ clause goes on its own line.

```csharp
var result =
    from player in players
    where player.IsActive
    select player.Name;
```

---

## 7. Spacing Rules

### `csharp_space_after_cast = false`
No space between a cast and the value.

```csharp
var x = (int)obj;    // Correct
var x = (int) obj;   // Wrong
```

### `csharp_space_after_keywords_in_control_flow_statements = true`
Space after `if`, `for`, `while`, `foreach`, etc.

```csharp
if (x) { }       // Correct
if(x) { }        // Wrong

for (int i = 0; i < 10; i++) { }   // Correct
for(int i = 0; i < 10; i++) { }    // Wrong
```

### `csharp_space_around_binary_operators = before_and_after`
Spaces on both sides of binary operators (`+`, `-`, `==`, `&&`, etc.).

```csharp
var z = x + y;      // Correct
var z = x+y;        // Wrong

if (a == b) { }     // Correct
if (a==b) { }       // Wrong
```

### `csharp_space_after_comma = true`
Space after commas.

```csharp
Foo(x, y, z);    // Correct
Foo(x,y,z);      // Wrong
```

### `csharp_space_after_colon_in_inheritance_clause = true` / `csharp_space_before_colon_in_inheritance_clause = true`
Space on both sides of `:` in inheritance declarations.

```csharp
class Dog : Animal { }    // Correct
class Dog:Animal { }      // Wrong
```

### `csharp_space_between_method_call_name_and_opening_parenthesis = false`
No space between the method name and `(`.

```csharp
Foo();        // Correct
Foo ();       // Wrong
Bar(x, y);    // Correct
Bar (x, y);   // Wrong
```

### `csharp_space_between_method_declaration_name_and_open_parenthesis = false`
Same rule applies in method declarations.

```csharp
public void Start() { }     // Correct
public void Start () { }    // Wrong
```

### `csharp_space_between_parentheses = false`
No space inside parentheses.

```csharp
if (x > 0) { }      // Correct
if ( x > 0 ) { }    // Wrong
```

### `csharp_space_before_open_square_brackets = false` / related bracket rules
No extra spaces around square brackets.

```csharp
var item = list[0];       // Correct
var item = list [0];      // Wrong
int[] arr = new int[5];   // Correct
```

---

## 8. Wrapping Rules

### `csharp_preserve_single_line_statements = true`
The formatter does not break single-line statements.

```csharp
if (x) return;  // formatter leaves this as-is
```

> **Conflict with `csharp_prefer_braces`** — see [C# Indentation](#5-c-indentation) for details.

### `csharp_preserve_single_line_blocks = true`
Simple single-line blocks are not expanded by the formatter.

```csharp
public int X { get; set; }  // kept on one line
```

---

## 9. Expression-Bodied Members

### Accessors → `false:suggestion`
Block bodies are preferred for `get`/`set` accessors. Using `=>` is a suggestion, not a warning.

```csharp
// Preferred
public string Name
{
    get { return _name; }
    set { _name = value; }
}

// Suggestion (not wrong, just not preferred)
public string Name
{
    get => _name;
    set => _name = value;
}
```

### Lambdas → `true:warning` ⚠️

```csharp
// Correct
Func<int, int> square = x => x * x;

// Warning ⚠️
Func<int, int> square = x => { return x * x; };
```

### Methods → `false:suggestion`
Block bodies are preferred for methods (suggestion only, not a warning).

```csharp
// Preferred
public int Add(int a, int b)
{
    return a + b;
}

// Suggestion
public int Add(int a, int b) => a + b;
```

### Constructors → `false:suggestion`
Block bodies are preferred for constructors.

```csharp
// Preferred
public Player(int health)
{
    _health = health;
}
```

### Properties, Indexers, Operators, Local Functions → `false:suggestion`
Block bodies preferred.

---

## 10. Modifier Order & Access

### `csharp_preferred_modifier_order`
Modifiers must appear in this exact order:

```
public → private → protected → internal → static → extern → new →
virtual → abstract → sealed → override → readonly → unsafe → volatile → async
```

```csharp
public static readonly int MaxScore = 100;    // Correct
readonly static public int MaxScore = 100;    // Wrong order
```

### `dotnet_style_require_accessibility_modifiers = for_non_interface_members:suggestion`
Access modifiers must be written explicitly on all non-interface members.

```csharp
private int _count;      // Correct
public void Foo() { }   // Correct

int _count;              // Missing access modifier — suggestion
void Foo() { }           // Missing access modifier — suggestion
```

### `csharp_style_prefer_readonly_struct = true:suggestion`
Structs whose contents never change should be marked `readonly`.

```csharp
readonly struct Tile { public int X; public int Y; }
```

### `csharp_style_prefer_readonly_struct_member = true:suggestion`
Individual struct members that are never mutated should be marked `readonly`.

---

## 11. Modern C# & Pattern Matching

### `is` with pattern instead of cast → `warning` ⚠️

```csharp
// Correct
if (obj is string s)
    Console.WriteLine(s);

// Warning ⚠️
if (obj is string)
    Console.WriteLine((string)obj);
```

### `is` instead of `as` + null check → `suggestion`

```csharp
// Preferred
if (obj is string s)
    Use(s);

// Suggestion
var s = obj as string;
if (s != null) Use(s);
```

### `not` pattern → `suggestion`

```csharp
if (obj is not null)     // Preferred
if (!(obj is null))      // Suggestion
```

### Switch expression → `suggestion`

```csharp
// Preferred
var label = state switch
{
    GameState.Playing => "Playing",
    GameState.Paused  => "Paused",
    _                 => "Unknown"
};

// Suggestion
string label;
switch (state)
{
    case GameState.Playing: label = "Playing"; break;
    // ...
}
```

### `throw` expression → `suggestion`

```csharp
// Preferred
_name = name ?? throw new ArgumentNullException(nameof(name));

// Suggestion
if (name == null) throw new ArgumentNullException(nameof(name));
_name = name;
```

### Index and Range operators → `suggestion`

```csharp
var last  = arr[^1];     // ^ index operator
var slice = arr[1..4];   // .. range operator
```

### `default` simplification → `suggestion`

```csharp
int x = default;         // Preferred
int x = default(int);    // Suggestion
```

### Deconstructed variable declaration → `suggestion`

```csharp
var (x, y) = GetPoint();    // Preferred
int x = point.Item1;        // Suggestion
```

### Implicit object creation when type is apparent → `suggestion`

```csharp
List<string> names = new();    // Preferred
List<string> names = new List<string>();    // Suggestion
```

### Inlined variable declaration → `suggestion`

```csharp
if (int.TryParse(input, out int result)) { }    // Preferred
int result;
if (int.TryParse(input, out result)) { }        // Suggestion
```

### `csharp_style_prefer_extended_property_pattern = true:suggestion`

```csharp
if (obj is { Address.City: "London" })    // Preferred
if (obj is { Address: { City: "London" } })    // Suggestion
```

### `csharp_style_prefer_pattern_matching = true:suggestion`
Use pattern matching constructs where possible.

### `csharp_style_conditional_delegate_call = true:suggestion`
Use `?.Invoke()` instead of null-checking before delegate calls.

```csharp
OnDeath?.Invoke();      // Preferred
if (OnDeath != null) OnDeath();    // Suggestion
```

---

## 12. var Usage

All three rules are set to `warning` — `var` is required in every situation. Not using it produces a **warning**.

### `csharp_style_var_when_type_is_apparent = true:warning` ⚠️

```csharp
var list = new List<string>();           // Correct
List<string> list = new List<string>();  // Warning ⚠️
```

### `csharp_style_var_for_built_in_types = true:warning` ⚠️

```csharp
var x    = 42;     // Correct
var name = "Ada";  // Correct

int x       = 42;     // Warning ⚠️
string name = "Ada";  // Warning ⚠️
```

### `csharp_style_var_elsewhere = true:warning` ⚠️

```csharp
var result = GetPlayer();      // Correct
Player result = GetPlayer();   // Warning ⚠️
```

---

## 13. Using Directives

### `csharp_using_directive_placement = outside_namespace:error` ❌
`using` directives must be outside the `namespace` block. Violations are a **compile error**.

```csharp
// Correct
using System;
using UnityEngine;

namespace MyGame
{
    class Player { }
}

// Error ❌
namespace MyGame
{
    using System;
}
```

### `dotnet_sort_system_directives_first = true:warning` ⚠️
`System.*` namespaces must appear first. Violations produce a **warning**.

```csharp
// Correct
using System;
using System.Collections.Generic;
using UnityEngine;
using MyGame.Core;

// Warning ⚠️ — System.* not first
using UnityEngine;
using System;
```

### `dotnet_separate_import_directive_groups = false:warning` ⚠️
No blank lines between groups of `using` directives. Blank lines between them produce a **warning**.

```csharp
// Correct — no blank lines between usings
using System;
using System.Collections.Generic;
using UnityEngine;

// Warning ⚠️ — blank line separating groups
using System;
using System.Collections.Generic;

using UnityEngine;
```

### `csharp_prefer_simple_using_statement = true:suggestion`
Use the declaration-only `using` when the scope covers the rest of the method.

```csharp
// Preferred
using var reader = new StreamReader(path);

// Suggestion
using (var reader = new StreamReader(path))
{
    // ...
}
```

### `csharp_style_namespace_declarations = block_scoped:suggestion`
Namespaces use the traditional block style, not the file-scoped style.

```csharp
// Preferred
namespace MyGame
{
    class Player { }
}

// Suggestion (file-scoped)
namespace MyGame;
class Player { }
```

---

## 14. Expression-Level Preferences

### Null propagation `?.` → `suggestion`

```csharp
var name = player?.Name;                          // Preferred
var name = player != null ? player.Name : null;  // Suggestion
```

### Null coalescing `??` → `suggestion`

```csharp
var x = value ?? defaultValue;                   // Preferred
var x = value != null ? value : defaultValue;    // Suggestion
```

### `is null` check → `warning` ⚠️

```csharp
if (obj is null) { }                            // Correct
if (object.ReferenceEquals(obj, null)) { }      // Warning ⚠️
```

### Object initializer → `suggestion`

```csharp
var p = new Point { X = 1, Y = 2 };   // Preferred
var p = new Point();                   // Suggestion
p.X = 1; p.Y = 2;
```

### Collection initializer → `suggestion`

```csharp
var list = new List<int> { 1, 2, 3 };   // Preferred
var list = new List<int>();              // Suggestion
list.Add(1); list.Add(2); list.Add(3);
```

### Explicit tuple names → `warning` ⚠️

```csharp
var t = (Name: "Ada", Score: 100);
Console.WriteLine(t.Name);      // Correct
Console.WriteLine(t.Item1);     // Warning ⚠️
```

### Compound assignment → `suggestion`

```csharp
score += 10;    // Preferred
score = score + 10;    // Suggestion
```

### Conditional expression over assignment → `suggestion`

```csharp
var label = isAdmin ? "Admin" : "User";    // Preferred

string label;
if (isAdmin) { label = "Admin"; }
else         { label = "User"; }           // Suggestion
```

### Conditional expression over return → `suggestion`

```csharp
return isReady ? "Ready" : "Waiting";    // Preferred

if (isReady) return "Ready";
return "Waiting";                         // Suggestion
```

### Auto-properties → `suggestion`

```csharp
public int Score { get; set; }    // Preferred

private int _score;
public int Score { get => _score; set => _score = value; }    // Suggestion
```

---

## 15. Parentheses Preferences

### Arithmetic operators — remove if unnecessary → `warning` ⚠️

```csharp
var total = a + b * c;         // Correct
var total = (a) + (b * c);    // Warning ⚠️ — unnecessary parentheses
```

### Other binary operators — always for clarity → `warning` ⚠️

```csharp
var result = (isReady && hasAccess) || isAdmin;    // Correct — clarity parens
```

### Relational and other operators — remove if unnecessary → `warning` ⚠️

```csharp
if (x > 0) { }      // Correct
if ((x > 0)) { }    // Warning ⚠️
```

---

## 16. Predefined Type Keywords

```ini
dotnet_style_predefined_type_for_locals_parameters_members = true:warning
dotnet_style_predefined_type_for_member_access = true:warning
```

Use C# keyword aliases instead of .NET type names.

```csharp
// Correct
int count = 5;
string name = "Ada";
bool isReady = true;

// Warning ⚠️
Int32 count = 5;
String name = "Ada";
Boolean isReady = true;
```

---

## 17. Type Qualification (`this.`)

All four qualification rules are set to `false:suggestion` — `this.` is not needed unless resolving a naming conflict.

```csharp
// Preferred
public class Player
{
    private int _score;

    public void AddScore(int value)
    {
        _score += value;
    }
}

// Suggestion (unnecessary this.)
public class Player
{
    private int _score;

    public void AddScore(int value)
    {
        this._score += value;
    }
}
```

> `this.` is still valid and necessary when a local variable shadows a field name.

---

## 18. Naming Conventions

### Summary table

| Symbol | Convention | Example |
|--------|-----------|---------|
| Interface | `I` + PascalCase | `IVehicle`, `IPlayerController` |
| Class / Struct / Enum / Delegate | PascalCase | `PlayerController`, `GameState` |
| Method / Property / Event | PascalCase | `MovePlayer()`, `Health`, `OnDeath` |
| `const` | PascalCase | `MaxHealth`, `DefaultSpeed` |
| `static readonly` field | PascalCase | `DefaultConfig` |
| `public` / `internal` / `protected` field | camelCase | `moveSpeed`, `score` |
| `private` / `private protected` field | `_` + camelCase | `_health`, `_playerName` |
| Parameter / local variable | camelCase | `playerName`, `finalDamage` |
| Generic type parameter | `T` + PascalCase | `TEntity`, `TInput` |

### Examples

```csharp
// Interface
public interface IWeapon { }

// Class / Struct / Enum
public class PlayerController { }
public enum GameState { Playing, Paused }
public readonly struct Tile { }

// Method / Property / Event
public void Attack() { }
public int Health { get; set; }
public event Action OnDeath;

// Constants
public const int MaxAmmo = 30;
private static readonly string DefaultTag = "Player";

// Fields
public float moveSpeed;         // public — camelCase
private int _health;            // private — _camelCase

// Local variables and parameters
void SetName(string playerName)
{
    var trimmedName = playerName.Trim();
}

// Generic type parameter
public class Repository<TEntity> { }
```



---

## 19. IDE Diagnostic Severities

| Rule | Description | Severity |
|------|-------------|----------|
| `IDE0001` | Simplify type name | `warning` |
| `IDE0002` | Simplify member access | `warning` |
| `IDE0004` | Remove unnecessary cast | `warning` |
| `IDE0005` | Remove unnecessary `using` | `warning` |
| `IDE0010` | Populate `switch` statement | `warning` |
| `IDE0020` | Use pattern matching | `warning` |
| `IDE0029` | Null coalescing can be simplified | `warning` |
| `IDE0030` | Null coalescing can be simplified (nullable) | `warning` |
| `IDE0031` | Use null propagation | `warning` |
| `IDE0034` | Simplify `default` expression | `warning` |
| `IDE0036` | Order modifiers | `warning` |
| `IDE0040` | Add accessibility modifiers | `warning` |
| `IDE0044` | Add `readonly` modifier | `warning` |
| `IDE0051` | Unused private member | `warning` |
| `IDE0052` | Unread private member | `warning` |
| `IDE0055` | Fix formatting | `warning` |
| `IDE0058` | Expression value unused | `warning` |
| `IDE0059` | Unnecessary value assignment | `warning` |
| `IDE0060` | Unused parameter | `warning` |
| `IDE0071` | Simplify interpolation | `warning` |
| `IDE1006` | Naming rule violation | `warning` |
| `VSTHRD200` | Async method suffix | `none` (disabled) |

### Key rules explained

**`IDE0044` — Add readonly modifier**

```csharp
private readonly int _id;   // Correct
private int _id;            // IDE0044 — only set in constructor, never changed
```

**`IDE0051` — Unused private member**

```csharp
private int _unusedField;   // IDE0051 — never read or written
```

**`IDE0052` — Unread private member**

```csharp
private int _score;
void Foo() { _score = 10; }   // IDE0052 — written but never read
```

**`IDE0059` — Unnecessary value assignment**

```csharp
var x = Compute();     // value never used
x = OtherCompute();   // overwritten before being read — IDE0059
```

**`IDE0060` — Unused parameter**

```csharp
void Foo(int unused)   // IDE0060 — parameter is never referenced
{
    Bar();
}
```

**`VSTHRD200` — Async suffix (disabled)**

Async methods are not required to end in `Async`. This is intentional for Unity projects where the convention adds noise.

```csharp
// Both are valid in this project
async Task LoadLevel() { }
async Task LoadLevelAsync() { }
```

---

## 20. Known Inconsistencies

### Typo in `csharp_indent_labels`

```ini
csharp_indent_labels = one_less_then_current
```

The correct value is `one_less_than_current`. Some tools may not recognize `then` and fall back to a default. Worth fixing.

---

## Recommended Tooling

This file is developed for **Unity C# projects using Visual Studio Code**. VS Code has built-in `.editorconfig` support — no extension needed for basic formatting rules (indentation, line endings, charset). For the best experience, install the following VS Code extensions:

- **[C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)** — Roslyn-powered IntelliSense, diagnostics, and `.editorconfig` enforcement
- **[Unity](https://marketplace.visualstudio.com/items?itemName=VisualStudioToolsForUnity.vstuc)** — Unity project integration and debugger support

The IDE diagnostic rules (`IDE0xxx`) and naming conventions require the **C# Dev Kit** extension (or the standalone **C#** extension with Roslyn analyzers) to fire inside VS Code. Without it, only the basic formatting rules (indentation, line endings, spacing) will be enforced.

For CI pipelines, `dotnet format` applies all formatting and style rules from the command line without an IDE.

---

## References

- [editorconfig.org](https://editorconfig.org) — official EditorConfig specification and supported editor list
- [.NET code style rule options](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/code-style-rule-options) — Microsoft documentation for all `dotnet_*` and `csharp_*` rules
- [Naming rules](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/naming-rules) — `dotnet_naming_rule` reference
