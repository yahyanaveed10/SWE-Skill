# Dependency and Supply Chain Management

Adding a dependency is a long-term commitment. The package becomes part of your attack surface, your build time, your runtime memory, your operational complexity, and your security posture. Most of the cost is invisible at the moment of `npm install`.

Agents are particularly bad at this. They reach for a library when a 10-line function would do, pin to floating versions, and add transitive dependencies without checking maintenance status, licence, or security history.

---

## Version pinning strategies

The lock file is the source of truth for what gets installed. The `package.json` / `requirements.txt` / `Cargo.toml` declares what you *accept*; the lock file records what you *got*.

**Exact pinning (`1.2.3`):** the dependency is pinned to a specific version. Updates require an explicit change. Most predictable, hardest to keep current.

**Caret range (`^1.2.3`):** accepts compatible minor and patch updates (1.x.x). Standard for most JavaScript packages. Trusts the package author's semver discipline — which varies.

**Tilde range (`~1.2.3`):** accepts patch updates only (1.2.x). Tighter than caret; appropriate for less-trusted packages.

**Floating tags (`latest`, `*`):** never use in production. Means "give me whatever exists right now." Reproducible builds become impossible.

**The hard rule: commit the lock file.** Without a committed lock file, two installs produce two different sets of versions. CI builds become non-reproducible. Bug reports cannot be reproduced because the bug version is gone.

---

## Evaluating a new dependency

Before adding a dependency, answer:

**Is the functionality needed?** Could this be 10-50 lines of in-house code? A utility library imported for one function (e.g., `lodash` for `_.debounce`) is a poor trade — you ship the entire library or rely on tree-shaking that may not work. Write the function. The cost of maintaining 10 lines is less than the cost of a dependency.

**Maintenance signal.** Look at the project. Recent commits? Open issues being addressed? Open security advisories? A library with no commits in 3 years is unmaintained — security issues will not be fixed; compatibility with new platforms will not be added.

**Transitive footprint.** A small package that depends on 20 other packages brings 20+ packages into your build. Run `npm ls`, `pip show`, or your equivalent to see the full tree. A single 50KB direct dependency can pull in megabytes of transitive dependencies.

**Licence.** Permissive (MIT, Apache 2.0, BSD) — usually fine. Copyleft (GPL, AGPL) — restricts how you can ship your software; check with whoever owns licence policy. Custom or unclear licences — high risk; avoid.

**Popularity is not safety.** A package with millions of weekly downloads can still be malicious or abandoned. Popularity correlates with safety but does not guarantee it. The xz-utils backdoor (2024) was in a widely-used package; the malicious code was pushed by a maintainer who had built up trust over years.

**Maintainer trust signals.** Multiple active maintainers reduce risk. A single maintainer who recently took over the package is a yellow flag. A package transferred between owners with no community notification is a red flag.

---

## Vulnerability scanning

Automated scanning of dependencies for known CVEs is a baseline; without it, you ship known vulnerabilities by accident.

**Where it goes:**
- In CI on every PR, blocking merge for high-severity vulnerabilities in production dependencies
- On a schedule (daily or weekly) for the main branch, opening issues or PRs for new vulnerabilities discovered after merge
- Before promoting an artifact to production (deployment-time scan)

**Tools (concept-agnostic):** GitHub Dependabot, Snyk, npm audit, pip-audit, Trivy, Grype, etc. Most CI platforms have built-in support.

**The trap with vulnerability scanners:** noisy output produces alert fatigue. Filter by:
- Severity (critical and high block; medium and low go to backlog)
- Whether the vulnerable function is actually called by your code (some scanners do reachability analysis)
- Whether the vulnerable code path is reachable from external input

Triaging every CVE in a 5000-package transitive dependency tree is impossible; triaging the ones that affect production is necessary.

---

## Supply chain attack signals

Supply chain attacks compromise software not by attacking your code, but by compromising a dependency you trust.

**Common forms:**

**Typosquatting.** A malicious package with a name similar to a real package (`lodahs` for `lodash`, `request` vs `requesst`). Catches developers who mistype or copy-paste from an unreliable source.

**Account takeover.** A maintainer's account is compromised; a malicious version is published under the legitimate package name. Detection: unexpected version bumps, version notes that don't match the change, behaviour that differs from what the changelog promises.

**Malicious maintainer transfer.** A trusted maintainer hands the package to someone who turns out to be malicious (intentionally or via account compromise). The package keeps its name and reputation but starts behaving differently.

**Post-install scripts.** Many package managers run install scripts when a package is installed. Malicious install scripts can exfiltrate data, install backdoors, or modify other code. Disable post-install scripts where possible (`npm install --ignore-scripts` for production deploys; review carefully when needed).

**The xz-utils backdoor (March 2024)** demonstrated the most sophisticated form: a malicious actor gained maintainer status over years, then pushed a backdoor as part of a routine update. Discovered only because a researcher noticed unusual SSH performance.

**Mitigation:**
- Pin exact versions in production
- Review changelogs before upgrading critical dependencies
- Use a lockfile and commit it
- Restrict which package registries CI can install from (no random downloads)
- Monitor for unexpected dependency updates

---

## SBOM (Software Bill of Materials)

A Software Bill of Materials lists every component in a software artifact, including transitive dependencies, versions, and licences. SBOMs are increasingly required by:
- US Federal procurement (Executive Order 14028)
- EU Cyber Resilience Act
- Many enterprise customers as a compliance requirement

**Standard formats:** SPDX, CycloneDX. Most modern build tools can generate one (`syft`, `cyclonedx-bom`, native support in npm/Cargo).

**When to generate one:**
- Required by customer or regulatory contract
- Whenever a security incident occurs in a dependency, an SBOM tells you whether you ship the affected version
- For software shipped to environments outside your direct control

**The minimum:** generate an SBOM as part of every production build, archive it with the artifact. When CVE-2025-XXXXX is announced affecting library Y, you can answer "do we ship Y, and which version?" in seconds rather than hours.

---

## Update cadence

Dependencies need to be updated. The longer you wait, the more painful each update becomes — major version updates accumulate breaking changes; security patches require backporting that the maintainers don't do for old versions.

**Continuous low-risk updates.** Automated tools (Dependabot, Renovate) open PRs for patch and minor updates. With good test coverage, these can often be merged after CI passes. Critical: the updates must be tested in CI; auto-merging without tests is gambling.

**Scheduled major-version updates.** Major version bumps usually need manual review for breaking changes. Schedule a recurring time to address them rather than letting them pile up.

**Security updates immediately.** A high-severity CVE in a production dependency is treated like an incident. Update, test, deploy — measured in hours, not weeks.

**The signal of falling behind:** a routine security update requires updating 5 other things to get a compatible version because everything is so far behind. Now an hour of work is a week. Don't get into this state.
