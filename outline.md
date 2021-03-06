# PLISS 2019

# Responsive Compilers

Does this look familiar?

![Image result for dragon book](https://images-na.ssl-images-amazon.com/images/I/51FWXX9KWVL._AC_UL320_SR248,320_.jpg)

# Pipelines and passes

The traditional model for a compiler is something like:

- Lex the source, producing tokens
- Parse the tokens, producing an AST
- Do semantic analysis, resolving names 
- Type-check the source
- etc etc
# This is how rustc used to look

(Full confession: And sort of how it still looks)

# Yesterday’s compiler
    > mycc foo.c
    blah.c:23:22: you messed up here
        printf("Hello, world!\n');
                               ^
    > 
# Today’s compiler
![Image result for vscode](https://raw.githubusercontent.com/be5invis/vscode-custom-css/master/screenshot.png)

# Why should you care about IDEs?
- You have to write code in this language you’re making
- It affects your language design — dependencies matter
- Much easier if you do it correctly from the start =)


# Outline
- Besides IDEs, there are other problems with strict phase separation
    - Languages often get a bit messier in practice
    - Const evaluation: 
        - …
    - Trait specialization:
        - …
- Start from the *goal* and work your way backwards (“demand driven”)
    - able to figure out the minimum amount of work needed to do anything
    - harder than it sounds
- Not a solved problem =)
    - I’m going to present you the current state of my exploration of this space
    - This is also gleaned from (limited) conversations with other folks
- The start of the story:
    - rustc — too dang slow
    - RFC for incremental compilation
        - kind of a “make-like” system
        - aimed to be something we could incrementally add
- Key requirement:
  - be able to *prioritize* requests
  - e.g., user typed `.`, what is the minimal work we can do to get that result?
- “Salsa” — a system in rustc
    - High-level idea:
        - Separate out your **inputs** and **things derived from those inputs** 
        - For derived things, strictly separate out what they can get as input
    - Goals:
        - Parallel execution
        - Incremental re-execution
        - On-demand, minimal execution
- Alternatives and related work:
    - “Hand-coded”
        - My experience: quite challenging, not recommended
    - Salsa-like:
        - Adapton
        - Glimmer (part of a web programming framework called Ember)
    - Expressing using a more formal technique:
        - Attribute grammars
        - Datalog or other such things
- Entity-Component-System
    - Move away from structs with properties
    - Separate out *identity* from *type* 
    - Originated from games, but it’s kind of a “stepping stone” to how Salsa works.
    - **Entity**: the identity of something. Usually just an arbitrary integer.
        - In games, might be a monster or player or something.
    - **Component:** some property of an entity
        - In games, might be “position” or “health” or a character etc
- Salsa's model is a generalization of ECS
    - Base concept is the **query** -- something you want to compute
    - A query looks like `QueryName(K0..Kn)`
    - A "component" in ECS would be `Component(E)` where the key `E` is the entity
- Queries are divided into two main groups:
    - *input queries* -- set explicitly by the user
    - *derived queries* -- computed via a pure function
- Basic model:
    - loop:
        - user sets inputs
        - user "demands" derived queries as needed
            - results are memoized and recomputed only if necessary
- Making this concrete:

```rust
// create the salsa database:
let mut db = Database::new();

// Set the value for `my_input(key)` to 22
salsa.set_my_input(key, 22);

// Compute some derived value `my_derived(key)`
salsa.my_derived(key);

// Modify `my_input(key)` to 44
salsa.set_my_input(key, 44);

// Re-compute some derived value `my_derived(key)`
salsa.my_derived(key);
```

- Entities in a compiler
    - Entities can be a *path*
    - Details aren't so important, but it's important that it is a tree
    - Give an example of some Rust source
- Why a tree?
    - It remains stable as the user makes edits
- We're going to "layer" information on these entities
    - To start we have only the source for files
    - We compute other things -- like AST, etc
- Basic model is just a starting point
    - in practice, lots to tweak -- how much do you memoize? etc
    - can also demand things in parallel
- 

- Salsa’s model is similar to ECS, but different
- 
    - Building block is a **query** — something you want to compute.
        - An example might be to parse and produce the AST of a file
    - Queries have **keys** — there can be any number, and they can have any type, but
        - keys are **values** — like integers — no internal mutation, etc


Support incremental re-execution

- The "cutoff" or "firewall" pattern and how to make use of it
- Mapping
- Error handling — the show must go on!
    - techniques for this
- 

