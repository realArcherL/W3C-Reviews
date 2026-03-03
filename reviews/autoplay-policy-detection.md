# Review

1. Link to the Spec: https://www.w3.org/TR/autoplay-detection/
2. Security Review Request: https://github.com/w3c/security-request/issues/48
3. [Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/): https://github.com/w3c/autoplay/blob/main/security-privacy-questionnaire.md

# Security Review
1. Threat Modeling Guide: https://w3c.github.io/threat-modeling-guide/
2. Security & Privacy Questionnaire: https://www.w3.org/TR/security-privacy-questionnaire/

## Threat Model

### What is the Spec about?
> The Autoplay Policy Detection spec “provides web developers the ability to detect if automatically starting the playback of a media file is allowed in different situations.

> Most user agents have their own mechanisms to block autoplaying media, and those mechanisms are implementation-specific. Web developers need to have a way to detect if autoplaying media is allowed or not in order to make actions, such as selecting alternate content or improving the user experience while media is not allowed to autoplay. For instance, if a user agent only blocks audible autoplay, then web developers can replace audible media with inaudible media to keep media playing, instead of showing a blocked media which looks like a still image to users. If the user agent does not allow any autoplay media, then web developers could stop loading media resources and related tasks to save the bandwidth and CPU usage for users. 

The spec currently handles:

HTMLMediaElement (`<video>`/`<audio>`) and Web Audio API (AudioContext)

> Autoplay detection can be performed through the Navigator object.



