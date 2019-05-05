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

# Why should you care about IDEs?

???

--

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
- It affects your language design -- dependencies matter
- It's not that hard -- if you do it from the start!
- You might want this setup anyway

---

# Strict phase separation is a lie, man

.center[
.p80[
![That's just, like, your opinion man](content/images/opinion.gif)
]
]

---

# Strict phase separation is a lie, man

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

# Strict phase separation is a lie, man

But then, what if I were to do this?

```rust
const LEN: u8 = DATA[0] + DATA[1] + DATA[2];
const DATA: [u8; LEN] = [1, 1, 1];
```

Uh oh.

???


---

