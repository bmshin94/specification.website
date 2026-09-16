---
title: '<meta name="text-scale">'
date: "2026-09-16"
reason: too-early
revisit: "A second engine shipping it, and the section leaving the Editor's Draft — a Working Draft or Candidate Recommendation of CSS Fonts 5 that still contains `text-scale-meta`. Either alone is not enough: a stable definition nobody else implements still leaves a one-browser recommendation, and a second implementation of an unstable definition can be obsoleted by the next editorial pass."
sources:
  - title: "CSS Fonts Module Level 5 — The text-scale meta tag"
    url: "https://drafts.csswg.org/css-fonts-5/#text-scale-meta"
    publisher: "W3C CSS Working Group"
  - title: '<meta name="text-scale">'
    url: "https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta/name/text-scale"
    publisher: "MDN"
---

Operating systems let people enlarge text system-wide, and on mobile browsers that setting has historically had no reliable effect on a web page's root font size. `<meta name="text-scale" content="scale">` is the opt-in that changes this: it makes the root element's initial `font-size` scale in proportion to the OS and browser text-size settings, so a layout written in `rem` and font-size keywords grows with the user's preference — including `@media` breakpoints expressed in `rem`, which then move with the text rather than stranding it. It also turns off the browser's own text-autosizing heuristics, and on desktop it populates `env(preferred-text-scale)`. The default, `legacy`, is what every page gets today.

This is exactly the shape of thing the spec covers: a single element in the `<head>`, checkable from outside, whose "why" is about visitors rather than developers. It is not a build technique. What rules it out for now is the state of the definition and the state of support. The normative text lives in an Editor's Draft of CSS Fonts Level 5 — the CSS Working Group's own header on that document says it is for discussion only and may change at any moment — and the attribute ships only in Chromium: Chrome, Edge and Chrome Android since version 146 in March 2026, plus WebView and Opera. Firefox and Safari have not implemented it, which is the half of the market where OS text-size settings are most often the user's only lever.

The risk of writing the page early is not merely that the advice would be premature. Opting in is a commitment: MDN warns that a page carrying `content="scale"` must be tested against the full scaling range of its target platforms, which on mobile runs from 200% to beyond 300%. A spec entry telling sites to add one `<meta>` tag would understate that, and the honest version of the page — add the tag, then rebuild your layout to survive triple-size text — is advice about sizing in relative units, which [the viewport meta page](/spec/foundations/meta-viewport/) already reaches from the pinch-zoom side. Until a second engine makes the tag worth the retrofit, that is where the useful guidance lives.
