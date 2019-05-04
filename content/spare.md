---

name: oversharing
# Oversharing

```java
Applet applet = new Applet();
Vector names = new Vector();
names.add("Harry Potter");
applet.setAccessControlList(names);
names.add("Voldemort");
```

---

template: oversharing

.line4[
![Point at call to `setAccessControlList`](content/images/Arrow.png)
]

- `setAccessControlList` also checked that the list is reasonable

---

template: oversharing

.line5[
![Point at call to `setAccessControlList`](content/images/Arrow.png)
]


- `setAccessControlList` also checked that the list is reasonable
- Uh oh!


---

name: entry

# Example: the entry API

```rust
my_map
  .entry(some_key)
  .or_insert(Vec::new())
  .push(element);
```

---

template: entry

.line1[
![Arrow](content/images/Arrow.png)
]

- Given a hashmap `K -> Vec<V>`,

---

template: entry

.line2[
![Arrow](content/images/Arrow.png)
]

- Given a hashmap `K -> Vec<V>`,
  - lookup the value for the key `K` (if any)

---

template: entry

.line3[
![Arrow](content/images/Arrow.png)
]

- Given a hashmap `K -> Vec<V>`,
  - lookup the value for the key `K` (if any),
  - insert an empty vector for `K` if nothing exists,

---

template: entry

.vec-new[
![Arrow](content/images/Arrow.png)
]

- Given a hashmap `K -> Vec<V>`,
  - lookup the value for the key `K` (if any),
  - insert an empty vector for `K` if nothing exists,

---

template: entry

.line4[
![Arrow](content/images/Arrow.png)
]

- Given a hashmap `K -> Vec<V>`,
  - lookup the value for the key `K` (if any),
  - insert an empty vector for `K` if nothing exists,
  - push `element` onto the vector for `K`.

---

template: entry

- Given a hashmap `K -> Vec<V>`,
  - lookup the value for the key `K` (if any),
  - insert an empty vector for `K` if nothing exists,
  - push `element` onto the vector for `K`.
- **Ergonomic, yes.** But also:
  - **Efficient:** reuse hash, internal indices from first lookup.
  - **Robust:** `entry` borrows the map, so you can't intermix multiple insertions.




---

name: seq-iter

# Sequential iterators

```rust
fn load_images(paths: &[PathBuf]) -> Vec<Image> {
  paths
    .iter()
    .map(|path| Image::load(path))
    .collect()
}
```

---

template: seq-iter

.line1[![Point at `paths`](content/images/Arrow.png)]

---

template: seq-iter

.line3[![Point at `iter`](content/images/Arrow.png)]

- Create an iterator over paths

---

template: seq-iter

.line4[![Point at `iter`](content/images/Arrow.png)]

- Create an iterator over paths
- For each path, invoke `Image::load`

---

template: seq-iter

.line5[![Point at `iter`](content/images/Arrow.png)]

- Create an iterator over paths
- For each path, invoke `Image::load`
- Collect loaded images into a vector

---

name: rayon

# Parallel iterators

```rust
fn load_images(paths: &[PathBuf]) -> Vec<Image> {
  paths
    .par_iter()
    .map(|path| Image::load(path))
    .collect()
}
```

.line3[![Point at `par_iter`](content/images/Arrow.png)]

- One change to execute in parallel

---

name: rayon-race

# Parallel iterators

```rust
fn load_images(paths: &[PathBuf]) -> Vec<Image> {
  let mut jpegs = 0;
  paths
    .par_iter()
    .map(|path| {
      if path.ends_with(".jpg") {
        jpegs += 1;
      }
      Image::load(path)
    })
    .collect()
}
```

---

template: rayon-race

.line2[![Point at `jpegs`'](content/images/Arrow.png)]

---

template: rayon-race

.line7[![Point at `jpegs`'](content/images/Arrow.png)]

---

.center[![saved by the compiler](content/images/saved-by-compiler.png)]

> **The Rust compiler just saved me from a nasty threading bug.** I was working on cage (our open source development tool for Docker apps with lots of microservices), and I decided to parallelize the routine that transformed docker-compose.yml files. This was mostly an excuse to check out the awesome rayon library, but it turned into a great example of what real-world Rust development is like.

.citation[`https://blog.faraday.io/saved-by-the-compiler-parallelizing-a-loop-with-rust-and-rayon/`]

---
