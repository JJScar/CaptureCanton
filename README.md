# CaptureCanton

CaptureCanton is a collection of "capture the funds" exercises for devs and security researchers in the Canton ecosystem — think [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/), but for Daml/Canton.

Canton's model is different from a public EVM chain: workflows are private, authorization is explicit (`signatory` / `observer` / `controller`), and state transitions are enforced by the ledger rather than by contract bytecode alone. So instead of reentrancy and flash loans, these challenges are about **finding the gap in a contract's authorization logic or data model** — a party that can act when it shouldn't, a check that's missing, a state that can drift out of sync with reality.

Each challenge gives you a deliberately flawed Daml contract and a role to exploit it from. Your job: steal the funds anyway.

> **⚠️ Not production code.** Every contract in this repo is intentionally broken and exists purely to be exploited for learning. Don't copy these patterns into real projects, and don't treat any contract here as a reference implementation of "correct" Daml — assume it has bugs, because it does.

---

## Setup

You need the **Daml SDK** installed locally.

```bash
# macOS / Linux
curl -sSL https://get.daml.com/ | sh

# then make sure ~/.daml/bin is on your PATH, and verify:
daml version
```

For Windows or other install options, see the [official Daml installation docs](https://docs.daml.com/getting-started/installation.html).

Each challenge pins its own SDK version in `daml.yaml` (currently `3.4.11`). You don't need to match it manually — the `daml` assistant will fetch and use the right version automatically the first time you build a project that requests it.

## Repo layout

```
CaptureCanton/
├── 1-TheBeginning/        Challenge 1
├── 2-VaultItIs/            Challenge 2
├── common/                 Shared Daml packages used by some challenges
└── README.md               You are here
```

Every challenge folder has the same shape:

```
N-ChallengeName/
├── README.md               The brief: your role, your goal
├── main/
│   └── daml/*.daml          The flawed contract — this is what you're attacking
├── test/
│   └── daml/*Test.daml      Where you write your exploit, as a Daml Script
└── multi-package.yaml        Wires main + test together for the IDE/build tooling
```

- `main/` is the "deployed" contract. Treat it as read-only — you're not meant to edit it, you're meant to find a way to abuse the choices/authorization it already exposes.
- `test/` is your workspace. Each challenge's test file has a clearly marked block:

  ```haskell
  -- ENTER SOLUTION BELOW --
  -> Enter HERE!
  -- ENTER SOLUTION ABOVE --
  ```

  Write your exploit *only* inside that block. Everything else in the test file is scaffolding (party setup, the initial contract state, the final assertion that checks whether you actually pulled it off) — leave it as-is.

## How to play a challenge

1. `cd` into the challenge folder, e.g. `cd 2-VaultItIs`, and read its `README.md` for the specific scenario and goal.
2. Open `main/daml/*.daml` and study the contract: who are the signatories/observers, what choices exist, who can `controller` each one, what does each choice actually check before acting?
3. `main` and `test` are separate Daml packages — `test` depends on a **built `.dar`** of `main` (see `test/daml.yaml`'s `data-dependencies`), so build `main` first:

   ```bash
   cd main
   daml build
   ```
4. Write your exploit inside the marked block in `test/daml/*Test.daml`, then run it:

   ```bash
   cd ../test
   daml test
   ```

   `daml test` runs the Daml Script and checks the final assertion — if it passes, the assertion's condition (e.g. you drained the vault) held true and you've solved the challenge.
5. Prefer an interactive view? Run `daml studio` from the challenge's `test/` folder to open VS Code with inline Script Views, so you can watch contract state change as your exploit executes step by step.

## Tips

- These bugs live in *authorization design*, not cryptography or reentrancy. Ask: does every `controller` actually deserve the power a choice gives them? Does an `ensure` or `assert` check what it looks like it checks? Can a party reach a choice they were only meant to observe?
- The contracts intentionally have no explanatory comments — the challenge is reading and reasoning about Daml authorization semantics yourself, not pattern-matching a comment. Using AI to skip that step means you skip the point of the exercise.
- If `daml test` fails with a missing-`.dar` error, you skipped step 3 above (or changed `main` and need to rebuild it).

Good luck!
