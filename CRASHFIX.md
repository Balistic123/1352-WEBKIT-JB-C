# Kernel crash mitigations (jb.js v11-crashfix)

## Causes addressed

1. **`cr_prison` bug** — jailbreak wrote `KBASE + k_prison0` (address of the kernel
   `prison0` *variable*) into `ucred->cr_prison` instead of `*prison0`
   (`read8(KBASE + k_prison0)`). Wrong prison pointers often panic on later
   syscalls or process teardown.

2. **Exit restore race** — after **kpatch** / **payload2.bin**, the chain called
   `jbRestoreHook()` to put sandbox creds/fdesc back. That fights the running HEN
   thread and unbalanced kernel refcount paths → panic when you close User Guide.

## Behavior now

- Uses **`rootPrison = read8(prison0Slot)`** for `CR_PRISON`.
- **Skips** ucred/fdesc restore when `kpatched` or `payloadRunning`.
- UI message: **reboot PS4 before closing the browser** after success.

## After a successful run

1. Confirm HEN / homebrew as needed.
2. **Reboot the console** (power menu → restart). Do not just close the browser.
3. The exploit leaves kernel `.data` dirty until reboot (by design in this chain).

## Safer testing (no kpatch/payload)

`jb.html?log=1&patch=0&payload=0` — stops after sandbox proof if KRW passes (still dirty from 663 read path; reboot if anything kernel-touching ran).

## Deploy

Push **`jb.js`** and **`jb.html`** (`?v=11-crashfix`). Clear GitHub Pages / appcache or bump manifest if offline cache serves old JS.
