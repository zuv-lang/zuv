# Contributing to Zuv 🚀

Thank you for your interest in contributing to **Zuv**! We are building a modern, fast, compiled programming language optimized for minimal typing, maximum expressiveness, and zero-overhead memory safety.

---

## 💡 Our Core Philosophy

When writing or modifying Zuv code, remember our motto:
> **"Write Less, Do More. Code like you're chatting on a social media app."**

- **Zero Boilerplate**: Avoid unnecessary punctuation, parentheses, and keywords where the context is clear.
- **Parenthesis-Free Direct Calling**: `func arg1, arg2` rather than `func(arg1, arg2)`.
- **Minimalist Keywords**: `prnt`, `ret` or `->`, `mut`, `wh`, `fr`, `brk`, `cont`, `obj`, `asc`, `awt`, `spn`, `jn`, `wrk`.

---

## 🛠️ How to Contribute

### 1. Reporting Issues
- Check existing issues before opening a new one.
- Provide a clear, minimal reproducible example (`.zv` source code) demonstrating the bug.
- Include your operating system and environment details.

### 2. Developing & Submitting Code
1. **Fork the repository** on GitHub ([https://github.com/zuv-lang/zuv](https://github.com/zuv-lang/zuv)).
2. **Clone your fork**:
   ```bash
   git clone https://github.com/<your-username>/zuv.git
   cd zuv
   ```
3. **Create a branch** for your feature or bug fix:
   ```bash
   git checkout -b feature/my-cool-feature
   ```
4. **Build and test**:
   ```bash
   zuv check src/main.zv
   zuv build src/main.zv
   ```
5. **Commit your changes**:
   ```bash
   git commit -m "feat(parser): add support for XYZ feature"
   ```
6. **Push to your fork & open a PR**:
   ```bash
   git push origin feature/my-cool-feature
   ```

---

## 📜 Coding Conventions
- **Naming**: Use camelCase for functions and variables (`addNumbers`, `totalCount`), PascalCase for structs and types (`Point`, `HttpResponse`).
- **Formatting**: Keep code clean, using 4 spaces per indentation level.
- **Pure Zuv**: All compiler source files in `src/` must be written in valid, standard Zuv (`.zv`).

---

## 🤝 Community & Communication
- Please follow our [Code of Conduct](CODE_OF_CONDUCT.md) in all project spaces.
- Join the discussion in our GitHub Discussions tab and issue tracker.
