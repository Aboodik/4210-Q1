# Reflection

> **Note to me:** This is a draft based on a typical build. The assignment asks about *my own* experience, so I should edit the specifics — swap in the prompts I actually used, the bugs I actually hit, and what genuinely clicked — before submitting.

## What I built and why

I built an interactive visualization of a **stack buffer overflow**. The user types an input string, chooses a buffer size, optionally turns on a stack canary, and then runs an animated `strcpy()` that copies the input into the buffer one byte at a time. When the input is longer than the buffer, the animation shows the extra bytes spilling out of the buffer and into the saved frame pointer and the return address.

I chose buffer overflow (over integer overflow or Thompson's "Trusting Trust") because the mechanism is fundamentally *spatial* — it is about memory layout and the direction a copy writes — and that is exactly the kind of thing a static diagram in the slides never made fully concrete for me. Animating it seemed like the best way to turn an abstract picture into something I actually understood.

## How it deepened my understanding

The detail that finally stuck was the **direction** of the whole thing. I had memorized "overflow overwrites the return address" without really internalizing *why* it goes there and not somewhere else. To animate it correctly I had to get the layout right: local variables sit at lower addresses, the saved frame pointer and return address sit at higher addresses, and `strcpy` writes from low to high. Only then does it make sense that an over-long string climbs upward out of the buffer into the caller's saved state. Having to encode that as an array indexed by address — and watch the animation break when I had it backwards — taught me the concept more firmly than reading about it did.

Modeling the **stack canary** had a similar effect. I originally pictured a canary as something that blocks the overwrite. Implementing it made me realize it does no such thing: the bytes still get clobbered, and the check only happens later, at `return`. The protection is that the program *detects* the tampering and aborts before using the corrupted return address. Writing the "program aborted" branch made the real trade-off obvious — a canary turns a potential code-execution bug into a crash, which is much better but still a denial of service.

## Workflow I used

I used an AI assistant as a pair programmer rather than a one-shot generator:

1. **Framing first.** I described the concept, the controls I wanted (input box, buffer-size slider, canary toggle, run button), and the hard constraint that it must be a safe teaching tool with no exploit code. Being explicit about scope up front kept the output on the right side of that line.
2. **Iterating on correctness.** My first instinct for the memory model had the addresses running the wrong way, so the overflow appeared to flow away from the return address. I had to reason through the real stack direction and correct the layout — a good reminder that I can't outsource understanding; I still have to check the model against what I know is true.
3. **Refining the explanations.** I spent real effort on the status text for each outcome (safe / frame-pointer corrupted / return-address overwritten / canary-detected), because that narration is what makes it *educational* rather than just a moving diagram.
4. **Testing by hand.** I clicked through the preset inputs and edge cases (empty input, exact-fit input, maximum length) and adjusted the byte classification until each case reported the right verdict.

## What I'd do next

If I extended it, I would add a second tab contrasting this with an off-by-one overflow, or show the null-terminator's role more explicitly. But for the core goal — making the mechanism of a stack buffer overflow genuinely intuitive — building the visualization taught me more than studying it ever did.
