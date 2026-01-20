# Review


1. https://www.w3.org/TR/css-color-adjust-1/
2. Security Review Request ([GitHub Issue](https://github.com/w3c/security-request/issues/104))
3. [Self-Review Questionnaire](https://github.com/w3c/csswg-drafts/issues/12815#issuecomment-3304507485) filled by the authors



---

# Security & Privacy Review

**Specification:** CSS Color Adjustment Module Level 1
**Reviewer role:** Security & Privacy
**Guides used:**

* Threat Modeling Guide: [https://w3c.github.io/threat-modeling-guide/](https://w3c.github.io/threat-modeling-guide/)
* Security & Privacy Questionnaire: [https://www.w3.org/TR/security-privacy-questionnaire/](https://www.w3.org/TR/security-privacy-questionnaire/)

---

## 1. System Description (Threat Modeling Guide)

### What is the system?

A CSS module that allows negotiation between **user preferences**, **user agent behavior**, and **author intent** for:

* Color schemes (`color-scheme`)
* Forced colors / high-contrast mode (`forced-color-adjust`)
* Print color adjustment (`print-color-adjust`)
* **NEW:** Forced-colors emulation for automation/testing

> ```text
> /* “This specification introduces a model and controls over automatic color adjustment by the user agent…” */
> ```
>
> (Introduction)

---

### Who are the actors?

* **Users**, including accessibility users (forced colors / high contrast)
* **Web authors**
* **User agents (browsers)**
* **Automation / testing frameworks**
* **Potential attackers** (cross-origin documents, fingerprinting scripts)

### Stakeholder List

> [!Tip]
> [TM Guide](https://w3c.github.io/threat-modeling-guide/):
> 1. Explicitly distinguish stakeholders vs observers/threat actors (5.2 focuses on “who is impacted,” then 5.3 covers threats).
> 2. Add a 1-table stakeholder inventory: Stakeholder → Value → Harm → Spec surface involved
> > Note: we do NOT model potential attackers, as over-characterizing attackers can lead to analysis bias that is better avoided.
> 
> Makes sense ^^ why we don't add attackers in the TM

#### **P0 — Must-model first**

**End users (including users relying on accessibility modes)**  
- **Value:** Usable and legible UI; privacy of user preferences (contrast, forced-colors, dark mode).  
- **Potential harm:** Disclosure of preferences enabling fingerprinting or profiling; loss of accessibility if forced-colors are overridden.  


**User agents (browser vendors / implementers)**  
- **Value:** Consistent behavior across implementations; ability to mitigate cross-site observation.  
- **Potential harm:** Underspecified behavior leading to inconsistent exposure surfaces that can be exploited for fingerprinting.  


#### **P1**

**Web authors (first-party sites)**  
- **Value:** Predictable rendering across color schemes; ability to respect user preferences.  
- **Potential harm:** Compatibility pressure to probe user settings; accidental accessibility regressions via `forced-color-adjust`.  


#### **P2**

**Regulators / policy bodies / auditors**  
- **Value:** Demonstrable privacy-by-design and accessibility-by-design.  
- **Potential harm:** Non-compliance risk if preference signals are exposed without clear boundaries or mitigations.  

#### **P3**

**Society / impacted groups (non-customers)**  
- **Value:** Reduced discrimination and profiling at scale.  
- **Potential harm:** Large-scale segmentation based on accessibility needs or appearance preferences.  
  
---

### What are the assets?

* User privacy (accessibility preferences, OS/UI state)
* Cross-origin isolation guarantees
* UI integrity and correct affordances
* Fingerprinting resistance

---

## 2. Threat Analysis (Threat Modeling Guide)

> Based on Section 5.3

> [!TIP]
> The section says output is “list of identified threats” but doesn’t force any structure. 
> We SHOULD probably provide a structure do this IMO:
> 
> Threat → impacted asset/stakeholder → preconditions (if any) → impact (e.g., fingerprinting, accessibility harm) → existing controls/Status.
> We already provide [Appendix 2: Threat Analysis Frameworks](https://w3c.github.io/threat-modeling-guide/#appendix-2-threat-analysis-frameworks) here but would still be nice to provide an example on what are we looking?

### What can go wrong?

#### T1: Fingerprinting via exposed user color preferences

The specification exposes user color preferences through computed styles.

> ```text
> /* “…exposes the user’s color preferences to the page via getComputedStyle(), which can increase fingerprinting surface.” */
> ```
>
> (Privacy Considerations)

**Impact:**

* Increased entropy
* Potential inference of accessibility-related settings
* Persistent cross-session signal (OS / UA preference)

**Status:**

* Acknowledged by the spec
* Accepted tradeoff for compatibility and accessibility

---

#### T2: Cross-origin state inference via timing side-channels

An embedded document may infer whether its color scheme matches its embedding document.

> ```text
> /* “It may be possible for an embedded document to use timing attacks to determine whether its own color-scheme matches that of its embedding iframe or not.” */
> ```
>
> (Security Considerations)

**Impact:**

* Cross-origin information leak (1-bit: match vs mismatch)
* Violates strict isolation expectations

**Status:**

* Explicitly documented
* No mitigations suggested

**Reviewer note:**
Even single-bit leaks are relevant under browser threat models, as timing side-channels can compose with other signals.

---

#### T3 (NEW): Additional observable state from forced-colors emulation

The specification introduces a new **emulated forced colors theme data** associated with each top-level traversable.

> ```text
> /* “For the purposes of user agent automation and application testing, this document defines the below emulations.” */
> ```
>
> ```text
> /* “Each top-level traversable has an associated emulated forced colors theme data…” */
> ```
>
> (Emulation section)

**Impact:**

* New state
* Forces style recalculation
* Alters system color resolution
* Potentially detectable via computed styles, layout, or timing

**Status:**

* Intended for automation/testing
* **Exposure boundaries not explicitly stated**

**Reviewer request:**
Clarify that emulation state is **not web-exposed** and is only controllable via UA automation interfaces.

---

## 3. Security & Privacy Questionnaire (Answered)

### 2.1 What information does this feature expose?

* User color preferences (light/dark)
* Forced-colors / high-contrast mode
* System color mappings (via computed styles)

> ```text
> /* “…exposes the user’s color preferences to the page via getComputedStyle()…” */
> ```

---

### 2.2 Does it expose the minimum amount of information necessary?

No. The spec explicitly states that minimizing exposure would break compatibility and readability.
Although the authors answered it as Yes here: https://github.com/w3c/csswg-drafts/issues/12815#issuecomment-3304507485

> ```text
> /* “Avoiding this comes with unfortunate drawbacks…” */
> /* “…lying about system colors… can result in… unreadable…” */
> ```

---

### 2.3 Does it expose personal data?

Indirectly yes. Forced-colors mode may correlate with accessibility needs.


---

### 2.4 Does it expose sensitive information?

Potentially yes (accessibility-related signals). BUT this is only if the browser could be fingerprinted

---

### 2.6 Does it introduce new state?

**Yes (NEW).**

* Emulated forced colors theme data
* Scoped to a top-level traversable
* Triggers style recalculation

Does this also count? 

```text
Out of the changes in https://drafts.csswg.org/css-color-adjust-1/#changes, the changes to emoji rendering in forced colors mode is the only new change that may fall into this category.
```
Source: https://github.com/w3c/csswg-drafts/issues/12815#issuecomment-3304507485


> ```text
> /* “Each top-level traversable has an associated emulated forced colors theme data…” */
> ```

---

### 2.7 Does it expose information about the underlying platform?

Yes (OS / UA color preferences and forced colors behavior). [Indirectly/directly?]

---

### 2.8 Does it allow an origin to modify user agent or OS state?

No (except via automation tooling; not stated explicitly).

---

### 2.10 Does it introduce new script execution mechanisms?

[No](https://github.com/w3c/csswg-drafts/issues/12815#issuecomment-3304507485).

---

### 2.12 Does it degrade security guarantees (e.g., isolation)?

Potentially, via timing side-channels between embedding and embedded documents. (but this is well documented, not sure if want to make any new suggestions)

---

### 2.16 Are Security and Privacy Considerations provided?

Yes.

---

### 2.21 Does it allow sites to infer assistive technology use?

Indirectly yes, via forced-colors detection.

---

## 4. Review of Recent Spec Changes (Security-Relevant)

### Change A — Forced-colors emulation (NEW)

* Introduces synthetic forced-colors state
* Bypasses OS theme
* Intended for automation/testing
* Not explicitly scoped as non-web-exposed

> ```text
> /* “To set emulated forced colors theme data…” */
> /* “UAs must consider this a change that requires style recalculation.” */
> ```

**Risk:**
Fingerprinting amplification and environment detection if observable. (this is known)

---

### Change B — Forced colors applies to `<color>` components of all properties

* Broader application increases observable differences
* Expands fingerprinting surface. note: I believe this should be called out because every other component becomes exposed to this, and we should be explicity about it

HOWEVER, also want to point out that ⚠ The fingerprinting surface already existed; this change standardizes it. (the discussion and reasoning [here](https://github.com/w3c/csswg-drafts/issues/5710) and actual statement in the privacy section of the RFC: https://www.w3.org/TR/css-color-adjust-1/#privacy)

> ```text
> /* “When forced colors mode is active… the <color> components of all properties… are force-adjusted…” */
> ```

---

### Change C — Emoji fallback to monochrome

* Platform- and UA-dependent behavior
* Potentially measurable rendering differences (I have a hunch, but no practical exploit)

BUT
* Windows High Contrast Mode already renders emoji as monochrome.
* This change aligns Chrome/Firefox behavior with Windows and Edge: https://github.com/w3c/csswg-drafts/issues/8064
* Font fingerprinting was already possible with normal text; emoji are not special.

> ```text
> /* “UAs should force any emoji… to its monochrome variant…” */
> ```

---

> [!TIP]
> This is what I meant by structure even though users would be the ones deciding which "Threat Analysis Frameworks" they want to pick

## STRIDE Threat Analysis — CSS Color Adjustment Module Level 1

| S.No | STRIDE Category | Threat | Impacted asset / stakeholder | Preconditions | Impact | Existing controls / Status |
|-----|----------------|--------|------------------------------|---------------|--------|----------------------------|
| 1. | **Information Disclosure** | Exposure of user color preferences via computed styles | End users (privacy), User agents | Page can read `getComputedStyle()`; user has color preferences set | Fingerprinting, inference of accessibility or OS/UA state | Explicitly acknowledged in Privacy Considerations; accepted trade-off for compatibility/accessibility |
| 2. | **Information Disclosure** | Cross-origin inference of color-scheme match via timing | End users (privacy), Cross-origin isolation guarantees | Embedded cross-origin iframe; measurable timing differences | 1-bit cross-origin state leak (match vs mismatch); composable with other signals | Explicitly documented in Security Considerations; no mitigations specified |
| 3. | **Information Disclosure** | Detectability of forced-colors / high-contrast mode | End users (accessibility privacy) | Forced colors enabled; styles rendered accordingly | Inference of assistive technology usage; profiling risk | Documented; considered necessary for correct rendering |
| 4. | **Information Disclosure** | New observable state from forced-colors emulation | End users, User agents | Emulation enabled via UA automation; rendering/state changes observable | Additional fingerprinting or environment-detection surface | Intended for automation/testing; exposure boundaries not explicitly stated |
| 5. | **Tampering** | Author overrides of forced colors via `forced-color-adjust` | End users (accessibility users) | Author opts out of forced colors | Reduced readability; accessibility regression | Spec allows override; relies on author responsibility |
| 6.  |**Denial of Service** | Style recalculation triggered by emulated forced-colors changes | User agents | Emulation state changes; large documents | Performance impact; potential rendering jank | UA-managed; scoped to top-level traversable |
| 7. | **Repudiation** | Lack of attribution for preference-derived signals | Regulators / auditors | Preference signals exposed without explicit consent surface | Difficulty demonstrating privacy-by-design | Not addressed explicitly |
| 8. | **Elevation of Privilege** | N/A | N/A | N/A | No privilege escalation introduced | Not applicable |
| 9. | **Spoofing** | N/A | N/A | N/A | No identity spoofing introduced | Not applicable |


---


## 5. Summary of Findings / Requests

### Findings

* The spec knowingly increases fingerprinting surface and documents it.
* A timing-based cross-origin inference is acknowledged.
* **New emulation feature introduces additional state without explicit exposure constraints.**

### Requests / Suggestions


- Forced-colors emulation
  - Why: new emulation state affects rendering; unclear if web content can see/control it → fingerprinting / env detection risk
  - What: say clearly this is automation-only, not web-exposed
  - How: add a short note like “intended only for UA automation/testing; not observable by web content”

- Iframe canvas timing
  - Why: spec already says timing attacks may be possible; no guidance given
  - What: suggest minimizing timing differences where possible
  - How: non-normative note encouraging reduced timing distinguishability

- Fingerprinting (recent changes)
  - Why: forced-colors now applies to all `<color>` props; emoji fallback + emulation add new observable behavior
  - What: call out incremental fingerprinting impact vs earlier versions
  - How: short sentence added to Privacy Considerations
 

Other Considerations:
The specification does not address how **browser extensions**, **assistive technologies** (such apple assist etc), or managed environment policies (mostly through corporate security teams) may modify or override color adjustment behavior. These actors can introduce observable, origin-independent visual changes, creating fingerprinting vectors, UI integrity risks, and ambiguity around the source of color adjustments. The spec should clarify precedence, observability expectations, and threat considerations for non-author interventions.

