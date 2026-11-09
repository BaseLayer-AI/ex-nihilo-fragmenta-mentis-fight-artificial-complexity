# Fight artificial complexity in Nix ecosystem 

Part of the [Ex Nihilo: Fragmenta Mentis (Out of Nothing: Fragments of the Mind)](https://flakehub.com/flake/BaseLayer-AI/ex-nihilo-fragmenta-mentis-000?view=readme) series.

This is a powerful stance. The "Nix idiomatic" way often prioritizes **DRY (Don't Repeat Yourself)** and **Functional Elegance**, but for **Agentic Coding**, those are often secondary to **Observability** and **Linearity**.

In the Nix ecosystem, "Idiomatic" usually means "highly abstracted." This creates a "hidden state" problem where an AI agent (or a human) has to simulate the Nix preprocessor and the internal logic of helper functions (like `cmakeWrap` or `python3Wrap`) just to understand what a single line of code is doing.

Here is the manifesto for **Thin Stack Nix**—the antidote to artificial complexity.

---

### The Thin Stack Manifesto

#### 1. Replace Wrappers with Naked Derivations
*   **The Complexity:** Using `pkgs.cmakeWrap`, `pkgs.python3Wrap`, or `pkgs.makeWrapper`. These are "black boxes" that hide the shell commands.
*   **The Thin Stack:** Use `pkgs.stdenv.mkDerivation`. Write the `buildPhase` and `installPhase` yourself.
*   **Why:** If a build fails, the error message points to a specific line in your shell script. If it fails inside a wrapper, the agent has to "guess" which part of the wrapper's internal logic failed.

#### 2. Eliminate Preprocessor Sugar
*   **The Complexity:** Using `inherit`, `args`, or complex `lib` function pipelines.
*   **The Thin Stack:** Use explicit assignments: `src = src;`.
*   **Why:** `inherit` is a preprocessor trick. For an agent, every line of code should represent a literal operation. Explicit assignment ensures the "Data Flow" is visible in the text, not hidden in the pre-evaluation phase.

#### 3. Raw Strings over List Transformations
*   **The Complexity:** Defining a list of flags `[ "-DFlag1", "-DFlag2" ]` and having Nix join them into a string.
*   **The Thin Stack:** Define the full string: `cmakeArgs = "-DFlag1 -DFlag2";`.
*   **Why:** It removes one layer of "transformation." The agent doesn't have to reason about how a list becomes a string; it sees the final command that will be executed.

#### 4. Linear "Extraction $\rightarrow$ Mapping"
*   **The Complexity:** Deeply nested `let` blocks where variables are defined inside other variables.
*   **The Thin Stack:** A flat structure. 
    *   **Phase 1 (Extraction):** Fetch the raw assets (Source, Hash, Deps).
    *   **Phase 2 (Mapping):** Define the build parameters (Strings, Paths).
    *   **Phase 3 (The Derivation):** The final construction.
*   **Why:** This limits the "Mental Stack." The agent can look at any variable and find its definition in a single glance without traversing a tree.

#### 5. No "Clever" Nix Functions
*   **The Complexity:** Using `lib.concatStringsSep`, `lib.path`, or complex `map` operations to construct paths.
*   **The Thin Stack:** Use standard shell interpolation inside the `buildPhase` (e.g., `$out`, `../tools`).
*   **Why:** Nix is often used to generate shell scripts. Let the shell do the work of path manipulation. Shell is the "ground truth" of the build system.

---

### Comparison: Complexity vs. Thin Stack

**The "Idiomatic" Complexity (Hard for Agents to reason about):**
```nix
# Hidden logic inside cmakeWrap
# Preprocessor sugar with inherit
# List manipulation to join flags
pkgs.cmakeWrap {
  pkgName = "llama-cpp";
  src = src;
  defaultTools = [ "cmake" "ninja" ];
  cmakeFlags = [ "-DGGML_METAL=ON" "-DGGML_OPENSSL=ON" ]; 
  # The agent has to know how cmakeWrap joins these flags and interacts with the shell
}
```

**The Thin Stack (Optimized for Agents):**
```nix
# Naked derivation
# Explicit assignments
# Raw string for flags
pkgs.stdenv.mkDerivation {
  pname = "llama-cpp";
  src = src;
  CMAKE_ARGS = "-DGGML_METAL=ON -DGGML_OPENSSL=ON"; 
  buildPhase = ''
    mkdir build && cd build
    cmake .. -DCMAKE_BUILD_TYPE=Release ${CMAKE_ARGS}
    ninja
  '';
  # The agent sees the EXACT command that runs. No mystery.
}
```

### The Goal
**The goal of Nix is to produce a reproducible build. The "Thin Stack" ensures that the path from "Source Code" to "Reproducible Build" is as short and visible as possible.** 

By fighting artificial complexity, you make the system **debuggable by machines**, which is the ultimate requirement for agentic workflows.

https://nix-ontouchstart.github.io/nix-ontouchstart-llama_cpp-sessions/2026-11-09_01-02-53_conv_7d89e49b_fight_artificial_complexity
