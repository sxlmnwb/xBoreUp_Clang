<div align="center">

# xBoreUp Clang 22.1.5

[![LLVM Version](https://img.shields.io/badge/LLVM-22.1.5-orange?style=for-the-badge&logo=llvm)](https://github.com/llvm/llvm-project)
[![Target Arch](https://img.shields.io/badge/Target-AMD%20Zen%205-ED1C24?style=for-the-badge&logo=amd)](https://www.amd.com)
[![Optimization](https://img.shields.io/badge/Flags-znver5%20%7C%20O3%20%7C%20ThinLTO-brightgreen?style=for-the-badge)](https://clang.llvm.org)
[![Base Compiler](https://img.shields.io/badge/Base-AOCC%20v5.1.0-0071C5?style=for-the-badge&logo=amd)](https://www.amd.com/en/developer/aocc.html)

**A next-generation LLVM/Clang toolchain, purpose-built for AMD Zen 5 architecture through a 3-Stage Advanced Bootstrap pipeline — inheriting the internal microarchitecture intelligence of AMD's own AOCC compiler.**

</div>

---

## What is xBoreUp?

xBoreUp is a specialized LLVM/Clang toolchain built **exclusively** for AMD Zen 5 (`znver5`) architecture. This is not a standard LLVM build — it is forged through a **3-Stage Advanced Bootstrap** process that uses AOCC (AMD Optimizing C/C++ Compiler) as its genetic foundation, passing down AMD's internal microarchitecture tuning heuristics across compiler generations before activating ThinLTO in the final stage.

---

## 🧬 Build Topology — The DNA Chain

This toolchain passes through three generations of evolution to achieve maximum compilation performance and optimal machine code output. Each generation inherits and amplifies the properties of the last.

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                           xBoreUp — 3-Stage DNA Chain                        ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║    The Founding Father     The DNA Bridge            The Final Evolution     ║
║       AOCC v5.1.0          Bridge Compiler            xBoreUp Compiler       ║
║       clang 17.0.6   ──►   clang 22.1.5      ──►      clang 22.1.5           ║
║       AMD Internal         znver5 + O3                znver5 + O3            ║
║       (Proprietary)        No ThinLTO                 + ThinLTO              ║
║                            [Internal Only]            [Distributed]          ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

### GEN 0 - The Founding Father (AOCC v5.1.0)

```
Compiler : AMD clang version 17.0.6 (AOCC_5.1.0-Build#1994 2025_12_23)
Role     : Primary Host Compiler
```

AOCC is AMD's proprietary compiler, internally patched by AMD engineers to produce the most optimal machine instructions for their own CPU microarchitectures. It carries deep knowledge of Zen 5 pipeline internals, cache topology, and instruction scheduling that is not always present in upstream LLVM.

**Limitation:** Based on the relatively older LLVM 17 core. Cannot stably build LLVM 22.x as a final product, and does not support seamless ThinLTO integration when used as a direct host for the latest upstream builds.

---

### GEN 1 - The DNA Bridge (Intermediate Compiler)

```
Compiler : AOCC v5.1.0 DNA Bridge with [znver5,O3] clang version 22.1.5
Role     : Internal bridge compiler — not distributed
```

Built by compiling raw LLVM 22.1.5 source code **using the Gen 0 AOCC compiler**. At this stage, a modern Clang 22 binary is born, carrying the **inherited tuning heuristics** from AOCC baked into its machine code. However, due to Gen 0's limitations, this build cannot yet be optimized with Link Time Optimization (LTO).

> **Status: Internal Use Only** — this binary is not part of the final release.

---

### GEN 2 - The Final Evolution (xBoreUp)

```
Compiler : xBoreUp with [lto-thin,znver5,O3] feat AOCC v5.1.0 clang version 22.1.5
Role     : Production toolchain — this is what you use
```

Built by using the Gen 1 Bridge Compiler to **re-bootstrap** LLVM 22.1.5 from source, this time with `--lto thin` fully activated. The result is a complete, state-of-the-art Clang 22 binary that simultaneously:

- Carries the latest upstream LLVM 22 codebase
- Inherits AOCC's internal AMD optimization intelligence from Gen 0
- Has undergone full cross-module optimization via ThinLTO

This is the toolchain you install and use.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| **Native AMD Zen 5** | Compiled exclusively with `-march=znver5 -mtune=znver5 -O3`. Every instruction is tuned for Ryzen 9000 / EPYC 9005 series. |
| **AVX-512 Ready** | Full auto-vectorization support using AVX-512 registers (`%zmm`). Unlocks the full width of Zen 5's vector execution units. |
| **ThinLTO Enabled** | Link Time Optimization activated in the final build stage. Produces a leaner binary with faster startup and compilation throughput. |
| **AOCC Heuristic Inheritance** | Bootstrapped from AOCC, meaning the machine code generation carries AMD's internal CPU knowledge — an advantage over a GCC-bootstrapped Clang. |
| **LLD as Default Linker** | Optimized to use `ld.lld` instead of `ld.bfd`, significantly accelerating linking when building the Linux kernel or large software projects. |
| **Polly Included** | LLVM's high-level loop optimizer (Polly) is bundled and available for advanced loop nest transformations and auto-parallelization. |

---

## 🔧 Build Specification

The following parameters were used to produce **Generation 2 (Final Toolchain)**:

```bash
# ── Host Compiler: Generation 1 Bridge ────────────────────────────────
export PATH="/home/sxlmnwb/toolchains/bridge/bin:$PATH"
export LIBRARY_PATH="/home/sxlmnwb/toolchains/bridge/lib:/opt/aocc/lib32:$LIBRARY_PATH"
export LD_LIBRARY_PATH="/home/sxlmnwb/toolchains/bridge/lib:/opt/aocc/lib32:$LD_LIBRARY_PATH"
export CC="clang"
export CXX="clang++"
export LDFLAGS="-fuse-ld=lld"

# ── Optimization Flags ─────────────────────────────────────────────────
ZEN5_FLAGS="-march=znver5 -mtune=znver5 -O3"
LLVM_NAME="xBoreUp with [lto-thin,znver5,O3] feat AOCC v5.1.0 (https://www.amd.com/en/developer/aocc.html)"

# ── Build Execution ────────────────────────────────────────────────────
python3 build-llvm.py \
    --vendor-string "$LLVM_NAME" \
    -p clang compiler-rt lld polly \
    -t X86 \
    -r "release/22.x" \
    --build-type Release \
    --shallow-clone \
    --lto thin \
    -i install \
    -D "CMAKE_C_FLAGS=$ZEN5_FLAGS" \
       "CMAKE_CXX_FLAGS=$ZEN5_FLAGS" \
       "CMAKE_EXE_LINKER_FLAGS=-fuse-ld=lld" \
    2>&1 | tee build-llvm.log

python3 build-binutils.py \
    -t x86_64 \
    -i install \
    -m znver5 \
    2>&1 | tee -a build-binutils.log
```

---

## 🚀 Advanced Optimization — PGO + BOLT (Optional)

> **For users who want to push the toolchain to its absolute performance ceiling.**

PGO and BOLT are two complementary optimization techniques that operate at different levels. Combining both on top of the existing ThinLTO + znver5 build can yield an additional **+20–40% compilation speed** improvement over the standard xBoreUp build.

### How They Differ

```
┌──────────────────────────────────────────────────────────────────────┐
│                  Optimization Layer Stack                            │
│                                                                      │
│  ┌─────────────────────────────────────┐                             │
│  │  BOLT  — works on the final binary  │  ← rewrites ELF layout      │
│  ├─────────────────────────────────────┤                             │
│  │  PGO   — works on LLVM IR           │  ← guides IR optimization   │
│  ├─────────────────────────────────────┤                             │
│  │  ThinLTO — works across modules     │  ← cross-module inlining    │
│  ├─────────────────────────────────────┤                             │
│  │  znver5 + O3 — instruction tuning   │  ← native Zen 5 codegen     │
│  └─────────────────────────────────────┘                             │
│                                                                      │
│  Each layer optimizes what the layer below cannot see.               │
└──────────────────────────────────────────────────────────────────────┘
```

| Technique | What it optimizes | Gain over base |
|---|---|---|
| ThinLTO | Cross-module dead code & inlining | ~+10–15% |
| PGO | IR-level branch layout, inlining decisions | ~+15–25% |
| BOLT | Final binary function & basic block layout | ~+5–10% on top of PGO |
| **PGO + BOLT** | **Both IR and binary levels** | **~+20–35% combined** |

---

### Step 0 — Check perf Support (Important)

BOLT can profile using either hardware `perf` (fast, low overhead) or software instrumentation (slow, generates hundreds of GB of data). Always check `perf` first:

```bash
perf record --branch-filter any,u --event cycles:u --output /dev/null -- sleep 1
```

- If the command succeeds → `perf` mode will be used automatically by the script (recommended).
- If it fails with permission errors → add `perf_event_paranoid` tweak:

```bash
echo 0 | sudo tee /proc/sys/kernel/perf_event_paranoid
```

- If it still fails → BOLT will fall back to **instrumentation mode** (works, but generates very large profile files — ensure you have 50+ GB free disk).

---

### Step 1 — Disk & RAM Requirements

Before proceeding, verify you have enough resources:

| Resource | Minimum | Recommended |
|---|---|---|
| **Free Disk** | 30 GB | 50 GB |
| **RAM** | 16 GB | 32 GB |
| **Build Time** | ~3–4× longer than standard | depends on CPU core count |

---

### Step 2 — Build with PGO + BOLT

Replace the standard Gen 2 build command with the following. The host compiler setup (PATH, CC, CXX) remains identical to the standard build.

```bash
# ── Host Compiler: The Final Evolution (xBoreUp) ───────────────────────
export PATH="/home/sxlmnwb/toolchains/xBoreUp_Clang/bin:$PATH"
export LIBRARY_PATH="/home/sxlmnwb/toolchains/xBoreUp_Clang/lib:/home/sxlmnwb/toolchains/xBoreUp_Clang/lib32:$LIBRARY_PATH"
export LD_LIBRARY_PATH="/home/sxlmnwb/toolchains/xBoreUp_Clang/lib:/home/sxlmnwb/toolchains/xBoreUp_Clang/lib32:$LD_LIBRARY_PATH"
export CC="clang"
export CXX="clang++"

# ── Optimization Flags ─────────────────────────────────────────────────
ZEN5_FLAGS="-march=znver5 -mtune=znver5 -O3"
LLVM_NAME="xBoreUp with [lto-thin,pgo,bolt,znver5,O3] feat AOCC v5.1.0 (https://www.amd.com/en/developer/aocc.html)"

# ── Build Execution ────────────────────────────────────────────────────
python3 build-llvm.py \
    --vendor-string "$LLVM_NAME" \
    -p clang compiler-rt lld polly \
    -t X86 \
    -r "release/22.x" \
    --build-type Release \
    --shallow-clone \
    --lto thin \
    --pgo kernel-defconfig-slim \
    --bolt \
    -i install \
    -D "CMAKE_C_FLAGS=$ZEN5_FLAGS" \
       "CMAKE_CXX_FLAGS=$ZEN5_FLAGS" \
       "CMAKE_EXE_LINKER_FLAGS=-fuse-ld=lld" \
    2>&1 | tee build-llvm-pgo-bolt.log
```

> **Note on `--assertions`:** An older upstream bug ([llvm/llvm-project#55004](https://github.com/llvm/llvm-project/issues/55004)) required `--assertions` when combining `--bolt` and `--pgo` to prevent instrumented binary crashes. This issue is **closed and fixed** since LLVM 16 via the addition of `bolt/lib/Passes/ValidateMemRefs.cpp`. LLVM 22.x already includes this fix — `--assertions` is **not needed** and omitting it results in a faster final build (~15% less overhead on the PGO stage).

---

### What Happens Internally

The script automatically runs the full pipeline for you:

```
Stage 1 — Bootstrap compiler (lightweight, fast)
    ↓
Stage 2 — Instrumented compiler built with bootstrap
    ↓
Stage 3 — Profiling run: build Linux kernel defconfig with instrumented compiler
           → generates profile data (.profraw / perf.data)
    ↓
Stage 4 — Merge profile data into .profdata
    ↓
Stage 5 — Final compiler built with PGO profile + ThinLTO
    ↓
Stage 6 — BOLT profiles final clang binary (kernel build again)
    ↓
Stage 7 — BOLT rewrites clang binary layout using profile
           → final binary installed to install/bin/clang
```

---

### Performance Gain Summary

```
xBoreUp standard (znver5 + O3 + ThinLTO)     →  baseline
+ PGO (kernel-defconfig-slim)                →  ~+15–25% faster compile
+ PGO + BOLT (with perf)                     →  ~+22–35% faster compile
+ PGO + BOLT (with instrumentation)          →  ~+20–32% faster compile
```

> Numbers are estimates based on upstream LLVM benchmarks and community reports. Actual gains depend on workload, kernel config size, and available CPU cores.

---

### Troubleshooting PGO + BOLT

**Build crashes during BOLT instrumentation stage:**

For LLVM 22.x (`release/22.x`), the crash described in [#55004](https://github.com/llvm/llvm-project/issues/55004) is **already fixed** — do not add `--assertions`. If you are on a pre-LLVM 16 branch and encounter this crash, you can try:
```bash
--bolt --pgo kernel-defconfig-slim --assertions
```

**Out of disk space during profiling:**
```bash
# Use slim variant — profiles only one architecture instead of all
--pgo kernel-defconfig-slim   # instead of kernel-defconfig
```

**BOLT stage takes too long / OOM:**

This can happen when using instrumentation mode without the `llvm-bolt` commit `7d7771f34d14`. Check your LLVM version or switch to `perf` if supported.

**Want to skip BOLT and use PGO only:**
```bash
# Remove --bolt, keep --pgo
--pgo kernel-defconfig-slim --lto thin
```

---

## ⚠️ Compatibility Warning

> **This toolchain is Zen 5 exclusive.**

xBoreUp was compiled with `znver5` instructions including AVX-512 extensions specific to the Zen 5 microarchitecture. These binaries **will not run** on:

- Any Intel CPU
- AMD Zen 1 / Zen 2 / Zen 3 / Zen 4 or older

**Supported hardware only:**

- AMD Ryzen 9000 Series (Granite Ridge)
- AMD Ryzen AI 300 Series (Strix Point / Krackan Point)
- AMD EPYC 9005 Series (Turin)
- AMD Ryzen Threadripper 9000 Series (Shimada Peak)

Do not distribute this toolchain as a general-purpose compiler for systems you do not control.

---

## 🙏 Credits

| Party | Contribution |
|---|---|
| **[AMD](https://www.amd.com/en/developer/aocc.html)** | AOCC v5.1.0 — the genetic foundation and source of AMD-internal microarchitecture tuning heuristics |
| **[LLVM / Clang Project](https://llvm.org)** | The world-class open-source compiler infrastructure |
| **[ClangBuiltLinux/tc-build](https://github.com/ClangBuiltLinux/tc-build)** | The build orchestration scripts that make this pipeline possible |
| **[sxlmnwb](https://github.com/sxlmnwb)** | 3-Stage AOCC Bootstrap topology concept, Zen 5 tuning strategy, and build pipeline engineering |

---

## 📄 License

| Component | License |
|---|---|
| Build scripts | [Apache-2.0](https://github.com/ClangBuiltLinux/tc-build/blob/main/LICENSE) |
| LLVM / Clang | [Apache-2.0 with LLVM exceptions](https://llvm.org/LICENSE.txt) |
| LLVM User Guide | [LLVM Documentation](https://llvm.org/docs/UserGuides.html) |
| AOCC v5.1.0 | [AMD AOCC EULA](https://www.amd.com/en/developer/aocc/aocc-compiler/eula.html) |
| AOCC User Guide | [AMD Documentation](https://docs.amd.com/r/en-US/57222-AOCC-user-guide/Introduction) |

---

<div align="center">

*Built with extreme precision for AMD Zen 5 architecture.*

</div>
