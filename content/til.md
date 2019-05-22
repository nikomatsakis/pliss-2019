class: center
name: title
count: false

.TIL[TIL]

.me[.grey[*by* **Nicholas Matsakis**]]
.citation[`https://github.com/nikomatsakis/pliss-2019`]

---

# TIL: what TIL means 

.center[.p60[![TIL stands for "today I learned"](content/images/screenshot-til.png)]]

imo, it should be "things I learned".

so there.

---

# Professional language designer

--

.center[.p80[![Ooooh](content/images/ooo.gif)]]

---

# Professional language designer

.center[
.avatar[![SPJ](content/images/spj.jpg)]
.avatar[![Joe Armstrong](content/images/joe-armstrong.jpg)]
.avatar[![Grace Hopper](content/images/grace-hopper.jpg)]
.avatar[![Matz](content/images/matz.jpg)]
.avatar[![Guido](content/images/guido.jpg)]
.avatar[![Niklaus Wirth](content/images/wirth.jpg)]
]

---

# Professional language designer

.center[.p80[![We're not worthy](content/images/not-worthy.gif)]]

---

background-image: url(content/images/DaphneDino.jpg)
background-size: contain

---

# TIL: No straight lines

.center[.p80[![Spongebob is lost](content/images/lost-spongebob.gif)]]

---

# TIL: No straight lines

.center[.p80[![Vader boards](content/images/vader-boards.gif)]]

---

# TIL: No straight lines

.center[.p80[![Vader is lost](content/images/lost-vader.gif)]]

---

# Rust's story

- **Minimal core:**
    - Ownership and borrowing, traits
    - Zero-cost abstractions
    - No runtime, no garbage collector, not even a concept of threads
- **Extended with libraries:**
    - Fundamentals: vectors, hashmaps, reference-counted pointers
    - Higher-level: Work-stealing runtimes, servers, database links

---

# Rust's story

```rust
fn dot_product(vec1: &[u32], vec2: &[u32]) -> u32 {
  vec1
    .iter()
    .zip(vec2)
    .map(|(a, b)| a * b)
    .sum()
}
```

--

- Compiles to vectorized loop

---

# Rust's story

```rust
fn dot_product(vec1: &[u32], vec: &[u32]) -> u32 {
  vec1
    .par_iter()
    .zip(vec2)
    .map(|(a, b)| a * b)
    .sum()
}
```

--

.line3[![Arrow](content/images/Arrow.png)]

- Vectorized loops across a work-stealing scheduler

---

# Rust's story

```rust
fn dot_product(vec1: &[u32], vec2: &[u32]) -> u32 {
  let mut counter = 0;
  vec1
    .pari_ter()
    .zip(vec2)
    .map(|(a, b)| {
       counter += 1; // <-- ERROR
       a * b
    })
    .sum()
}
```

---

# But it wasn't always this way

- Early days:
  - Erlang-like tasks, a runtime, garbage collector
  - Ownership added as a perf optimization
  - Unsound, easy to crash

---

# But it wasn't always this way

- Introduced region types to gain flexibility
  - But retained an ocaml- or C-like mutation model

---

# But it wasn't always this way

- Moved to inherited mutability and "IMHTWAMA"
  - `T` -- owned
  - `&T` -- shared, immutable
  - `&mut T` -- **unique**, mutable 

--

.center[.p80[![IMHTWAMA](content/images/imhtwama.png)]]


---

# But it wasn't always this way (part 2)

- Iterated also on a few other axes:
    - generic programming
        - simple generics => classes+interfaces => traits
    - threads and runtime
        - moved into libraries, out from language

---

# What *was* always true

- uncompromised efficiency
- safety and correctness
- CoC and culture emphasizing **curiosity and deep research**

--

**Focused on finding the best answer, not on winning the argument.**

---

# TIL: True artists ship

.center[.p80[![Marge vacuums her vacuum](content/images/marge-vacuums-vacuum.gif)]]

---

# TIL: True artists ship

.center[.p60[![I would rather not do it at all](content/images/would-rather-not-at-all.gif)]]

---

# 2014 or so: Rust 1.0 is coming

*everybody hide*

.center[.p40[![dog-hide](content/images/dog-hide-suitcase.gif)]]

---

# Release channels

- Stable release every 6 weeks

--
- Nightly release every night (mostly)

---

# Feature gates

```rust
#![feature(xxx)]

fn main() { }
```

Usable only on **nightly**

--

Feature gates are **infectious**

---

# TIL: People don't care about your warnings

.center[.p80[![Kitten hi mommy](content/images/kitten-hi-mommy.gif)]]

---

# TIL: People don't care about your warnings

.center[.p80[![Kitten hi mommy](content/images/dog-jump-sled.gif)]]

---

# Unstable dependency? Use nightly


---

# Future-compatibility warnings

- Grace period to correct code that should not have been accepted
- But: lint warnings are suppressed on dependencies

---

# TIL: Account for versioning 

.center[.p80[![Scientifically proven](content/images/homer-proven.gif)]]

???

When we prove a type system to be *sound*, we mean that the program as
written is fine.

---

# TIL: Account for versioning 

.center[.p80[![Things have changed](content/images/dylan-changed.gif)]]

???

---

# Semantic versioning ("semver")

- Version 1.0
- Version 1.1 -- adds new features, but 1.0 users still work
- Version 1.1.1 -- just bug fixes, you definitely want this
- Version 2.0 -- 1.x users may require changes

---

# Traits

```rust
trait Hash {
  fn hash(&self) -> u32;
}

struct HashSet<K: Hash> {
  //           ^^^^^^^
}
```

---

# Impls

```rust
struct u32 { .. }

impl Hash for u32 {
  fn hash(&self) -> u32 {
    *self
  }
}
```

---

# "The HashTable problem"

- Initial designs permitted "scoped impls"

```rust
// Scope 1: Hash of u32 is identity
fn create_set() -> HashSet<u32> {
  let mut set = HashSet::new();
  set.insert(22);
  set.insert(44);
  set
}
```

--

```rust
// Scope 2 has some other definition of Hash for u32
fn use_set() {
  let set = create_set();
  set.contains(22) // ???
}
```

---

# Coherence

- At most one impl for any type
- Relatively easy to enforce within one crate: check!
- Across crates:
  - Impl must define *either* the self-type *or* the trait type
  - "Orphan rule" from Haskell

---

# Checking for duplicates

Obvious collision:

```rust
impl Hash for u32 { ... }
impl Hash for u32 { ... }
```

---

# Checking for duplicates

What about this:

```rust
trait AsU32 { 
  fn as_u32(&self) - &u32;
}

impl Hash for u32 { ... }
impl<T: AsU32> Hash for T { ... }
//   ^^^^^^^^
```

---

# Checking for duplicates

If this impl exists:

```rust
impl AsU32 for u32 { ... }
```

then these two impls overlap:

```rust
impl Hash for u32 { .. }
impl<T: AsU32> Hash for T { .. }
```

so which should we use?

---

# Checking for duplicates

.center[.p80[![RFC 48](content/images/rfc-48.png)]]

---

# Enter: time and versioning 

```rust
// Crate 1, v1.0:
trait AsU32 { }
struct MyType { }

// Crate 2, v1.0:
trait Hash { }
impl<T: AsU32> Hash for T { }
impl Hash for crate1::MyType { }
```

---

# In v1.1, crate 1 adds an impl

```rust
// Crate 1, v1.1:
trait AsU32 { }
struct MyType { }
impl AsU32 for MyType { } // added in 1.1...

// Crate 2, v1.0:
trait Hash { }
impl<T: AsU32> Hash for T { }
impl Hash for crate1::MyType { } // ...now we have a conflict
```

---

# Negative reasoning

```rust
trait Hash { }
impl<T: AsU32> Hash for T { }
impl Hash for crate1::MyType { } // ...now we have a conflict
```

--

- Crate 2 is employing a sort of **negative reasoning** here:
  - no intersection because `not { crate1::MyType: AsU32 }`

--
- These rules make it a **semver breaking change** to add an impl:
  - not ok

---

# Rebalancing

.center[.p80[![RFC 48](content/images/rfc-1023.png)]]

---

# Coherence still a problem

- The rules here continue to bedevil us
- Very tricky balancing act

---

# Another semver related change

```rust
struct Point {
  x: u32,
  y: u32,
}

fn example() {
  let p1 = Point { x: 22, y: 44 };
  let p2 = p1;
  draw(p1);
  draw(p2);
}
```

--

.line8b[![Hi](content/images/Arrow.png)]

--

- In earlier versions of Rust, this would be a copy
    - but then adding `z: Box<u32>` is a breaking change
- Now, it's a move
    - must explicitly `impl Copy for Point`

---

# But auto traits

Rust has a `Send` trait: can something be sent across threads safely?

Most things are `Send`:

```rust
u32
Vec<Element>
HashMap<Key, Value>
```

---

# Reference counting is not `Send`

Reference counting:

```rust
Rc<Vec<Element>>
```

But atomic reference counting is (though it's slower):

```rust
Arc<Vec<Element>>
```

---

# Send is "automatic"

```rust
struct Context {
  data: HashMap<SomeKey, SomeValue>
}
```

Here, we assume that `Context: Send`. 

--

So if you changed to `Rc<HashMap<..>>`, that's a semver violation.

--

Considerations

- re-usable libraries tend to eschew `Rc`, preferrering ownership
- we wanted to enable parallelism deeply
- basically a "bet" that this wouldn't be a big deal *in practice*
    - so far, seems to have paid off

---

# More versioning cases

- Exhaustive matching on enums
    - What if people add variants?
- Definite tension:
    - Tight construction
    - Evolving needs
- Ye Olde "Expression Problem"

---

# TIL: Many definitions of "simple"

.center[.p80[![Just that simple](content/images/just-that-simple.gif)]]

---

# Borrow checking initially

```rust
fn foo() {
  let mut v = vec![1, 2, 3];
  let p = &v; // ----+
  v.push(3);  //     |
} // <---------------+ scope of the borrow
```

---

# Borrow checking now

```rust
fn foo() {
  let mut v = vec![1, 2, 3];
  let p = &v; // ----+
  v.push(3);  // <---+
}
```

---

# TIL: Error messages are the first teacher

--

.center[.p80[![cat reading](content/images/cat-rtfm.gif)]]

---

# People just want to get started!

.center[.p80[![cat reading](content/images/homer-working-out.gif)]]

---

# Example

.center[.p80[![Rust error tweet](content/images/tweet-rust-errors.png)]]

---

.center[.p80[![racket errors](content/images/racket-errors-1.png)]]

.center[.p60[![racket errors](content/images/racket-errors-2.png)]]

---

# TIL: Stroustrup's rule

.center[.p80[![Just that simple](content/images/suspicious-svu.gif)]]

---

# TIL: Stroustrup's rule

.center[.p80[![Just that simple](content/images/meh-new-year.gif)]]

---

# Rust error handling

```rust
enum Result<T, E> {
  Ok(T),
  Err(E),
}
```

---

# Rust error handling

```rust
fn process_data() -> Result<u32, Error> {
  match read_data() {
    Err(e) => e,
    Ok(d) => process(d),
  }
}

fn read_data() -> Result<Data, Error> {
  ...
}
```

---

# Rust error handling

```rust
fn process_data() -> Result<u32, Error> {
  let data = try!(read_data()); // match, return if err
  process(data)
}

fn read_data() -> Result<Data, Error> {
  ...
}
```

try:

- matches and, in the case of error, returns the error

--

.line2[![Arrow](content/images/Arrow.png)]


---

# Rust error handling

```rust
fn process_data() -> Result<u32, Error> {
  let data = read_data()?;
  process(data)
}
```

--

.line2[![Arrow](content/images/Arrow.png)]

---

# Controversy

- `?` was, until recently, one of the most controversial RFCs
- We still debate about how/whether to extend it
- Was it too hard to see?

---

# TIL: Hard decisions are hard...but not impossible

.center[.p80[![Never clear](content/images/never-clear.gif)]]

---

# TIL: Hard decisions are hard...but not impossible

.center[.p80[![Never clear](content/images/ergonomics.png)]]

---

# Three axes

How much information do you need to confidently understand what a
particular line of code is doing, and how hard is that information to
find?

--
- **Applicability:** Where can you elide? Is there a heads-up?

--
- **Power:** How much can it change behavior?

--
- **Context-dependence:** How much context do you need?


The larger on one axis, the smaller the others must be.

---

# The `?` sugar

```rust
fn process_data() -> Result<u32, Error> {
  let data = read_data()?;
  process(data)
}
```

- Applicability: Small, you have a heads-up
- Power: High, it can radically change control-flow (return)
- Context-dependence: Medium, have to consult return type

---

# Type inference

```rust
fn squares(data: impl Iterator<Item = u32>) -> Vec<u32> {
  data.map(|element| element*2).collect()
}
```

- Applicability: Medium -- always intrafunction.
- Power: High, it can radically change control-flow (return)
- Context-dependence: Varies -- mostly intrafunction, but signatures matter

But compare to global inference like in ML.

---

# TIL: Fundamental tradeoffs are often anything but

--

.center[.p80[![dogs wrestling over ball](content/images/dogs-wrestling-over-ball.gif)]]

---

# Example: Error handling

- Error codes:
    - easy to forget, obscures the happy path
    - but you can see errors and reason about what will happen
- Exceptions:
    - can't forget, happy path is clear
    - but hard to tell which errors can occur and where
- Compromise:
    - in Rust, the `?` operator
    - in Swift, the `try` keyword

---

# Example: Zero-cost abstractions

- Two objectives in tension:
    - high-level code
    - high-performance code
- Zero-cost abstractions:
    - set of techniques to resolve those tensions

---

# No free lunch

- Consider Rust's borrow checker:
    - you can have threads and shared memory
    - without data-races
- Too good to be true?

---

# Technique: Map the space

- Nothing magic:
    - when you find tension, dig into it
    - identify your requirements firmly
- Often:
    - some requirement can be refined
- But this takes time

---

# Sometimes the resolution is not in the language

- Alternatives:
    - conventions
    - lints
    - documentation or good error messages

---

# TIL: Pluralism

One of the most momentous things to happen to Rust?

--

.center[.p80[![the Beast reading](content/images/tweet-wycats-loves-rust.png)]]

---

.center[.p80[!["Let our powers combine"](content/images/captainplanet.gif)]]

---

.center[.p100[!["captain planet"](content/images/captainplanet2.gif)]]

--

.rust-label[Rust]

--

.cpp-label[C++]

--

.js-label[JavaScript]

--

.academia-label[Academica]

--

.new-label[New Developers]

---

# Pluralism, defined

> a condition or system in which two or more states, groups, principles, sources of authority, etc., coexist.

.citation[https://www.google.com/search?q=pluralism&oq=pluralism&aqs=chrome.0.69i59j0l5.1076j1j1&sourceid=chrome&ie=UTF-8]

---

.center[.p80[![tug of war](content/images/tugofwarfail.gif)]]

---

.center[.p80[![together](content/images/surprise.gif)]]

---

.center[.p80[![together](content/images/together.gif)]]

---

# In a word, communicate.

--

Communicate.

--

Communicate.

--

Communicate.

---

# Problem: Random PRs and random ideas

.center[.p60[![together](content/images/badsurprise.gif)]]

---

# Articulate the vision

---

# Roadmaps

- `#rust2018`, `#rust2019` blog posts
- don't be afraid to make your case

???

The truth will out. Make your case.

---

# No new criteria

> Importantly though: There was almost zero participation from members of the core team in the public discussion thread. Why is it a good idea for members of the core team to be entitled to skip this, to keep their reasoning and discussions to themselves, and only reveal it together with their final decision?

--

- Use meetings to explore space
- Create summary documents exploring the spcae
- Post a tentative decision

---

# Summary documents

- Collect and collate the arguments and trade-offs
- Tell your "narrative"

---

# Communicate progress and plans

- Honestly, Rust is really not good at this. Working on it. =)

---

# Haters gonna hate

Doesn't mean you have to.

--

Ask yourself: why are they angry?

--

But don't let them bully you.

---

# Core team must, but the core team can't

.center[.p60[![cat on keyboard](content/images/cat-on-keyboard.gif)]]

---

# Projects don't need a dictator

But they can be useful.

And they do need leaders.

More than one way to do it.

---

# TIL: You all are a great audience.

Thanks for listening! <3

