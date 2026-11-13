# HANDOFF-0004 — OBM Website public launch with isolated DB

Date: 2026-11-12 (America/New_York)
Status: OWNER APPROVED; SOURCE, DNS, DB AND DEPLOYMENT NOT YET VERIFIED.

## Confirmed decisions
- Publish the customer-facing OBM corporate site publicly at **https://www.nailssalonsoftware.com/**, accessible by anyone with the URL so visitors can leave feedback.
- Focus first on Home, OBM Apps, Benefits, Pricing, Compatible Hardware, Partners, Contact/Demo, Give Feedback. Internal admin/CMS/dashboard, checkout and advanced orders are deferred.
- Corporate site DB must be **separate from every tenant/API/POS production DB**. Proposed PostgreSQL DB `obm_website_pg`, least-privilege runtime role `obm_website_app` (verify against existing canonical configuration).
- Keep prior *temporary* public-site-only `noindex, nofollow` in both HTTP X-Robots-Tag header and server-rendered HTML meta tag. Public sharing works; noindex is not access control. Do not set across wildcard tenant hosts. Remove only when owner approves SEO.
- Prices, subscription terms, devices, compatibility, dealer terms and sales guarantees remain owner-pending. Do not invent figures or policies.

## Codex execution sequence
1. Identify the actual active Windows workspace and the specific corporate marketing website. Read applicable AGENTS.md, existing plan/handoff, framework/deploy docs. Audit Git root, branch, HEAD, dirty changes, remotes without modifications. Historical F:\Clean_Project is a clue; `6Website/OBM.TenantWeb.Mvc` was tenant GuestWeb, not an established corporate site.
2. Read-only verify www/apex DNS, existing destinations, TLS, Nginx/hosting, current production isolation, PostgreSQL roles/ACLs and migrations. Never blindly overwrite an existing www route, alter wildcard routing or production DB grants.
3. Build working mobile-friendly customer pages. Make a real **Give Feedback** form with server validation, rate limiting, anti-spam, privacy notice and persistent writes to the dedicated DB. Anonymous public read/list of submissions is forbidden. Demo/hardware/partner CTAs must have real verified destinations.
4. Provision isolated website DB and scoped runtime credentials; distinct migration owner where appropriate, reliable backups and no cross-DB access. Check effective privileges including PUBLIC CONNECT. If shared DB cluster cannot safely isolate without production ACL changes, use separate cluster/instance rather than touching production.
5. Local build/test with synthetic data, then deploy isolated corporate app at **www.nailssalonsoftware.com** (HTTPS). Verify actual externally visible HTTP status, header+raw HTML noindex on homepage and key routes, feedback submission persist, responsiveness, all links, and unchanged tenant site indexing.
6. Report exact source path, Git status, changed files, DB/migration facts, HTTPS URL, artifact/commit/hash, test evidence, deployment time, rollback method and any blockers. Never claim deployment without proof.

## Safety
No Royal Nail Spa testing. No POS, tenant GuestWeb, production API/DB, payment or other shared-service alterations. Preserve dirty source. No reset/clean/stash/force push. No secrets, full proprietary source or customer data in this **public** coordination repo.

## Handoff location
This is a sanitized coordination copy for Codex; the actual canonical `AGENTS.md`, website plan and working handoff belong in the verified **local source** and should be updated in place, without duplicate/conflicting instructions.
