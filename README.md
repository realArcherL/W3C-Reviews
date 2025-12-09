# W3C Reviews

Important Links
1. [How to do a security review](https://github.com/w3c/securityig/blob/main/administration/how-to-review.md)
2. How I conducted the review - @innotommy
    As there is no threat model, I create a threat model by myself using the Threat Modeling Manifesto:
    ● Ask four core questions:
    1. **What are we working on?** – Screen Orientation API: type/angle, events, lock/unlock.
    screen-orientation
    2. **What can go wrong?** – Cross-origin leakage, fingerprinting through divergence, UX/DoS.
    screen-orientation
    3.** What are we going to do about it?** – Permissions Policy, stronger normative privacy mitigations.
    screen-orientation-issues
    4. **Did we do a good enough job?** – Check impact on developers, UAs, and users; look for residual risk.
3. [Threat Model Manifesto](https://www.threatmodelingmanifesto.org/)



My first Spec Review

1. [screen-orientation](https://www.w3.org/TR/screen-orientation/) GitHub Issue: [Screen Orientation 2025-09-04 > 2025-10-16 #101](https://github.com/w3c/security-request/issues/101)
2. [CSS Color Adjustment Module Level 1](https://www.w3.org/TR/css-color-adjust-1/) GitHub Issue: [css-color-adjust-1 2025-09-17 > 2025-10-31 #104
](https://github.com/w3c/security-request/issues/104)
