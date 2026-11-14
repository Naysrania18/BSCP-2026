# BSCP-2026

Solved PortSwigger Web Security Academy labs with written walkthroughs, collected while preparing for the Burp Suite Certified Practitioner (BSCP) exam.

## Overview

Each file documents one lab: the vulnerability, how it was found in Burp Suite, and the payload or request that solved it. The aim is a searchable personal reference, not a replacement for the Web Security Academy.

## Coverage

| Topic | Labs |
|---|---|
| SQL injection | WHERE clause data retrieval, login bypass |
| Cross-site scripting | Reflected, stored and DOM-based (document.write, jQuery sinks) |
| Access control | IDOR, parameter-based roles, URL-based and method-based bypass |
| Authentication | 2FA bypass, username enumeration |
| CSRF | No-defences case |
| OS command injection | Simple case |
| API testing | Using documentation, server-side parameter pollution |

## Structure

One Markdown file per lab, named after the lab title, in the repository root.

## Tools

Burp Suite, a browser with developer tools.

## Scope

All labs are PortSwigger's intentionally vulnerable training targets. Nothing here targets real systems.

## Status

Preparation in progress. This repository does not claim the BSCP certification.

## License

No license file yet. Author: [Sandeep Kumar Naysrania](https://github.com/Naysrania18)
