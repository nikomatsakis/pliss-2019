XXX

GIFs

"Open your eyes to the expanding universe" 

https://giphy.com/gifs/science-gif-richard-dawkins-249l5Py3jt4Jy

"Homer Simpson -- brain bust out of head"

https://giphy.com/gifs/homer-simpson-horror-12Itd4w3M5HqRa

"Collapsing grain tower"

https://giphy.com/gifs/4VXVA0H5geRSQdw1Pe

"Puzzle collapsing -- omg"

https://giphy.com/gifs/collapse-fCkd0tasUZpDi

Homer "why didn't somebody tell me what I was volunteering for"


https://giphy.com/gifs/season-10-the-simpsons-10x8-xT5LMwaX8KTFY3nxYY

XXX

# TIL



# Things I Learned

## Been working on Rust for 8 years

- show picture of Daphne as a baby
- lots of languages
- relatively few *widely used production languages*

## Exploration / Iteration

- Research spiral?
- Rust's Parallel Story
    - Rayon
    - Tasks
    - Borrow check
    - IMHTWAMA
    - Pythonesque
    - Rayon
- "true artist's ship"
    - when is something "done"?
    - how can we preserve future flexibility
    - something we faced with 1.0
    - train model, feature gates
        - why feature gates are not on stable: need a blunter message
        - maybe too blunt?
- Editions
    - Example:
      - Rust modules?
    - Rust 2015 vs Rust 2018
        - core idea: an underlying "common concept"
        - compile down to that
        - key constraints: seamless interop between editions
        - similar to Java/C++
    - the future?
        - maybe something about minimal cores, moving into libraries?
        
## Table stakes

- cargo
- IDE support

## Integration

- I spent a lot of time reading and learning
  about type systems
- Left me blind to some problems
    - not saying that these are unstudied =)
- semver
    - explain how traits and type classes work
    - explain "the hashtable problem"
    - talk about the haskell term: coherence
    - orphan rule
    - little orphan impl considerations
    - rebalancing coherence
    - still a "live problem" today
- comes up a lot:
    - auto traits, copy trait
    - exhausting matching on enums (`#[non_exhaustive]`)
    - implied bounds
- "what is simple"
    - lambda calculus can be defined in two rules
        - but try to program in it
        - seems like an abstract problem?
    - borrow check
        - initial version with scoped rules
        - new version with NLL rules
        - polonius
        - going further
- "what is explicit"
    - Rust uses `Result` for error handling
    - elegant, minimal, but try programming in it
    - early stab: the `try!` macro
    - introducing the `?` operator
    - the debate rages:
        - too hard to see!
        - maybe we should use monads?
            - connected to "true artists ship"
    - boats wrote a nice blog post here entitled ["Not explicit"]
        - explicit means "you can figure it out from the source"
        - example:
            - in Rust, struct declaration tells you a lot
            - struct vs heap, pointer not pointer
            - does not tell you the *order* of the fields in memory, though
        - consider `try!` vs `?` though
            - here, this is more about *noise*
            - in other cases, the goal is more to make it *annoying*
                - `let x = Box::new(MyStruct);`
                - vs in Swift:
                    - `indirect struct MyStruct`
    - aturon's blog post ["Rust's language ergonomics initiative"][ergonomics]s
        - How much information do you need to confidently understand
          what a particular line of code is doing, and how hard is
          that information to find?
        - *Applicability.* Where are you allowed to elide implied
          information? Is there any heads-up that this might be
          happening?
        - *Power.* What influence does the elided information have?
          Can it radically change program behavior or its types?
        - *Context-dependence.* How much of do you have to know about
          the rest of the code to know what is being implied, i.e. how
          elided details will be filled in? Is there always a clear
          place to look?
        - goal: no one dimension should be very large -- if one is, the other two should be small
    - examples:
        - `?` -- 
            - applicability is "small", you have a heads-up
            - power -- fairly high, it can cause a return or branch to occur
            - context-dependence -- somewhat, need to consider the return type
        - type annotations
            - applicability -- in Rust, always intrafunction, helps a lot
            - power -- very high
            - context-dependence -- intrafunction
        - lifetime elision -- a bit subtle
        - implied bounds
            - does well on this test, the tricky one is the semver test
        - ownership vs borrowing in Rust -- subtle to go into
- "what is obvious"
    - [Stroustroup's rule][sr]
        - For new features, people insist on LOUD explicit syntax.
        - For established features, people want terse notation.

## Fundamentals

- "Move fast and break things"
- Academic collab.
- Polonius, Chalk
    - much more formally defined
    - but not yet proven
- Unsafe code guidelines
    - huge, complex problem
    - C got away with it... =)
- No Rust reference
- This will be an area of focus for us

## Community interaction

- RFCs
- Working groups
- challenges


["Not explicit"]: https://boats.gitlab.io/blog/post/2017-12-27-things-explicit-is-not/
[ergonomics]: https://blog.rust-lang.org/2017/03/02/lang-ergonomics.html
[sr]: https://thefeedbackloop.xyz/stroustrups-rule-and-layering-over-time/


TIL IDEAS:

- TIL: gifs are good
    - embrace the gif
- TIL: pluralism
    - gif: tweet-wycats-loves-rust.png -- one of the great things to happen to rust
    - gif: captainplanet.gif
    - gif: captainplanet2.gif
    - gif: tugofwarfail.gif
    - languages don't need a dictator
- TIL: Fundamentals are useful
    - gif: beast-book.gif
    - no way we could do rust without the fundamentals
    - challenge: reality getting in the way (falling tree)
    - unsafe code guidelines
        - huge, complex problem
        - C got away with it...
    - rust reference
    - ongoing challenge
- TIL: Desugaring is good, more of that
    - gif: 
    - gif: simplify-man.gif
- TIL: Error messages as the first teacher
    - trying to teach the borrowing rules?
    - cite the racket folks here
    - `await foo`
- TIL: Fundamental tradeoffs are often anything but
    - gif: einstein-loop.gif
    - gif: dogs-wrestling-over-ball.gif
    - data races: X + Y + Z -- no sharing/borrowing *at the same time*
    - `?` sugar: exceptions are silent, error codes are obvious...
    - Zero-cost abstractions: XXX
    - others?
    - walk through my blog post here?
- TIL: Mentoring instructions
    - gif: ne-patriots-left-hanging.gif
    - gif: arrested-dev-left-hanging.gif


- TIL: pluralism
    - gif: tweet-wycats-loves-rust.png -- one of the great things to happen to rust
    - gif: captainplanet.gif
    - gif: captainplanet2.gif
    - gif: tugofwarfail.gif
    - languages don't need a dictator


GIFs:

- know-yourself-find-truth.gif
- find-freedom-on-this-canvas.gif
- carl-sagan-brain.gif
- engage.gif -- "with community" :)







---

# Life of a feature

- Design, RFC
- Land on nightly
  - experimentation, tinker with design
- Stabilization
  - available for all! :tada:
  
---

# Shortcomings

- Feedback before stabilization
- Understanding the current state of things

---

# Editions

The problem:

Want the ability to make small changes

???

- Mention C++ committee feedback

---

name: the-dyn-keyword

# The dyn keyword

Rust in 2015:

```rust
trait Write {
    fn write(&mut self, data: &[u8]) { ... }
}
```

---

template: the-dyn-keyword

Parametric polymorphism...

```rust
fn write_to<T: Writer>(out: &mut T) {
//                               ~
  out.write(...); // <-- statically known
}
```

---

template: the-dyn-keyword

...vs "ad-hoc" polymorphism:

```rust
fn write_to(out: &mut Writer) {
//                    ~~~~~~ "object"
  out.write(...); // dispatched through a vtable
}
```

---

# The dyn keyword

But we wanted to make a change. First, a sugar:

```rust
fn write_to(out: &mut impl Writer) { ... }
//                    ~~~ "some type that implements"
```

desugared to

```rust
fn write_to<T: Writer>(out: &mut T) { ... }
```

---

# The dyn keyword

Then introduce `dyn`:

```rust
fn write_to(out: &mut dyn Writer) {
//                    ~~~ "dynamic"
  out.write(...); // dispatched through a vtable
}
```

--

Problems:

- Existing code didn't use it
- New keyword (`dyn`)

---

# Ecosystem compatibility

- Allow crates to migrate at their own pace
- Provide tooling to help with the migration
- Key goal: **seamless interop between editions**
