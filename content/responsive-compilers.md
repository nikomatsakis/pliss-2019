class: center
name: title
count: false

# Responsive compilers

.me[.grey[*by* **Nicholas Matsakis**]]
.citation[`https://github.com/nikomatsakis/pliss-2019`]

---

# Does this look familiar?

.center[
![Image result for dragon book](content/images/dragon-book.jpg)
]

---

# Pipelines and passes

Traditional compiler model is a series of **passes**:

- `lex(source) -> tokens`
- `parse(tokens) -> ast`
- `semantic_analysis(ast)`, `type_check(ast)`
- loop:
  - apply optimizations
- etc

---

# Pipelines and passes

That is precisely how rustc used to look.

--

Oooookay...maybe it still sorta' looks like that.

---

# Yesterday's compiler

```bash
> mycc foo.c
blah.c:23:22: you messed up here
    printf("Hello, world!\n');
                           ^
>
```

---

# Today's compiler

.center[
![Image result for vscode](content/images/vscode.png)
]

---

# My goal today

What if...

.center[
![Dragon book](content/images/dragon-book.jpg)
]

...were written today?

---

# My goal today

.center[
![Dragon grammar book](content/images/dragon-grammar-book.jpg)
]

???

OK, let's start with the cover. I love dragons, but I'm not so into
combat. So I found this book that had a more friendly looking dragon.
And the title even seemed almost suitable?

--

.LR[
LR
]

???

Just gotta add one thing...

OK, seriously, what I want to talk about is some ideas for how to
structure a compiler so that it can be readily integrated into an IDE.

---

# How to IDE

.center[
![Language Server Protocol](content/images/lsp-languages-editors.png)
]

???

In Ye Olden Days, writing an IDE involved a lot of grody Java coding
against some obscure set of APIs. Nowadays, you can use the Language
Server Protocol, introduced as part of VSCode, but supported in a
number of editors.

---

# The "responsive compiler"

- **Compiler as an actor**
- Editor sends diffs and requests completions, diagnostics
- Compiler responds

--

**Key point:** need to be able to respond as quickly as possible!

---

# Demand driven

- Start from **goal**
- Figure out what is needed for that

---

# Niko, why are you doing this to me?

.center[.p60[
![Panda facepalm](content/images/panda-facepalm.gif)
]]

???

I can hear you now... don't I already have enough to do?  I have to
not only design a really cool new idea for a language, but now I have
to worry about IDEs too? My advisor is going to kill me.

---

# Why should you care about IDEs?

- You have to write code in this language you’re making

--
- It affects your language design -- dependencies matter

---

# Dependencies matter

Rust allows arbitrary nesting of declarations inside functions:

```rust
fn foo() {
  // Equivalent to a struct declared at the root of the
  // file, but only visible inside this function.
  struct Bar { }

  let x = Bar { ... };
}
```

???

We have permitted this sort of nesting since basically forever.  It
seemed "quirky but harmless".

---

# Dependencies matter

You can nest an `impl` block too, which adds methods:

```rust
fn foo() {
  struct Bar { }

  impl Bar {
    pub fn method() { .. }
  }
}
```

---

# Dependencies matter

You can even add methods to types that *are* visible from the outside:

```rust
struct Bar { }

fn foo() {
  impl Bar {
    pub fn method() { .. }
  }
}
```

---

# Dependencies matter

```rust
struct Bar { }

fn some_method() {
  let bar = Bar::new();


  bar. // <-- what methods should we offer as auto-completion here?
}
```

A side-effect of this is that auto-completion requires looking inside
a lot of function bodies.

---

# Why should you care about IDEs?

- You have to write code in this language you’re making
- It affects your language design -- dependencies matter

--
- Strict phase separation is impossible anyway

---

# Strict phase separation is impossible anyway

.center[
.p80[
![That's just, like, your opinion man](content/images/opinion.gif)
]
]

???

OK, impossible is a strong claim. But what I mean is this:

It's convenient to structure your compiler as "do X to the program"
then do Y.  e.g., resolve all the names, then do type-checking, then
lower to a new IR, then optimize, etc.

But often, you need to be more flexible on the order you do things
anyway, and drive the control-flow based on the structure of the
**program itself**.

---

# Strict phase separation is impossible anyway

Rust, for example, lets you do this:

```rust
const LEN: u8 = 1 + 1 + 1;
const DATA: [u8; LEN] = [1, 1, 1];
```

???

Rust permits you to define named constants and then use those
constants in types. For this to work, we need to be able to evaluate
those constants to integers. They can be (near) arbitrary expressions,
so that us to be able to type-check those expressions.

This means we need to be able to type-check and evaluate the body of `LEN`
before we

---

# Strict phase separation is impossible anyway

But then, what if I were to do this?

```rust
const LEN: u8 = DATA[0] + DATA[1] + DATA[2];
const DATA: [u8; LEN] = [1, 1, 1];
```

Uh oh.

???

Now, in order to construct the type of `DATA`, we have to evaluate
`LEN` to an integer, but that requires us to access `DATA` (which
wants to know its type). Kind of a mess. (And Rust indeed presently
errors in scenarios like this, though you can imagine making it work
out by being a bit more sophisticated.)

---

# Strict phase separation is impossible anyway

Other examples:

- Inferred types across function boundaries
- From Rust, specialization, which requires solving some traits but not others
- Java and its lazy class file loading
- Phase separation and Scheme macros

???

These sorts of dependencies are very important, actually.

---

# Not a solved problem

- "Hand coded"
- "Salsa" &larr; what I'm largely presenting today
- More formal techniques:
  - Attribute grammars
  - Datalog or structured queries

???

Here's the thing. There are a lot of people taking stabs at this
overall problem. I'm going to talk to you about one approach we've
been using that seems to work fairly well, but it's not without its
flaws. And we've not gotten it fully working yet.

---

# Salsa

- High-level idea:
  - Inputs
  - Derived queries
- Most closely related work:
  - "Adapton"
  - "Glimmer" (from the Ember web framework)
  - "Build systems a la carte"

???

The core idea of salsa is that you are going to setup your program in
two ways:

- First, you have your base inputs.
- Then you have your derived data. These are defined by pure functions
  that can access base inputs or other derived data.

So your base input might be the raw program text, for example. Derived
information might be things like "the set of errors in this file" or
"completions at this point".

---

# Salsa core idea

```rust
let mut db = Database::new();
loop {
  db.set_input_1(...);
  db.set_input_2(...);

  db.derived_value_1();
  db.derived_value_2();
}
```

???

- Base idea:
  - set your inputs
  - demand whatever derived info you want
- System does minimal work to return up-to-date value
- Composing your program from pure functions is useful
- This is still a WIP; based on experience and conversations, but I
  don't feel we've got all the pieces in place yet

---

# Entity Component System

- **Entity:** Unit of entity
- **Component:** Data about an entity

```rust
fn move_left(entity: Entity, amount: usize) {
  let pos = DB.position(entity);
  pos.x -= amount;
  DB.set_position(entity, pos);
}
```

???

- Useful starting point to explain salsa
- Developed in games
- Alternative to strict OO hierarchies
- Attach components to entity -- things like health, position, etc
- In games, able to do things like "give me all entities with a position"
- Salsa doesn't readily support that, but it *is* often convenient to
  be able to attach "things with a path" or "things with a type",
  which can be a wide range of things

---

# Entities in a compiler

- Often called "symbols"
- Things like:
  - input files
  - struct declarations
  - fields
  - function declarations
  - parameters or local variables
- Something "addressable" by other parts of the system

---

# Components in a compiler

- Things like:
  - the **type** of an entity
  - the **signature** of a function

---

# Salsa queries

```
Q(K0..Kn) -> V
```

- `Q` is the **query name** (e.g., `AST`)
- The `K0..Kn` are the **query keys**
  - Atomic **values** of any type
- The `V` is the **value** associated with the query

???

- The basis in salsa is the **query**.
- The query name `Q` is like the "component".
- Whereas in an ECS, components are attached to a single entity, queries in salsa
  can have multiple **keys**, though often they only have one in practice.

---

# Example queries

- `input_text(FileName)`
- `ast(FileName) -> Ast`
- `signature(Entity) -> Signature`
- `completions(FileName, Line, Column) -> Vec<String>`

???

- Queries are used to capture all kinds of things:
  - base inputs like the raw text
  - intermediate values like ast, signature
  - final results like completions

---

# Query group

```rust
#[salsa::query_group]
trait CompilerDatabase {

  #[salsa::input]
  fn input_text(&self, filename: FileName) -> String;

  fn ast(&self, filename: FileName) -> Ast;

  fn signature(&self, entity: Entity) -> FnSignature;

}
```

---

# Input queries

```rust
//  #[salsa::input]
//  fn input_text(&self, filename: FileName) -> String;

let text = db.input_text(filename);
db.set_input_text(filename, text);
```

- Input queries are essentially a field:
  - The `#[salsa::input]` annotation generates accessors

---

# Derived queries

```rust
// fn ast(&self, filename: FileName) -> Ast;

fn ast(
  db: &impl CompilerDatabase,
  filename: FileName,
) -> Ast {
  let input_text = db.input_text(filename);
  ... /* implement parser here */ ...
}
```

- Derived queries are defined by a function:
  - The `db` parameter is the database, gives access to other queries
  - Given keys, return results

---

# How Salsa works

```rust
db.ast("foo.rs")
```

- Database contains:
    - global revision counter
    - one map per query (e.g., `ast`)
    - maps query key to:
         - Memoized result
         - Vector of dependencies (`[input_text("foo.rs")]`)
         - Revision when last changed

---

# Recomputation (simplified)

- When `db.ast("foo.rs")` is invoked:
    - If no entry yet, execute query and store result.
    - Otherwise, if any input dependency is out of date:
        - Re-execute `ast` function, recording new result + dependencies
        - Update the "last changed" revision

---

# Recomputation

```rust
db.signature(..)
```

- Current revision: R1
- `db.signature(..)`

--
- `db.ast(..)`

--
- `db.input_text(..)` -- changed in R1

---

# Recomputation

```rust
db.signature(..)
```

- Current revision: R1
- `db.signature(..)` -- changed in R1
- `db.ast(..)` -- changed in R1
- `db.input_text(..)` -- changed in R1

???

Situation when complete:
  - We've recorded a signature, AST, and we have the input
  - All were last changed in revision 1

---

# Recomputation

```rust
db.set_input_text(..)
```

- **Current revision: R2**
- `db.signature(..)` -- changed in R1
- `db.ast(..)` -- changed in R1
- `db.input_text(..)` -- **changed in R2**

???

- User alters the input text
- Start of a new revision
- Nothing happens to `ast` and `signature` ... yet

---

# Recomputation

```rust
db.signature(..)
```

- Current revision: R2
- `db.signature(..)` -- changed in R1
- `db.ast(..)` -- changed in R1 &larr; **out of date**
- `db.input_text(..)` -- changed in R2

???

- User requests up to date signature
- But look, one of signature's dependencies is out of date
- Therefore, `ast` will get re-executed, and so will `signature`.

---

# But suppose input change is not important

Before:

```rust
// foo.rs

fn foo() {
  do_something();
}
```

After:

```rust
// foo.rs

fn foo() {
  do_something(); // FIXME
}
```

---

# Recomputation (less simplified)

- When `db.ast("foo.rs")` is invoked:
  - If any input dependency is out of date:
    - Re-execute `ast` function, recording new result + dependencies
    - **If the new result is different from old result:**
      - Update the "last changed" revision

---

# Recomputation

```rust
db.signature(..)
```

- Current revision: R2
- `db.signature(..)` -- changed in R1
- `db.ast(..)` -- **changed in R1**
- `db.input_text(..)` -- changed in R2

???

- We still re-execute `ast`
- But we do not re-execute `signature`
  - and anything dependent on `signature`

---

# General idea

```
A --> C --> E --> F --> G
            ^
            |
B --> D ----+---> H
```

- Re-execute the "early steps"
- But cut off as quickly as you can

???

- Salsa is always executing the "whole program"
  you wrote, in some sense, buy re-using pieces
- You could get better efficiency by updating old
  pieces "in place", but that's not supported (yet)

---

# Layering

- Common pattern:
  - produce a base structure
  - other queries "layer" structure on top
- Example:
  - parser produces AST
  - name resolution resolves names to entities
  - type check adds types

???

- we would like to be able to re-compute the AST 
  without disturbing the other layers, so they
  can be re-used
- if you store symbol information and so forth directly in the AST,
  you lose that capability

---

# Represent with maps

- Give each node in the AST a numeric id `AstId`
- Name resolution produces `(AstId -> Entity)`
- Type-check produces `(AstId -> Type)`

???

- indices and maps are an easy option
- rustc has used them for a long time, I think in part
  because the compiler started off as o'caml, and ast was
  an immutable data structure
- raises a question though: what should you use as your indices?

---

# Rust compiler of yore

- one big ast
- nodes in the assigned a pre-index (`NodeId`)
- this ID was used everywhere

```rust
fn foo() { // node 0
  let x = 3; // node 1
}

fn bar() { // node 2
  let y = 4; // node 3
}
```

???

- problem: if you edit foo, the nodes in bar would change 

---

# Trees are your friends

```
Entity = FileName
       | Entity "." Index
```

```rust
fn foo() { // node `"foo.rs".0`
  let x = 3; // node `"foo.rs".0.0`
}

fn bar() { // node `"foo.rs".1`
  let y = 4; // node `"foo.rs".1.0`
}
```

???

- editing foo does not affect the entity names in bar
- but, reordering foo and bar would change a lot of ids

---

# Trees are your friends

```
Entity = FileName
       | Entity "." Name [ Index ]
```


```rust
fn foo() { // node `"foo.rs".foo[0]`
  let x = 3; // node `"foo.rs".foo[0].expr[0]`
}

fn bar() { // node `"foo.rs".bar[0]`
  let y = 4; // node `"foo.rs".bar[0].expr[0]`
}
```

???

- introduce *names*
- but what is this integer? That's called the *disambiguator*
- it is used for anonymous things, like expr, but also for conflicts

---

# Trees are your friends

```
Entity = FileName
       | Entity "." Name [ Index ]
```

```rust
fn foo() {} // node `"foo.rs".foo[0]`
fn foo() {} // node `"foo.rs".foo[1]`
```

???

Illegal in Rust, but we have to accept legal *and* illegal programs.

---

# Interning

```rust
struct Entity {
  value: u32
}

enum EntityData {
  Root(FileName),
  Child(Entity), 
}

#[salsa::query_group]
trait CompilerData {

  #[salsa::intern]
  fn intern_entity(&self, data: EntityData) -> Entity;

}
```

???

- Salsa has built-in support for *interning* these sorts of ids.
- It will convert them into unique integers for you (as shown here)
- And you can then lookup the underlying data
- In languages with a GC, you could also just allocate them and store
  them in some sort of global table.

---

# Tree-based entities also give context

```rust
enum EntityData {
  Root(FileName),
  Child(Entity), 
}
```

???

- Given an entity, you can extract the parent function or file it is
  found within
- Lookup the entity, walk upwards to find the root
- Maybe useful to encode more such information in the entity as well,
  such as the "type" of the entity

---

# Signature

Example from earlier:

- `db.signature(entity)`
- `db.ast(file_name)`
- `db.input_text(file_name)`

How do we get the `file_name` from the `entity`?

---

# Tightening queries with projection

- `db.signature(entity)` -- signature of **a function**
- `db.ast(file_name)` -- ast of the **entire file**

???

- Big gap between information returned by `ast` query
- And information needed by `signature`

---

# Tightening queries with projection

- `db.signature(entity)`
- `db.entity_ast(entity)` -- extract AST of a *single* entity
- `db.ast(file_name)`

???

- Introducing an intermediate query tightens that gap
- Now, even if AST changes, if the AST for a particular *entity* has not changed,
  we can avoid recomputing the signature
- Is this worth it? Depends. 
    - One thing I've learned is that it is often a better trade-off to
      re-execute more often.
      
---

# The "outer spine"

Question: How do we get "all the errors in the project"?

--

- `db.all_errors()`
- `db.filenames()` -- get a list of all filenames
- `db.entities(filename)` -- for each filename, get all entities
    - `db.parse(filename)` -- parse file to AST
- `db.type_check(entity)` -- for each entity, do type-check
    - `db.ast(entity)` -- get the AST for this entity
        - `db.parse(filename)` -- parse file to AST (memoized)


???

- This system is kind of like quantum theory, I've found:
  - Quantum theory vs gravity, one works at small scale, one at large scale
  - Not clear how to go from one to the other
- Similarly, it's fairly easy to understand any individual query
- I've found it's fairly easy to understand how the "first few phases" of compiler work
- And kind of clear how "middle bits" work
- But not so obvious how to connect them

---

# The "outer spine"

What if the user edits a comment?

- `db.all_errors()`
- `db.filenames()` -- get a list of all filenames
- `db.entities(filename)` -- for each filename, get all entities
    - `db.parse(filename)` -- parse file to AST
- `db.type_check(entity)` -- for each entity, do type-check
    - `db.ast(entity)` -- get the AST for this entity
        - `db.parse(filename)` -- parse file to AST (memoized)

--

.parse-bullet[![Arrow](content/images/arrow-left.svg)]

Reparse, but that's it.

---

# The "outer spine"

What if the user edits a fn body?

- `db.all_errors()`
- `db.filenames()` -- get a list of all filenames
- `db.entities(filename)` -- for each filename, get all entities
    - `db.parse(filename)` -- parse file to AST
- `db.type_check(entity)` -- for each entity, do type-check
    - `db.ast(entity)` -- get the AST for this entity
        - `db.parse(filename)` -- parse file to AST (memoized)

--

.parse-bullet[![Arrow](content/images/arrow-left.svg)]

Reparse the file.

--

.ast-bullet[![Arrow](content/images/arrow-left.svg)]

Extract the AST.

--

.type-check-bullet[![Arrow](content/images/arrow-left.svg)]

Type-check the function that changed.

---

# Error handling

How not to handle an error in a compiler:

```java
throw new TypeError();
```

---

# Error handling

Another way not to handle errors:

```
if some kind of error {
  return fake-but-otherwise-legal-value;
}
```

---

# Recovery from day one

- Create a sentinel value that means "bad user code here"
- Invariant:
  - If you see this sentinel value, errors have been reported
  - So you can feel free to suppress downstream errors
- No such thing as a "fallible" compiler operation

???

- Much easier if you try to add this from the start
- Not gonna lie, it can still be a pain
    - diminishing returns at some point
    - example: `Vec<Foo>` 

---

# Example: error type

```rust
enum Type {
  Integer,
  Character,
  Error,
}
```

---

# Diminishing returns

```
struct MethodSignature {
  argument_types: Vec<Type>,
  return_type: Type,
} 
```

???

- Can you spot the hidden assumption?
- We assume we know the arity
- In some sorts of parse errors, it might be unclear how many 
  arguments the user intended
- Probably ok, though

---

# Tracking location information ("spans")

- Various techniques:
  - rustc appends all the input files to one big string
  - stores 32-bit indices into that string
  - compact, but hostile to incremental
- One alternative:
  - in the AST node, just stores its offset from previous sibling + length
  - in other nodes, track the AST node id
  - recompute the starting offset when needed

---

# "Zooming out" or "zooming in"

- Eliding data is good
    - especially when it can be recovered
- On-demand system is a good fit for this

???

- Calculating the starting offset
- Storing whitespace and comments in your AST

---

# Threading

- 

---

# Cancellation

???

