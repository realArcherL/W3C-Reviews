# W3C Reviews

## Important Links

1. [How to do a security review](https://github.com/w3c/securityig/blob/main/administration/how-to-review.md)
2. [Threat Modeling Guide](https://w3c.github.io/threat-modeling-guide/)
3. [Privacy Questionaire](https://www.w3.org/TR/security-privacy-questionnaire/)
4. How I conducted the review - @innotommy
   As there is no threat model, I create a threat model by myself using the Threat Modeling Manifesto:
   ● Ask four core questions:
   1. **What are we working on?** – Screen Orientation API: type/angle, events, lock/unlock.
      screen-orientation
   2. **What can go wrong?** – Cross-origin leakage, fingerprinting through divergence, UX/DoS.
      screen-orientation 3.** What are we going to do about it?** – Permissions Policy, stronger normative privacy mitigations.
      screen-orientation-issues
   3. **Did we do a good enough job?** – Check impact on developers, UAs, and users; look for residual risk.
5. [Threat Model Manifesto](https://www.threatmodelingmanifesto.org/)
6. UX Designs issues can also be problmeatic
7. [Web sustainability guide](https://www.w3.org/TR/web-sustainability-guidelines/#user-controlled-media)
8. [Attention is also going to be an issue](https://www.w3.org/WAI/standards-guidelines/act/rules/aaa1bf/proposed/)

## Spec Reviews

1. [Screen Orientation](https://www.w3.org/TR/screen-orientation/) — Review: [reviews/screen-orientation.md](reviews/screen-orientation.md) — GitHub Issue: [#101](https://github.com/w3c/security-request/issues/101)
2. [CSS Color Adjustment Module Level 1](https://www.w3.org/TR/css-color-adjust-1/) — Review: [reviews/css-color-adjust-1.md](reviews/css-color-adjust-1.md) — GitHub Issue: [#104](https://github.com/w3c/security-request/issues/104)
3. [Autoplay Policy Detection](https://www.w3.org/TR/autoplay-detection/) — Review: [reviews/autoplay-policy-detection.md](reviews/autoplay-policy-detection.md) — GitHub Issue: [#48](https://github.com/w3c/security-request/issues/48)
4. [CSS View Transitions Module Level 2](https://drafts.csswg.org/css-view-transitions-2/) — GitHub Issue: [#66](https://github.com/w3c/security-request/issues/66)
5. [WebMCP](https://webmachinelearning.github.io/webmcp/) — Review: [reviews/webmcp.md](reviews/webmcp.md) — GitHub Issue: [#154](https://github.com/webmachinelearning/webmcp/issues/154)

## Presentation

You can find the presentations here: https://realarcherl.github.io/W3C-presentations/
