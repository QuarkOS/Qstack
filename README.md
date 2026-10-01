# qstack

Cursor skills for QuarkOS coding agents. Skills, a verification loop, and review modes. It sits beside [pstack](https://github.com/cursor/plugins/tree/main/pstack).

**qstack is not a fork of pstack.** pstack owns the playbooks (`/poteto-mode`, `/swarm`, `/create-verification-skill`). qstack ships skills that call that loop, plus OS verification and React review habits.

## install

Install **pstack** first, then this plugin from the repo.

```bash
/add-plugin pstack
```

Point Cursor at this repository as a local or GitHub plugin source, or clone and add the plugin path for `QuarkOS/Qstack`.

## get started

1. Install pstack and run `/setup-pstack` if you have not already.
2. Install qstack.
3. Invoke the skill that matches the job. Each skill is `skills/<name>/SKILL.md`.

```
/verify-on-os prove this on Ubuntu 24.04.
```

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

The plugin manifest points `skills` at `./skills/`. Layout matches pstack.

| skill | use it when |
|---|---|
| [`/verify-on-os`](./skills/verify-on-os/SKILL.md) | prove a change on a fresh Ubuntu 24.04, Fedora 44, Windows 11, or Android 12 VM. SSH through Cloudflare Access to verify.emilioschwaiger.com. |
| [`/react`](./skills/react/SKILL.md) | end-to-end React change: poteto-mode build, verify, swarm, open PR, lauren and eps1lon dual review, fix, publish ready. |
| [`/lauren-mode`](./skills/lauren-mode/SKILL.md) | match Lauren (GitHub poteto) React Compiler and rust-compiler habits. |
| [`/eps1lon-mode`](./skills/eps1lon-mode/SKILL.md) | match Sebastian Silbermann (eps1lon) Fiber, DOM, Flight, test, and CI habits. |

## verification

A verification skill has to exist before the loop is trusted. After a coding job, run a swarm of read-only verify agents against the project's `verify-<app>` skill (pstack `/swarm` and `/create-verification-skill`).

`/verify-on-os` is the homelab check: one SSH login, one fresh VM, destroyed on disconnect. `/react` runs the same verify-then-swarm loop, then dual-reviews.

## agents

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
    verify-on-os/SKILL.md
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
