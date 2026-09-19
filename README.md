# qstack

React-focused Cursor skills for QuarkOS. Pair with [pstack](https://github.com/cursor/plugins/tree/main/pstack) for playbooks, then use qstack for React core conventions and a dual-review publish pipeline.

**qstack is not a fork of pstack.** It sits beside it. `/react` calls pstack's `/poteto-mode`, `/swarm`, and `/create-verification-skill`, then reviews with lauren and eps1lon habits mined from public `react/react` work.

## install

Install **pstack** first, then this plugin from the repo.

```bash
/add-plugin pstack
```

Point Cursor at this repository as a local or GitHub plugin source, or clone and add the plugin path for `QuarkOS/Qstack`.

## get started

1. Install pstack and run `/setup-pstack` if you have not already.
2. Install qstack.
3. For React work, start with `/react`.

```
/react fix Fragment blur inside ShadowRoot. repro on main first, then dual-review before ready.
```

```
/lauren-mode review this compiler PR the way poteto would.
```

```
/eps1lon-mode review this Fiber PR the way sebbie would.
```

## skills

Layout matches pstack. Each skill lives under `skills/<name>/SKILL.md`.

| skill | use it when |
|---|---|
| [`/react`](./skills/react/SKILL.md) | end-to-end React change. poteto-mode build, verify, swarm, open PR, lauren+eps1lon dual review, fix, publish ready. |
| [`/lauren-mode`](./skills/lauren-mode/SKILL.md) | match Lauren (GitHub poteto) React Compiler and rust-compiler habits. |
| [`/eps1lon-mode`](./skills/eps1lon-mode/SKILL.md) | match Sebastian Silbermann (eps1lon) Fiber, DOM, Flight, test, and CI habits. |

### `/react` pipeline

1. Enter poteto-mode
2. Ensure verification skill
3. Build the change
4. Swarm verify
5. Open the PR
6. Dual review (`lauren-mode` + `eps1lon-mode`)
7. Fix review issues
8. Publish (ready PR; land only on explicit ask)

### agents

| agent | role |
|---|---|
| [`react-agent`](./agents/react-agent.md) | routing target for `/react` dual-review workers |

## depends on

| from pstack | why |
|---|---|
| `/poteto-mode` | playbooks, opening-a-pr, shipping |
| `/swarm` | parallel verify slices |
| `/create-verification-skill` | project-local `verify-<app>` |

## layout

```
Qstack/
  .cursor-plugin/plugin.json
  agents/
    react-agent.md
  skills/
    react/SKILL.md
    lauren-mode/SKILL.md
    eps1lon-mode/SKILL.md
  docs/
    guide/README.md
  README.md
  LICENSE
```

## provenance

`lauren-mode` and `eps1lon-mode` were mined from public GitHub activity on `react/react`. They are engineering habit skills, not biographies, and they are not affiliated with Meta or the React team beyond what the public trail shows.

## license

MIT. See [LICENSE](./LICENSE).
