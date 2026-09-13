---
title: "The /.well-known/hosting-provider URI"
date: "2026-09-13"
reason: out-of-scope
revisit: "If something a visitor, crawler or agent uses ever reads the file — a browser, a registrar, an abuse-reporting flow — and the specification grows past a hint its own author calls spoofable. More hosts serving it would not be enough on its own: the string would still be the platform's claim about itself rather than anything the site says."
sources:
  - title: "IANA — Well-Known URIs registry (hosting-provider, provisional, registered 2020-07-21)"
    url: "https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml"
    publisher: "IANA"
  - title: "hosting-provider Well-Known Resource Identifier"
    url: "https://github.com/Automattic/hosting-provider"
    publisher: "Automattic"
  - title: "RFC 8615 — Well-Known Uniform Resource Identifiers (URIs)"
    url: "https://www.rfc-editor.org/rfc/rfc8615"
    publisher: "IETF"
---

A domain's DNS usually points at a CDN rather than at whoever actually stores the files, which makes "who hosts this site?" surprisingly hard to answer from outside. `/.well-known/hosting-provider` is Automattic's answer: a participating host serves a bare `text/plain` string at that path naming itself — a URL, a domain, or a business name — and a third party combining that with hostname and IP checks can work out, with reasonable confidence, which provider or reseller account is serving the content. IANA carries the suffix as provisional, registered in July 2020; the specification is a single README that has not changed since April 2019.

It does not land here, and the reason is not adoption. A website does not write this file — its host does, in server configuration the site's author usually cannot reach, about the host rather than about the site. Nor is the value meant to be believed: the README's own security considerations note that anyone can spoof it and self-report as a different provider, which is why every use case it describes pairs the string with independent hostname and IP evidence. A spec page needs an instruction to give and an outcome to check, and here there is neither — a reader following it would be told to ask their hosting provider to make an unverifiable claim to nobody in particular.

That makes it a useful counterpart to [`/.well-known/scitt-keys`](/considered/#scitt-keys-well-known), which failed a different half of the same test. Scitt-keys failed on who _serves_ the file: a transparency service, not a website. Hosting-provider is genuinely served by the website's own origin, and still fails — on who _benefits_. The beneficiary is a third party doing attribution or abuse investigation, and the origin is merely the subject of the enquiry. The test that survives both cases is the same one that admits [`security.txt`](/spec/security/security-txt/) and [`change-password`](/spec/well-known/change-password/): serving the file has to make this origin better for the people and programs that visit it. Being in the registry never settles that, and neither does serving the file yourself.
