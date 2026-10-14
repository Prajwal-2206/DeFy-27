# security.md - Hackathon Website Hardening Checklist

Working checklist for the tech team. Scope: the hackathon's public website (registration, tracks, prizes, submissions, judging, leaderboard). Not a finished policy doc - check items off as they're implemented and link the PR/commit that did it.

---

## 1. Auth & Access Control

- [ ] Rate limiting on every auth endpoint individually (login, signup, OTP request, OTP verify, password reset) - not just a global rate limiter on `/api/*`
- [ ] Lockout/backoff after N failed login attempts, with CAPTCHA fallback (hCaptcha or Cloudflare Turnstile) after the threshold
- [ ] JWT/session tokens: short access-token expiry (~15 min) + refresh token rotation
- [ ] Tokens stored in httpOnly + secure + sameSite=strict cookies - never localStorage/sessionStorage
- [ ] Role-based access control (participant / organizer / admin) enforced server-side on every protected route, not just hidden in the UI
- [ ] Admin/organizer routes return 403/404 to unauthorized users even if they guess the URL
- [ ] CSRF tokens on all state-changing requests if using cookie-based sessions

## 2. Input & Data Handling

- [ ] Server-side validation on every field, even ones already validated client-side (assume client is bypassed)
- [ ] Parameterized queries / ORM only - zero raw string-concatenated SQL
- [ ] File upload handling (team docs, project proof, avatars):
  - [ ] File type whitelist (check actual bytes/magic number, not just extension or `Content-Type` header)
  - [ ] Size cap enforced server-side
  - [ ] Images re-encoded on upload (strips embedded exploits/metadata)
  - [ ] Stored outside the web root or on S3/R2 with short-lived signed URLs
- [ ] Output encoding / sanitization anywhere user input is rendered back (team names, project descriptions, bios) - XSS prevention
- [ ] Reject/escape anything that looks like a template injection or SSTI payload in free-text fields

## 3. Infra & Secrets

- [ ] No secrets committed to the repo, ever - `.env` in `.gitignore` from commit #1
- [ ] If a secret ever touched git history (even once, even deleted later), rotate it - history isn't erased by a force-push
- [ ] Secrets live in platform env vars or a secrets manager, not in code or Slack/Discord messages
- [ ] HTTPS enforced site-wide, HSTS header enabled
- [ ] Security headers set: CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy (cheap, ~10 min of work, do this early)
- [ ] Dependency scanning on (`npm audit` / Dependabot / Snyk) - catch known CVEs before launch
- [ ] Separate staging and prod environments; prod secrets never used in staging

## 4. Hackathon-Specific (highest priority - this is where real prize money is visible)

- [ ] Submission endpoints locked down and tested against tampering (can't submit as another team, can't edit after deadline via API even if UI hides the button)
- [ ] Idempotency on submission/registration endpoints - no double-submit races
- [ ] Judging/scoring endpoints restricted to authenticated judges only, server-side checked
- [ ] All admin actions (declaring winners, editing prize pools, adding/removing tracks) logged with who + when - audit trail
- [ ] Prize pool and winner data can't be mutated by a participant-level token under any circumstance (test this explicitly, don't assume)
- [ ] Deadline enforcement happens server-side on a trusted clock, not client-side `Date.now()`

## 5. Traffic & Availability (the "no 404 under load" half)

- [ ] Load test before the event (k6 / Artillery) against realistic peak - e.g. 500+ concurrent hits when a track/prize reveal or leaderboard update goes live
- [ ] App servers stateless (session state in Redis/DB, not in-process memory) so you can scale horizontally without breaking sessions
- [ ] CDN in front of static assets (Cloudflare / Vercel edge) to cut origin load
- [ ] Caching with short TTL on read-heavy, rarely-changing endpoints (tracks, prize pools, leaderboard), invalidated on admin edit
- [ ] DB connection pool sized and tested - the #1 cause of random 500s under concurrent load
- [ ] Graceful degradation: serve cached/stale data instead of erroring if DB is slow - a stale leaderboard beats a 500 page
- [ ] Documented scale-up step (manual or auto) for whatever PaaS you're on - know what button to press when traffic spikes
- [ ] Health check endpoint + uptime monitoring (UptimeRobot / BetterStack) so the team knows before participants do
- [ ] Heavy/slow operations (file processing, emails, notifications) go through a queue, not inline in the request

---

## Pre-event sign-off

- [ ] Full checklist reviewed by at least 2 tech team members
- [ ] Load test results attached/linked here
- [ ] Final secrets rotation done within 48h of event start
