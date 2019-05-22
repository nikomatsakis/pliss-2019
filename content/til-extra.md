---

name: rusts-parallel-story-1

# Rust's parallel story

```rust
fn load_images(
  paths: &[PathBuf],
) -> Vec<Image> {
  paths
    .iter()                 // Iterate over the paths...
    .map(|path| {           // ...invoke closure...
        Image::load(path)   // ...load image...
    })
    .collect()              // ...construct a vector.
}
```

---

template: rusts-parallel-story-1

.line5[![Hi](content/images/Arrow.png)]

---

template: rusts-parallel-story-1

.line6[![Hi](content/images/Arrow.png)]

---

template: rusts-parallel-story-1

.line7[![Hi](content/images/Arrow.png)]

---

template: rusts-parallel-story-1

.line9[![Hi](content/images/Arrow.png)]

---

# Rust's parallel story

```rust
fn load_images(
  paths: &[PathBuf],
) -> Vec<Image> {
  paths
    .par_iter()             // Iterate in parallel
    .map(|path| {
        Image::load(path)
    })
    .collect()
}
```

.line5[![Hi](content/images/Arrow.png)]

---

# Data-races

```rust
fn load_images(
  paths: &[PathBuf],
) -> Vec<Image> {
  let mut count = 0;
  paths
    .par_iter()
    .map(|path| {
        count += 1;       // Data race! Won't compile.
        Image::load(path)
    })
    .collect()
}
```

.line8[![Hi](content/images/Arrow.png)]


---

# Rust before 2011...

- Initially, Rust had:
  - Tasks that did not share data
  - Garbage collected types (`@T`)
  - Immutable data structures, copy on write
  - A borrowing system based on "parameter modes"
  
```
let x = [1, 2, 3];
let y = x;
x.push(...); // copy on write
```

---

# Ownership as a perf optimization

- Added ownership sigil `~` as a perf optimization

```
let x = ~[1, 2, 3];
let y = x;
x.push(...); // copy on write
```

---

name: adding-regions

# Adding regions

```rust
type MyContext = {
  mut count: u32
};

fn foo(&&x: MyContext) {
  x.count += 1; // Legal!
}
```

- Rust always had references, but as a 'parameter mode'

---

template: adding-regions

.line2[![Arrow](content/images/Arrow.png)]

---

template: adding-regions

.line5[![Arrow](content/images/Arrow.png)]

---

template: adding-regions

.line6[![Arrow](content/images/Arrow.png)]

---

name: adding-regions

# Regions become part of type


---

---

# Ownership

```rust
let v = vec![1, 2, 3, 4];
let w = v; // moves v
access(v); // error!
```

--

.line3[![Arrow](content/images/Arrow.png)]

---

name: shared-borrow

# Shared borrow

```rust
let mut v = vec![1, 2, 3, 4];
v.push(5);
let w = &v;     // Shared borrow starts here...
v.len();        // ...ok, just reads...
v.push(6);      // ...error, immutable while borrowed...
access(w);      // ...shared borrow ends here, last use of `w`.
v.push(7);      // mutable again
```

---

template: shared-borrow

.line2[![Arrow](content/images/Arrow.png)]

---

template: shared-borrow

.line3[![Arrow](content/images/Arrow.png)]

---

template: shared-borrow

.line4[![Arrow](content/images/Arrow.png)]

---

template: shared-borrow

.line5[![Arrow](content/images/Arrow.png)]

---

template: shared-borrow

.line6[![Arrow](content/images/Arrow.png)]

---

template: shared-borrow

.line7[![Arrow](content/images/Arrow.png)]

---

name: mutable-borrow

# Mutable borrow

```rust
let mut v = vec![1, 2, 3, 4];
v.push(5);
let w = &mut v; // Mutable borrow starts here...
v.len()         // ...error, borrowed...
v.push(6);      // ...error, borrowed...
access(w);      // ...shared borrow ends here, last use of `w`.
v.push(7);      // mutable again
```

---

template: mutable-borrow

.line4[![Arrow](content/images/Arrow.png)]

---

template: mutable-borrow

.line5[![Arrow](content/images/Arrow.png)]

---

template: mutable-borrow

.line6[![Arrow](content/images/Arrow.png)]

???

`w` could write to `v`.

---

# Ownership + borrowing

- Benefits:
  - ensures **data-race freedom**
  - prevents sequential errors like **dangling pointers** 
- Threads live in libraries (rayon, tokio, crossbeam)
- Ownership can model a lot of capabilities

--
  - ps, not our idea; this has been the goal since the foundational
    work like [Linear Types Can Change The World][lt]

[lt]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.55.5439&rep=rep1&type=pdf

---

# Rust's parallel story

```rust
fn load_images(
  paths: &[PathBuf],
) -> Vec<Image> {
  paths
    .iter()                 // Iterate over the paths...
    .map(|path| {           // ...invoke closure...
        Image::load(path)   // ...load image...
    })
    .collect()              // ...construct a vector.
}
```

---

# Rust's parallel story

```rust
fn load_images(
  paths: &[PathBuf],
) -> Vec<Image> {
  paths
    .par_iter()             // Iterate in parallel
    .map(|path| {
        Image::load(path)
    })
    .collect()
}
```

.line5[![Hi](content/images/Arrow.png)]

---

# Data-races

```rust
fn load_images(
  paths: &[PathBuf],
) -> Vec<Image> {
  let mut count = 0;
  paths
    .par_iter()
    .map(|path| {
        count += 1;       // Data race! Won't compile.
        Image::load(path)
    })
    .collect()
}
```

.line8[![Hi](content/images/Arrow.png)]

---

# Presentation is kinda' slick too

- Define data races
- Show how data races leads to ownership + borrowing
- Show other applications

---

# Route we took

- No ownership, very limited borrowing
- Added ownership as a perf optimization


---

# TIL: Fundamentals matter

.center[.p60[![the Beast reading](content/images/beast-book.gif)]]

???

- No way we could have built a language like Rust

---

# TIL: Fundamentals matter

.center[.p60[![tree falling](content/images/tree-fall.gif)]]

???

- But I won't lie: there's a real challenge if you're trying to ship a language
- "Reality" kinds of gets in the way
- A million things you have to do to get the language done:
    - Figure out how it's going to work

---

# Unsafe code guidelines

```rust
fn foo(
  vec: Vec<u32>, 
) {
  // Can we optimize to `vec[0] += 2`..?
  vec[0] += 1;
  bar();
  vec[0] += 1;
}
```

---

# Unsafe code guidelines

```rust
fn foo(
  vec: Vec<u32>, 
) {
  // Can we optimize to `vec[0] += 2`..?
  vec[0] += 1;
  bar();
  vec[0] += 1;
}

fn bar() {
  // .. depends if we think it's legal for
  // this unsafe code to observe `vec`.
  unsafe { .. }
}
```

---

# Hard call

- Things we've not created:
    - Rust reference
    - Formal semantics
    - Rules for unsafe code
- Things we have created:
    - Language used in production at a number of major companies





---

# TIL: Hard to make time for fundamentals

--

.center[.p80[![SML Definition](content/images/sml-defn.png)]]

---

# The rust reference

--

.center[.p80[![Rust reference](content/images/rust-reference.png)]]

---

.center[.p60[![tree falling](content/images/tree-fall.gif)]]

???

- But I won't lie: there's a real challenge if you're trying to ship a language
- "Reality" kinds of gets in the way
- A million things you have to do to get the language done:
    - Figure out how it's going to work

---

# Unsafe code guidelines

```rust
fn foo(
  vec: Vec<u32>, 
) {
  // Can we optimize to `vec[0] += 2`..?
  vec[0] += 1;
  bar();
  vec[0] += 1;
}
```

---

# Unsafe code guidelines

```rust
fn foo(
  vec: Vec<u32>, 
) {
  // Can we optimize to `vec[0] += 2`..?
  vec[0] += 1;
  bar();
  vec[0] += 1;
}

fn bar() {
  // .. depends if we think it's legal for
  // this unsafe code to observe `vec`.
  unsafe { .. }
}
```

---

# "It's ok, Niko"

- Things we've not created:
    - Rust reference
    - Formal semantics
    - Rules for unsafe code
- Things we have created:
    - Language used in production at a number of major companies
