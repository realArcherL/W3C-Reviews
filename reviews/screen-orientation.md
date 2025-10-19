# Screen Orientation API — Spec Summary (WD 09 Aug 2023)

## Abstract

>  The Screen Orientation specification standardizes the types and angles for a device's screen orientation, and provides a means for locking and unlocking it. The API, defined by this specification, exposes the current type and angle of the device's screen orientation, and dispatches events when it changes. This enables web applications to programmatically adapt the user experience for multiple screen orientations, working alongside CSS. The API also allows for the screen orientation to be locked under certain preconditions. This is particularly useful for applications such as computer games, where users physically rotate the device, but the screen orientation itself should not change. 

1. https://www.w3.org/TR/screen-orientation/#abstract (Updated: https://w3c.github.io/screen-orientation/#screenorientation-interface)
2. https://github.com/w3c/security-request/issues/101
3. https://github.com/w3c/screen-orientation/issues/201#issuecomment-1289977250
4. Test Suite: https://wpt.live/screen-orientation/



## Security Review

- **What it does (goal):** Exposes current screen orientation **type** and **angle**, fires a **change event**, and (when conditions are met) lets web apps **lock/unlock** orientation—primarily for games, video, and installed apps. [W3C](https://www.w3.org/TR/screen-orientation/)
- **Core surface (high level):**
    - `window.screen.orientation.{type, angle, onchange}`
    - `orientation.lock(OrientationLockType)` → Promise
    - `orientation.unlock()`
- **Key processing rules (developer-impacting):**
    - Locking is **gated by common safety checks**; UAs commonly require **fullscreen and/or user activation**. Promise **resolves after** the change event (predictable state).
    - **Fullscreen interaction:** UA **SHOULD** restrict `lock()` to **simple fullscreen** docs; **exiting fullscreen fully unlocks** orientation. (Pre-lock + cleanup.)
    - **Web App Manifest:** Installed apps may declare a default orientation; UA **SHOULD** require **fullscreen display mode** as a pre-lock condition.

> Note: Which browsers support `lock()`/`unlock`: https://developer.mozilla.org/en-US/docs/Web/API/ScreenOrientation

- **Privacy & security (what matters):**
    - **Fingerprinting:** `type/angle` are recognized **fingerprinting vectors**; spec allows UAs to **coarsen/spoof** (e.g., only primary types, angle 0/90) and **suppress events** in privacy-sensitive modes. (Non-normative “MAY”.)
    - **Hidden/embedded contexts:** Locking from **hidden/inactive** documents is rejected (conformance tests); **iframes** require **Permissions Policy** opt-in (`allow="orientation-lock"`). (Behavior verified in WPT/MDN.)
    - **Accessibility:** Refer to **WCAG 2.2 SC 1.3.4 Orientation**—content/functionality should work regardless of orientation; if an orientation is essential, inform the user. (Authoring guidance.)
- **Why the recent changes help:**
    - **Active/visible only + event-then-promise** tighten security/ergonomics (no background manipulation; deterministic sequencing) without harming common use cases.
 


| STRIDE | Threat | Example Scenario | Existing Defense (what is enforced) | Known Gaps / Weaknesses | Reference / Section to check |
|---|---|---|---|---|---|
| **Spoofing / Impersonation** | A malicious site pretends to be a native app by locking orientation and forcing UI layout | A phishing site enters fullscreen, locks orientation to “landscape,” and displays a banking UI that never rotates when user tilts device | UA **SHOULD** restrict `lock()` to simple fullscreen documents; exiting fullscreen triggers “fully unlock” steps. | None (iOS even blocks this) [1]  | **Spec §9 “Interaction with Fullscreen API”** — “A user agent SHOULD restrict the use of lock() to simple fullscreen documents”, and “When a document exits fullscreen, it also runs the fully unlock the screen orientation steps.” (W3C) |
| **Tampering / Manipulation** | Script rapidly toggles lock/unlock or rotates unexpectedly to break UI alignment or lure clicks | A page toggles orientation lock/unlock every few frames to misalign buttons, tricking user into clicking hidden UI | Only valid enumerated lock types allowed by spec; `lock()` promise is resolved only **after** orientation change event; thus reducing racey UI states, enforcing predictable ordering | There is no built-in rate limit or throttle to prevent high-frequency toggling; a page that meets fullscreen + gesture may still disrupt layout (although this is a bit far-fetched and not sure it would work in real life) **OR** Encourage (non-normative) debounce/heuristics for repeated lock/unlock requests within short intervals, while keeping current event-ordering rules intact | **Spec §5.2 “Locking the screen orientation”** and **§9 “Interaction with Fullscreen API”** define valid lock types and preconditions for lock/unlock behaviors. (W3C) |
| **Information Disclosure / Fingerprinting** | Trackers record orientation type & angle across visits or tabs to help re-identify user | A tracker notes that on multiple sites, a device always reports `angle = 270` in landscape secondary, and ties that pattern to user sessions, another variation is through errors [2] | Spec recognizes `type` & `angle` as fingerprinting vectors and **permits** browsers to **spoof / quantize** orientation (e.g., only primary types, quantize angles, suppress change events) | None | **Spec §12 “Privacy and Security Considerations”** describes these mitigations. (W3C) |
| **Denial of Service / Usability / Accessibility** | Orientation lock causes content to be unusable or disorients user | On a device mounted in fixed orientation (e.g., wheelchair), forcing an orientation prevents use; or frequent lock switching annoys user | UA auto-unlocks when exiting fullscreen; spec includes **Accessibility Considerations** referencing WCAG’s Orientation Success Criterion, requiring that essential content remain usable regardless of orientation | The spec’s accessibility requirement is **guidance**, not enforced by UA. Also, there is no mandatory user-visible indicator that a site currently holds orientation lock **although OSes already show rotation-lock system indicators** (e.g., iOS status bar). **Should we recommend user-visible indicator? Would that hamper UX? OR leave non-normative UA UI.** | **Spec §11 “Accessibility considerations”** warns developers to support both orientations and advise users. (W3C) |
| **Elevation of Privilege (Cross-Origin / iframe)** | An embedded ad iframe attempts to lock orientation of parent page | A malicious third-party iframe in a page tries `screen.orientation.lock('landscape')` to override the host’s view | UAs should enforce **Permissions Policy** so that iframes by default cannot lock orientation unless explicitly allowed (`allow-orientation-lock`) | None | **Spec §9 “Interaction with Fullscreen API”** hints limiting lock to fullscreen context; embedding/iframe control is convention with **Permissions Policy** (documented in MDN / UA implementation) (W3C) |
| **Side-Channel / Sensor Inference (bounded)** | An attacker attempts to infer tapping or motion by observing rapid orientation changes | During gameplay, user rotates device slightly while tapping; attacker infers patterns from `orientationchange` timestamps | Screen Orientation API gives only **coarse** orientation changes (type + angle), not continuous sensor streams; browsers now **gate DeviceMotion/DeviceOrientation** APIs, reducing side-channel possibility | None (If an implementation erroneously gives more frequent events or intermediate angles, the side-channel risk increases **and is probably a quality bug**.) | **Spec §11 “Accessibility considerations”** (W3C) |


## The issues

### References

1. [https://developer.mozilla.org/en-US/docs/Web/API/Screen/orientation](https://developer.mozilla.org/en-US/docs/Web/API/ScreenOrientation)
2. https://github.com/w3c/screen-orientation/issues/260 

### Security concerns

I didn't find any issues with the current spec. Few things I had on mind and wanted to run past the IG group.

> There is no built-in rate limit or throttle to prevent high-frequency toggling; a page that meets fullscreen + gesture may still disrupt layout (although this is bit far fetched and I am not sure if it would work in real life) OR Encourage (non-normative) debounce/heuristics for repeated lock/unlock requests within short intervals, while keeping current event-ordering rules intact

Should we recommend such a rate-limit?

> The spec’s accessibility requirement is guidance, not enforced by UA. Also, there is no mandatory user-visible indicator that a site currently holds orientation lock ALTHOUGH OSes already show rotation-lock system indicators (e.g., iOS status bar). Should we recommend user-visible indicator? Would that hamper user experience? OR This is better left non-normative UA UI

Would recommending “orientation locked” indicator be a good idea? 
