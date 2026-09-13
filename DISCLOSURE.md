# Disclosure

Responsible disclosure for findings against **third-party** controllers.

This policy exists before F2 (trust). Do not publish third-party defects
until the steps below have been followed.

## Default embargo

90 days from first private contact, or until the maintainer publishes a fix
or a coordinated advisory — whichever is earlier.

## How to report

1. Prefer the target project's `SECURITY.md` or listed security contact.
2. If none exists, open a **private** security advisory on their GitHub repo
   or email the listed maintainers.
3. Attach ReconcileBench evidence JSON: target/version, Kubernetes version,
   scenario, findings, as-of date.
4. Do not include credentials, customer data, or a public exploit script.

## How we report

OpenReconcile maintainers use the same path. We do not post "we found N bugs
in $operator" as marketing.

## Our own projects

Use GitHub private vulnerability reporting on the affected repo, or email
the address listed in that repo's `SECURITY.md` once it exists.
