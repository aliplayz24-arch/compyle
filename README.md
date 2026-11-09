# compyle

A tiny, cross-platform tool that turns a Python script into a single native executable. It is a rewrite of [pyle](https://github.com/aliplayz24-arch/pyle) (Haskell) in JavaScript, to run on Bun.

`compyle` compiles your script to Python bytecode, embeds that bytecode in a small C program, and builds it with your C compiler. The resulting binary starts the system's CPython and runs your code, so every library that works with `python3 main.py` works the same way. It has been tested with **PyQt6 / QtWebEngine** and **Pygame**.

> **What compyle is not:** it does not translate Python into native machine code, so it is not faster than normal Python. It trades speed for compatibility: your script behaves exactly as it does under `python3`.

---

## Why compyle?

Tools like PyInstaller copy a whole Python interpreter and your libraries into every executable, so even a small script typically becomes several megabytes, and far more with Qt or Pygame. `compyle` uses the Python that is already installed instead.

* **Tiny output.** A small script becomes a launcher of roughly 17 KB (measured on Linux with Python 3.13). The size grows only with your script's bytecode.
* **No bundled interpreter.** Nothing is duplicated, and the system's Python and libraries are used as they are.
* **Full compatibility.** Your script runs on real CPython, so Pygame and PyQt6 behave exactly as they do under `python3`.
* **One small file.** The whole tool is `compyle.js`, with no npm packages.

**The tradeoff:** the binary is tied to the Python it was built against, so a machine with a different Python version may not run it. PyInstaller avoids that by bundling everything, at the cost of size. Choose `compyle` when you want small, and PyInstaller when you need one file that runs anywhere.

---

## Requirements

* **[Bun](https://bun.sh)** to run `compyle.js`.
* **Python 3 with its development files** (headers and `libpython`). The built binary links against `libpython` and needs Python installed to run.
* **A C compiler:** `gcc`, `clang` or `cc` on Linux and macOS; `gcc`, `clang` or `cl` on Windows.

## Usage

```bash
bun compyle.js main.py -o main
./main
```

| Flag | Meaning |
|------|---------|
| `-o <name>` | Name of the output binary (default `a.out`, or `a.exe` on Windows) |
| `--keep-c` | Keep the generated `.c` file |
| `-v`, `--verbose` | Print the detected Python and the compiler command |
| `-h`, `--help` | Show usage |

### Optional: make `compyle` itself a single executable

```bash
bun build --compile compyle.js --outfile compyle
./compyle main.py -o main
```

This bundles the Bun runtime, so the `compyle` binary itself is large (about 100 MB). The programs it produces are still tiny.

## How it works

1. Reads your `.py` file and lists the imports it finds (for information only).
2. Compiles the script to bytecode using Python's own `compile()`.
3. Writes a C file that holds the bytecode and boots CPython.
4. Asks Python where its headers and library live (`sysconfig`), then calls your C compiler to build the executable.

## Platform support

| Platform | Status |
|----------|--------|
| Linux | Tested (Arch Linux; GCC) |
| macOS | Implemented, not yet tested |
| Windows | Implemented for MSVC (`cl`) and MinGW (`gcc`/`clang`), not yet tested |

On Windows, the built program also needs `python3XX.dll` to be found when it runs, so Python should be installed.

## Limitations

* The binary needs the same Python installed on the machine that runs it.
* Imported local `.py` files are **not** bundled. They are loaded from the current directory at runtime.
* Speed is the same as running the script with `python3`.
* The import list is only printed. Nothing uses it yet.

## License

GNU General Public License v3.0 (GPL-3.0). See `LICENSE`.
