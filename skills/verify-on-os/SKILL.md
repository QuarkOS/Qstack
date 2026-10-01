---
name: verify-on-os
description: >-
  Use this when a cloud agent must prove code runs on a real Ubuntu, Fedora,
  Windows 11, or Android 12 (adb) machine: SSH through Cloudflare Access to
  verify.emilioschwaiger.com, get a fresh throwaway VM, run, and report the
  output.
disable-model-invocation: true
---

# Verify on a real OS (throwaway VM)

`verify.emilioschwaiger.com` is an SSH gateway on the homelab. Each login boots a fresh linked clone of an OS template. The clone is destroyed when you disconnect. At most 3 run at once (at most 1 Windows and 1 Android); extra logins wait in a queue for up to 15 min and print `verify-gw: queued, position N ...` to stderr. The login user picks the OS:

| User | OS |
|---|---|
| `ubuntu@` | Ubuntu 24.04 |
| `fedora@` | Fedora 44 |
| `windows@` | Windows 11 Enterprise Eval (OpenSSH, default shell cmd) |
| `android@` | Android 12 x86_64 (redroid) inside a throwaway Ubuntu VM, with `adb` preconnected |

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
   vssh android 'adb shell getprop ro.build.version.release'
   ```
   Windows boots in about 60s. On Windows, pass plain cmd commands (`ver`, `where node`); `cmd /c ...` gets mis-quoted.
4. Copy files in with `scp` using the same options, or pipe a tarball: `tar c . | vssh ubuntu 'mkdir w && tar x -C w && cd w && ./test.sh'`. Every call is a fresh VM, so put the whole job in one session.

## Android (`android@`)

The session is a bash shell on an Ubuntu VM that runs one Android 12 device (redroid, x86_64 only, 720x1280, UTC clock, no Google Play). The login waits until Android has booted, so the first command can use `adb` right away. Login takes about 30 s. `ANDROID_SERIAL=localhost:6555` is already set, so use plain `adb ...`.

- The APK must contain x86_64 native code or no native code at all. ARM-only APKs fail with `INSTALL_FAILED_NO_MATCHING_ABIS`.
- Launch with `am start`, not `monkey`, because monkey exits 251 on this image. To find the activity, run `adb shell cmd package resolve-activity --brief <pkg> | tail -n1`.
- Every call is a fresh device, so send the APK and a script in one tarball. Write screenshots to files and tar them back on stdout. Send all other output to stderr (`>&2`) so stdout carries only the tar:
  ```bash
  cat > t.sh <<'EOF'
  set -e
  adb install app.apk >&2
  ACT=$(adb shell cmd package resolve-activity --brief com.example.app | tail -n1 | tr -d '\r')
  adb shell am start -W -n "$ACT" >&2
  sleep 2
  mkdir -p /tmp/out
  adb exec-out screencap -p > /tmp/out/launch.png
  adb shell input tap 360 640           # also: input text hello, input keyevent KEYCODE_BACK, input swipe x1 y1 x2 y2
  sleep 1
  adb exec-out screencap -p > /tmp/out/after-tap.png
  adb shell dumpsys activity activities | grep -m1 ' ResumedActivity' >&2
  tar c -C /tmp/out .
  EOF
  tar c t.sh app.apk | vssh android 'mkdir w && tar x -C w && cd w && bash t.sh' > out.tar 2> android.log
  echo "exit=$?"; tar xvf out.tar; cat android.log
  ```
  For one screenshot alone, run `... 'adb exec-out screencap -p' > shot.png`.
- Logcat: `adb logcat -d -t 200 >&2`. Screen coordinates are in 720x1280 pixels.

## Report

Show the exact command, the exit code, the output lines that prove it, and the `[verify-gw] clone NNNN` line. The `[verify-gw] clone` line goes to stderr, so capture it with `2>&1`. Ignore the known-hosts warning and the remote `setlocale` warning when the clone line is present. `ver` prints a blank line first. A connection failure is BLOCKED, not PASS: `websocket: bad handshake` means the service token is wrong or missing, and `Permission denied (publickey)` means the key is wrong.
