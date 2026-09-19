# qstack guide

Short path from install to a dual-reviewed React PR.

## 1. Prerequisites

- Cursor with plugins enabled
- [pstack](https://github.com/cursor/plugins/tree/main/pstack) installed (`/add-plugin pstack`)
- qstack installed from `https://github.com/QuarkOS/Qstack`
- `gh` authenticated for the target React fork or checkout

## 2. First `/react` run

Open the React checkout (or your fork). Invoke:

```
/react <what you want changed>
```

The agent should:

1. Load `/poteto-mode` and match a playbook
2. Ensure a project `verify-*` skill exists
3. Build the smallest correct change
4. Swarm and verify
5. Open a ready PR
6. Dual-review with `/lauren-mode` and `/eps1lon-mode`
7. Fix surviving issues
8. Stop at a ready PR unless you ask to land

## 3. When to call modes directly

Use `/lauren-mode` alone for compiler or rust-compiler review style.

Use `/eps1lon-mode` alone for Fiber, DOM, Flight, test infra, or CI review style.

Use `/react` when you want the full build → verify → dual-review → publish loop.

## 4. Failure modes

| symptom | fix |
|---|---|
| agent invents lauren/eps1lon rules | skills missing from the plugin install |
| `/react` cannot find poteto-mode | install pstack |
| dual review only nits | triage; only correctness and security block publish |
| wants to merge react/react | refuse until you explicitly ask to land |
