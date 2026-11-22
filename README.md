# ramal

A hardened, dependency-free deployment engine in Rust, inspired by Kamal.

`ramal` is designed to simulate the full deployment lifecycle of multi-service applications and accessories without requiring Docker, a network connection, or external crates. It targets a local simulated cluster, executing everything via the local filesystem and isolated subprocesses. 

## Why Ramal?

Deployment tools are often complex, heavily reliant on external ecosystems, and assume a standard environment (Docker, network, remote SSH). `ramal` strips this down to the core mechanics of deployment:

*   **Zero External Dependencies:** The YAML parser, process runner, secrets manager, and HTTP/registry mock are all written from scratch using only the Rust standard library.
*   **Fully Offline:** It operates entirely on a simulated local cluster. No internet, no Docker daemon, and no real remote hosts are required.
*   **Hardened by Default:** The engine is built to survive hostile or corrupted inputs. The deployment path is continuously tested against injected faults (e.g., simulated host crashes, partial state corruption, malformed configs).
*   **Strict Secret Hygiene:** Guarantees that registry credentials and environment secrets never leak into process arguments, local files, or log output.

## Core Components

1.  **Config Compiler:** An in-repo YAML parser that validates schemas, detects cycles, resolves dependencies, and outputs a strictly deterministic, typed deployment plan.
2.  **Local Cluster Simulator:** Models multiple "hosts" using local directories and process namespaces. It executes the deployment plan by writing files, starting simulated processes, and managing lifecycle states without a real container runtime.
3.  **Secret & Auth Subsystem:** Interacts with an in-repo mock registry. Enforces that carriers are `0600`, cleans up temporary files, and guarantees output redaction.
4.  **Fault Injector & Harness:** A property-based testing suite that kills simulated hosts mid-deploy, corrupts partial states, and feeds oversized payloads to the mock registry to verify resilience.
5.  **CLI & Audit Layer:** A minimal argument parser and a deterministic, redacted audit log that records every action and its outcome for replay and inspection.

## Usage

*(Note: As this is a greenfield project, these commands are representative of the intended CLI surface.)*

Initialize a new simulated cluster and deploy a config:
```bash
cargo run -- simulate init --hosts 2
cargo run -- deploy --config kamal.yml
```

Inspect the redacted audit log:
```bash
cargo run -- audit show --last
```

Run the fault-injection conformance suite:
```bash
cargo run -- test fault-harness
```

## Development & AI Review

This project is built with an emphasis on strict review of AI-generated code. AI-written code is not executed until it passes:
1.  **Static analysis:** The in-repo linter (and `cargo clippy` if permitted in the environment).
2.  **Unit gates:** Targeted tests for the modified component.
3.  **Manual review:** A strict checklist applied to the hardened paths (credential handling, parser edge cases, and fault-injection hooks).
Only after these gates pass does the fault harness run the new code against the simulated cluster.
