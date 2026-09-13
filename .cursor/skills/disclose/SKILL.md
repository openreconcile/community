---
name: disclose
description: >-
  File a finding against a third-party operator without going public first.
  Use when ReconcileBench (or any engine) produced evidence about code you
  do not maintain.
---

# disclose

Follow `DISCLOSURE.md` (community repo). If that file is still a stub, **stop
and do not publish**.

Minimum:

1. Contact the project's SECURITY / maintainers privately.
2. Include target version, Kubernetes version, scenario, evidence JSON, reproducer.
3. Wait the embargo in `DISCLOSURE.md` (default 90 days) unless they publish first.
4. Do not tweet, blog, or open a public issue with exploit-like steps.
5. After disclosure, a public write-up may cite the evidence format, not a
   weaponised repro.
