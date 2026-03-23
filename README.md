# DixonRes: Dixon Resultant & Polynomial System Solver
A C implementation for computing Dixon resultants and solving polynomial systems over finite fields and the rationals ℚ, based on the FLINT and PML libraries.

Website: <https://dixonres.github.io>

**Note:** This repository is under active development. The version submitted to Crypto 2026 is archived and available at:  
 - Release page: <https://github.com/DixonRes/DixonRes/releases/tag/0.0.1>
 - Code tree: <https://github.com/DixonRes/DixonRes/tree/0.0.1>

## Features
- Dixon resultant computation for variable elimination
- Polynomial system solver for n×n systems
- Dixon with triangular ideal reduction
- Finite fields:
  - Prime fields F_p (any size): Implemented with FLINT modular arithmetic, optionally accelerated by PML.
  - Extension fields F_{p^k}: Further optimized for binary fields F_{2^n} with n in {8, 16, 32, 64, 128}.
- Rational field ℚ: Rational reconstruction via multi-prime CRT. Set field_size = 0 to enable.
- Complexity analysis — estimates Dixon matrix size, Bezout degree bound, and operation count before computing
- Command line input or file input. Automatic output to solution files
- SageMath interface — call DixonRes directly from a SageMath session via `dixon_sage_interface.sage`

---

## Dependencies
- **FLINT** (recommended version: 3.4.0)  
  <https://github.com/flintlib/flint>

Optional:
- **PML** (used automatically if available)  
  <https://github.com/vneiger/pml>

---

## Build
```bash
git clone https://github.com/DixonRes/DixonRes.git && cd DixonRes
./configure
make
make check                         # optional
make install                       # optional
```
For more options, run `./configure --help` or `make help`.
We also provide a Windows GUI at [DixonRes/DixonRes-Windows](https://github.com/DixonRes/DixonRes-Windows).

---

## Usage

### Dixon Resultant (Basic)
```bash
./dixon "polynomials" "eliminate_vars" field_size
```
Examples:
```bash
./dixon "x+y+z, x*y+y*z+z*x, x*y*z+1" "x,y" 257
./dixon "x^2+y^2+z^2-1, x^2+y^2-2*z^2, x+y+z" "x,y" 0
```

---

### Polynomial System Solver (n equations in n variables)
```bash
./dixon --solve "polynomials" field_size
```
Example:
```bash
./dixon --solve "x^2 + y^2 + z^2 - 6, x + y + z - 4, x*y*z - x - 1" 257
```

---

### Complexity Analysis
Estimates the difficulty of a Dixon resultant computation **without** performing it.
Reports equation count, variable count, degree sequence, Dixon matrix size
(via Hessenberg recurrence), Bezout degree bound, and complexity in bits.

```bash
./dixon --comp "polynomials" "eliminate_vars" field_size
./dixon -c     "polynomials" "eliminate_vars" field_size
```

Examples:
```bash
./dixon --comp "x^3+y^3+z^3, x^2*y+y^2*z+z^2*x, x+y+z-1" "x,y" 257
```

**Custom omega** — set the matrix-multiplication exponent used in the complexity formula
(default: 2.3):
```bash
./dixon --comp --omega 2.373 "polynomials" "eliminate_vars" field_size
./dixon -c -w 2.0            "polynomials" "eliminate_vars" field_size
```

File input uses the same format as Dixon mode:
```bash
./dixon --comp example.dat          # output: example_comp.dat
```

---

### Extension Fields
```bash
./dixon "x + y^2 + t, x*y + t*y + 1" "x" 2^8
```
The default settings use `t` as the extension field generator and FLINT's built-in field polynomial.
```bash
./dixon --solve "x^2 + t*y, x*y + t^2" "2^8: t^8 + t^4 + t^3 + t + 1"
```
(with AES custom polynomial for F_256)

---

### Dixon with Ideal Reduction
```bash
./dixon --ideal "ideal_generators" "polynomials" "eliminate_vars" field_size
```
Example:
```bash
./dixon --ideal "a2^3=2*a1+1, a3^3=a1*a2+3" "a1^2+a2^2+a3^2-10, a3^3-a1*a2-3" "a3" 257
```

### Field-equation reduction mode 
After each multiplication, reduces x^q -> x for every variable.
```bash
./dixon --field-equation "polynomials" "eliminate_vars" field_size
./dixon --field-eqution -r "[d1,d2,...,dn]" field_size
```
Example:
```bash
./dixon --field-eqution "x0*x2+x1, x0*x1*x2+x2+1, x1*x2+x0+1" "x0,x1" 2
./dixon --field-eqution -r [3]*5 2
```

---

### Silent Mode
```bash
./dixon --silent [--solve|--comp|-c] <arguments>
```
No console output is produced; the solution/report file is still generated.

---

## Random Mode

Generate random polynomial systems with specified degrees for testing and benchmarking.

### Basic Usage
```bash
./dixon --random "[d1,d2,...,dn]" field_size
./dixon -r       "[d]*n"          field_size
```

- `[d1,d2,...,dn]`: degree list (comma-separated) for n polynomials
- `[d]*n`: all n polynomials have same degree d
- `field_size`: field size (prime or extension); use `0` for Q

### Combine with Compute Flags
```bash
# Random + Dixon elimination
./dixon -r --solve "[d1,...,dn]" field_size

# Random + complexity analysis
./dixon -r --comp  "[d]*n" field_size
./dixon -r -c --omega 2.373 "[4]*5" 257   # custom omega

# Random + Dixon with ideal reduction
./dixon -r "[d1,d2,d3]" "ideal_generators" field_size
```

### Examples
```bash
# 3 polynomials (deg 3,3,2) in GF(257)
./dixon --random "[3,3,2]" 257

# 3 polynomials (deg 3,3,2) over Q
./dixon --random "[3,3,2]" 0

# Solve 3 quadratic system in GF(257)
./dixon -r --solve "[2]*3" 257

# Complexity analysis of 4 quartic polynomials
./dixon -r --comp --omega 2.373 "[4]*4" 257

# GF(2^8) with degrees 3 and 2
./dixon -r "[3,2]" 2^8
```

## File Input Format

### Dixon Mode / Complexity Mode (multiline)
```
Line 1 : field size (prime or p^k; use 0 for Q; generator defaults to 't')
Line 2+: polynomials (comma-separated, may span multiple lines)
Last   : variables to ELIMINATE (comma-separated)
         (#eliminate = #equations - 1)
```
Example:
```bash
./dixon       example.dat
./dixon --comp example.dat
```

### Polynomial Solver Mode (multiline)
```
Line 1 : field size
Line 2+: polynomials
         (n equations in n variables)
```

---

## SageMath Interface

`dixon_sage_interface.sage` is a thin wrapper that lets you call DixonRes directly from a SageMath session. Polynomials are passed as native Sage objects; the interface handles serialisation, subprocess invocation, and output parsing automatically.

### Setup

Load the interface once per session and set the path to the `dixon` binary:

```python
load("dixon_sage_interface.sage")
set_dixon_path("./dixon")   # set once; all subsequent calls use this path
```

`set_dixon_path()` stores a module-level default so you never need to pass `dixon_path=` explicitly. The current value can be retrieved with `get_dixon_path()`.

### Functions

#### `DixonResultant(F, elim_vars, ...)`
Compute the Dixon resultant, eliminating the specified variables.

```
DixonResultant(F, elim_vars, field_size=None, dixon_path=None,
               finput="/tmp/dixon_in.dat", debug=False, timeout=600)
```

- `F` — list of Sage polynomials and/or plain strings (strings are accepted so that the output of a previous call can be fed in directly for iterative elimination)
- `elim_vars` — variables to eliminate (Sage vars or strings)
- `field_size` — prime, prime power, `(p,k)` tuple, `GF(...)` object, or `0` for ℚ. Inferred automatically from `F[0].base_ring()` when `None`.
- `debug` — print detailed diagnostics including input/output file contents

Returns the resultant as a raw string, or `None` on failure.

```python
R.<x, y, z> = GF(257)[]
F = [x + y + z - 3, x*y + y*z + z*x - 3, x*y*z - 1]
res = DixonResultant(F, [x, y])
print(res)
```

#### `DixonSolve(F, ...)`
Solve a polynomial system and enumerate all solutions.

```
DixonSolve(F, field_size=None, dixon_path=None,
           finput="/tmp/dixon_solve_in.dat", debug=False, timeout=600)
```

Returns a `list of dict` (one `{var_name: value_str}` per solution), `[]` (no solutions), `"infinite"` (positive-dimensional system), or `None` on failure.

```python
sols = DixonSolve(F)
for s in sols:
    print(s)
```

#### `DixonComplexity(F, elim_vars, ...)`
Run complexity analysis without performing any polynomial arithmetic.

```
DixonComplexity(F, elim_vars, field_size=None, omega=None,
                dixon_path=None, finput="/tmp/dixon_comp_in.dat",
                debug=False, timeout=120)
```

Returns a dict with keys `complexity_log2`, `omega`, `bezout_bound`, `matrix_size`, `degrees`, or `None` on failure.

```python
info = DixonComplexity(F, [x, y])
print(info)
# {'complexity_log2': 7.93, 'omega': 2.3, 'bezout_bound': 8,
#  'matrix_size': 5, 'degrees': [1, 2, 3]}
```

#### `DixonIdeal(F, ideal_gens, elim_vars, ...)`
Dixon resultant with triangular ideal reduction.

```
DixonIdeal(F, ideal_gens, elim_vars, field_size=None,
           dixon_path=None, debug=False, timeout=600)
```

`ideal_gens` is a list of strings of the form `"a^3=2*b+1"`. Returns the resultant string or `None`.

### Field size argument

All functions accept `field_size` in any of these forms. When `None` and `F` contains Sage polynomials, the base ring is inferred automatically.

| Value | Meaning |
|---|---|
| `None` | Inferred from `F[0].base_ring()` |
| `257` or `"257"` | GF(257) |
| `(2, 8)` | GF(2^8) |
| `"2^8"` | GF(2^8) |
| `GF(257)` | GF(257) object passed directly |
| `0` | Rational field ℚ |

### Examples

**Basic resultant and solver**

```python
load("dixon_sage_interface.sage")
set_dixon_path("./dixon")

R.<x, y, z> = GF(257)[]
F = [x + y + z - 3, x*y + y*z + z*x - 3, x*y*z - 1]

res  = DixonResultant(F, [x, y])
sols = DixonSolve(F)
info = DixonComplexity(F, [x, y])

print(res)
print(sols)
print(info)
```

**Iterative / cascaded elimination**

The output of `DixonResultant` is a plain string and can be passed directly as an element of `F` in a subsequent call, enabling variable-by-variable elimination over large systems.

```python
load("dixon_sage_interface.sage")
set_dixon_path("./dixon")

R.<x, y, z> = GF(17)[]
f1 = x + y + z
f2 = x*y + y*z + z*x + 1
f3 = y*z - 1
f4 = z - 2

res1 = DixonResultant([f1, f2], [x])
res2 = DixonResultant([res1, f3], [y])
res3 = DixonResultant([res2, f4], [z])

print("res1 =", res1)
print("res2 =", res2)
print("res3 =", res3)
```

**Debug mode**

Pass `debug=True` to any function to print the input file, command line, and raw output:

```python
res = DixonResultant(F, [x, y], debug=True)
```

### Helper functions

| Function | Description |
|---|---|
| `set_dixon_path(path)` | Set the default path to the `dixon` binary |
| `get_dixon_path()` | Return the currently configured default path |
| `ToDixon(F, elim_vars, field_size, finput)` | Write a Dixon/complexity input file without running the binary |
| `ToDixonSolver(F, field_size, finput)` | Write a solver input file without running the binary |

---

## Output

| Mode | Command-line input | File input `example.dat` |
|---|---|---|
| Dixon / Solver | `solution_YYYYMMDD_HHMMSS.dat` | `example_solution.dat` |
| Complexity | `comp_YYYYMMDD_HHMMSS.dat` | `example_comp.dat` |

Each output file contains field information, input polynomials, computation time,
and the resultant, solutions, or complexity report.

### Complexity report contents
- Equation count, variable list, elimination variable list, remaining variables
- Degree sequence of input polynomials
- Bezout bound (product of degrees)
- Dixon matrix size (Hessenberg recurrence)
- Resultant degree estimate
- Complexity in log₂ bits (with the omega value used)

---

## Notes
- All computation modes generate a solution/report file by default
- Extension fields are slower than prime fields due to polynomial arithmetic
- The optional PML library only accelerates well-determined systems over prime fields
- Complexity analysis does not run any polynomial arithmetic; it parses only
- Over Q (`field_size=0`), `--solve`, `--ideal`, and `--field-equation` are not yet supported

---

## Feature Support by Field

| Feature | F_p (p<2^63) | F_p (p>2^63) | F_{p^k} (p<2^63) | Q |
|---|---|---|---|---|
| Dixon resultant | ✅ | ✅ | ✅ | ✅ |
| Complexity analysis (`--comp`) | ✅ | ✅ | ✅ | ✅ |
| Random mode (`-r`) | ✅ | ✅ | ✅ | ✅ |
| Polynomial solver (`--solve`) | ✅ | ❌ | ✅ | ❌ |
| Ideal reduction (`--ideal`) | ✅ | ❌ | ✅ | ❌ |
| Field-equation reduction | ✅ | ❌ | ✅ | ❌ |
| PML acceleration | ✅ | ❌ | ❌ | ❌ |

---

## License
DixonRes is distributed under the GNU General Public License version 2.0 (GPL-2.0-or-later). See the file COPYING.
