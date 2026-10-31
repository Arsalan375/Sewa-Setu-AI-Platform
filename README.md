Sewa Setu — Old Age Pension Portal with Official AI Agent Channels
A rebuild of the Purvanchal Old Age Pension portal (Sewa Setu) as a full-stack web application with two separately-scoped MCP servers so AI assistants can act officially for citizens and block officers — under the portal's own rules, with every decision audited.

Live app: https://sewasetu-assistant.lovable.app(opens in a new tab) Stack: TanStack Start (React 19, server functions) · Tailwind v4 · Lovable Cloud (Postgres, storage, sessions) · hand-rolled OAuth 2.1 + stateless MCP JSON-RPC over streamable HTTP.

The landing page doubles as an interactive showcase: animated register counters, a live JSON inspector of the scheme rules and the real MCP tool lists, tabbed explainers per audience, and a plain-words rules accordion.

What it does
Surface	Path	Who
Scheme landing	/	Public
Scheme information	/about	Public
Apply wizard (OTP → details → proof → declare → submit)	/apply	Citizens
Status portal (track, withdraw, re-upload proof, end assistant sign-ins)	/status	Citizens
SMS gateway console (simulated carrier inbox)	/sms-gateway	Testers
Officer dashboard, queue, case files, program tokens	/admin, /admin/dashboard, /admin/applications, /admin/cases/:id, /admin/tokens	Officers
Agent evaluation dashboard (database-checked evals)	/eval-dashboard	Reviewers
OAuth sign-in + consent for assistants	/oauth/authorize	Citizens & officers
Scheme rules enforced server-side (single source of truth: src/lib/domain.server.ts)
Benefit Rs 250/month; applicants must be 60+ (age computed on the IST clock).
Application window (reopened for the pilot to 31 March 2027 IST).
Required fields + age-proof document (JPG/PNG/PDF, ≤ 5 MB) uploaded at the portal.
One active application per mobile number (PENDING / APPROVED / DEEMED_APPROVED).
OTP citizen sign-in; status-portal password is DOB as DDMMYYYY.
Officer approve/reject with a reason recorded against the officer's name in the audit trail.
15-day SLA: pending applications are deemed approved by law (LAW (15-day SLA)).
Rejection requires a reason — enforced in the officer console, the officer MCP, and the database path.
MCP servers (streamable HTTP, bearer-token authenticated)
Endpoint	Scope	Audience check
/api/public/mcp/applicant	citizen	/api/public/mcp/applicant
/api/public/mcp/officer	officer	/api/public/mcp/officer
Discovery: /.well-known/oauth-authorization-server, /.well-known/oauth-protected-resource, /.well-known/jwks.json.
OAuth 2.1: dynamic client registration, PKCE (S256), consent page at /oauth/authorize, token refresh + rotation, revocation; RS256 access tokens.
Citizen tools: get_scheme_info, check_eligibility, get_my_applications, get_draft, save_draft, submit_application, withdraw_application.
Officer tools: get_queue, get_application, approve_application, reject_application, get_queue_stats.
A citizen token is refused (HTTP 403 insufficient_scope) at the officer endpoint and vice versa.
Unattended officer use is supported by revocable program tokens issued at /admin/tokens.
Evaluation (/eval-dashboard)
The dashboard runs a live, database-checked suite (src/lib/evals.server.ts): every check replays a real capability — OTP flow, OAuth consent, both MCP servers, portal endpoints — then asserts the outcome directly in Postgres (14 checks across citizen, officer, and statutory-rule areas). Fixtures use fabricated mobile numbers (9999xxxxxx); the 100,000 imported applications are never modified. Detailed personas are shown on the dashboard itself.

Data
seed/ in the original handout contains a 100,000-application production snapshot (six blocks). It was imported verbatim; current mix: ~82.9k approved, 9k deemed approved, 7k rejected, 700 pending, 400 withdrawn.

Access
Officer login: admin / sewasetu@2026 (seeded departmental account).
Citizen sign-in: mobile number + OTP (OTPs visible in the simulated carrier at /sms-gateway).
Notes
The telecom carrier is simulated: SMS rows are "delivered" after 1–4 s and appear at /sms-gateway and /api/public/gateway/messages.
Age proofs are stored in a private storage bucket (age-proofs); only the path is recorded in the database.
