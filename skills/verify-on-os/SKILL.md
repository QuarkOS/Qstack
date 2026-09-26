---
name: verify-on-os
description: >-
  Use this when a cloud agent must prove code runs on a real Ubuntu, Fedora, or
  Windows 11 machine: SSH through Cloudflare Access to
  verify.emilioschwaiger.com, get a fresh throwaway VM, run, and report the
  output.
disable-model-invocation: true
---

# Verify on a real OS (throwaway VM)

`verify.emilioschwaiger.com` is an SSH gateway on the homelab. Each login boots a fresh linked clone of an OS template. The clone is destroyed when you disconnect, and at most 2 run at once. The login user picks the OS:

| User | OS |
|---|---|
| `ubuntu@` | Ubuntu 24.04 |
| `fedora@` | Fedora 44 |
| `windows@` | Windows 11 Enterprise Eval (OpenSSH, default shell cmd) |

Two credentials are required. Cloudflare Access only lets the service token through, and the gateway only accepts the key.

## Needs (Cursor runtime secrets)

- `CLOUD_VERIFY_PRIVATE_KEY`: an ed25519 OpenSSH private key, no passphrase.
- `CF_ACCESS_CLIENT_ID` / `CF_ACCESS_CLIENT_SECRET`: the Access service token `verify-agents`.

If any is missing, stop and report which env name is absent. Never print the values.

## Steps

1. Install cloudflared:
   ```bash
   curl -fsSL -o /tmp/cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 && chmod +x /tmp/cloudflared
   ```
2. Write the key. The secret may arrive as one line with spaces where the newlines belong, so restore the newlines:
   ```bash
   mkdir -p ~/.ssh
   python3 - <<'EOF'
   import os,re
   k=os.environ["CLOUD_VERIFY_PRIVATE_KEY"].strip()
   if "\n" not in k:
       m=re.match(r"(-----BEGIN [A-Z ]+-----)\s*(.*?)\s*(-----END [A-Z ]+-----)",k)
       k=m.group(1)+"\n"+"\n".join(m.group(2).split())+"\n"+m.group(3)
   p=os.path.expanduser("~/.ssh/verify_key"); open(p,"w").write(k+"\n"); os.chmod(p,0o600)
   EOF
   ssh-keygen -y -P '' -f ~/.ssh/verify_key >/dev/null && echo key-ok
   ```
3. Run:
   ```bash
   vssh() { os=$1; shift; ssh -i ~/.ssh/verify_key -o IdentitiesOnly=yes -o BatchMode=yes \
     -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ConnectTimeout=150 \
     -o ProxyCommand="/tmp/cloudflared access ssh --hostname %h --loglevel error --service-token-id $CF_ACCESS_CLIENT_ID --service-token-secret $CF_ACCESS_CLIENT_SECRET" \
     "$os@verify.emilioschwaiger.com" "$@"; }
   vssh ubuntu 'grep ^PRETTY_NAME= /etc/os-release'
   vssh fedora 'grep ^PRETTY_NAME= /etc/os-release'
   vssh windows ver
   ```
   Windows boots in about 60s. On Windows, pass plain cmd commands (`ver`, `where node`); `cmd /c ...` gets mis-quoted.
4. Copy files in with `scp` using the same options, or pipe a tarball: `tar c . | vssh ubuntu 'mkdir w && tar x -C w && cd w && ./test.sh'`. Every call is a fresh VM, so put the whole job in one session.

## Report

Show the exact command, the exit code, the output lines that prove it, and the `[verify-gw] clone NNNN` line. The `[verify-gw] clone` line goes to stderr, so capture it with `2>&1`. Ignore the known-hosts warning and the remote `setlocale` warning when the clone line is present. `ver` prints a blank line first. A connection failure is BLOCKED, not PASS: `websocket: bad handshake` means the service token is wrong or missing, and `Permission denied (publickey)` means the key is wrong.
