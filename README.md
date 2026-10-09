# Stack Buffer Overflow Explorer

An interactive, browser-based teaching module that visualizes how a **stack buffer overflow** works. Type an input string, watch `strcpy()` copy it byte-by-byte into a fixed-size buffer, and see what happens when the input is longer than the buffer: the extra bytes spill into adjacent stack memory — the saved frame pointer and the return address.

It is a **safe teaching tool**. It contains no shellcode, no exploit, and no attack payload. The visualization deliberately stops at the moment the return address is overwritten — the point the concept is meant to illustrate.

## Demo

Open `index.html` in any modern browser. No build step, no dependencies, no server required.

```
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
open index.html        # macOS
# or just double-click the file
```

## What it shows

The app renders a simplified x86-style stack frame with the high address at the top:

```
┌─────────────────────────┐  high address
│   return address (RIP)  │
│   saved frame ptr (RBP) │
│   [ stack canary ]      │  ← optional
│   buffer[N]             │
└─────────────────────────┘  low address
```

Because local buffers sit at **lower** addresses than the saved registers, and `strcpy` writes toward **higher** addresses, an over-long copy marches upward out of the buffer into the caller's saved state. The visualizer animates exactly this, coloring each byte by the region it lands in.

## Interactive controls

| Control | Effect |
| --- | --- |
| **Input box** | The string passed to the vulnerable function. Preset chips load a safe case, a mild overflow, and a full return-address overwrite. |
| **Buffer size slider** (4–16 bytes) | Resizes `buffer[N]`; the code snippet and stack diagram update live. |
| **Stack canary toggle** | Inserts a canary between the buffer and saved registers and models the `__stack_chk_fail` abort that detects the overflow at return time. |
| **Run `strcpy()` button** | Animates the copy one byte at a time with a running address meter. |
| **Reset button** | Clears the frame. |

## Outcomes it demonstrates

- **No overflow** — input fits; adjacent memory is untouched (with a note about the null terminator).
- **Saved frame pointer corrupted** — overflow past the buffer but short of the return address.
- **Return address overwritten** — the classic control-flow hijack condition.
- **Overflow detected / aborted** — with the canary enabled, the overflow is caught at `return` and the process aborts before the corrupted return address is used, showing why stack canaries are an effective (if DoS-prone) mitigation.

## How it's built

A single self-contained `index.html`: vanilla HTML, CSS, and JavaScript, roughly 300 lines, no frameworks. Memory is modeled as a flat array of byte cells indexed by ascending address; the layout (`buffer → [canary] → saved RBP → return address`) is computed from the current controls and re-rendered on every change and on every animated write.

## Educational scope & disclaimer

This project is for learning how the vulnerability works and why mitigations exist. It is **not** an exploit, does not generate real addresses or payloads, and cannot be pointed at any real program. Please use it only for study.

## License

MIT — free to use and adapt for educational purposes.
