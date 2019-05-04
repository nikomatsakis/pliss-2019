class: center
name: title
count: false

# Responsive compilers

.me[.grey[*by* **Nicholas Matsakis**]]
.page-center[.p80[![Rust Logo](content/images/rust-logo-512x512.png)]]
.king[![Drawing of King](content/images/King.png)]
.queen[![Drawing of Queen](content/images/Queen.png)]
.citation[`https://github.com/nikomatsakis/rust-latam-2019`]

---

# Does this look familiar?

![Image result for dragon book](https://images-na.ssl-images-amazon.com/images/I/51FWXX9KWVL._AC_UL320_SR248,320_.jpg)

---

# Pipelines and passes

Traditional compiler model is a series of **passes**:

- `lex(source) -> tokens`
- `parse(tokens) -> ast`
- `semantic_analysis(ast)`, `type_check(ast)`
- loop:
  - apply optimizations
- etc

--

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

![Image result for vscode](https://raw.githubusercontent.com/be5invis/vscode-custom-css/master/screenshot.png)

# 
