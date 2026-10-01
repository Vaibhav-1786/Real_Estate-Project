# Security Policy

Iconic Estates India stores customer leads, contact details and uploaded documents (e.g. KYC, agreements). Security reports are taken seriously.

## Supported versions

Only the latest commit on the `main` branch receives security fixes.

## Reporting a vulnerability

**Please do not open a public issue for security problems.**

1. Use GitHub's private reporting: **Security → Report a vulnerability** on this repository (preferred), or
2. Email the maintainer, Vaibhav Chauhan, at `<your-email@example.com>` *(replace before publishing)*.

Please include a description, impact, steps to reproduce (endpoint, role, request/response) and a suggested fix if you have one. You can expect an acknowledgement within **7 days** and a status update within **14 days**. Please allow reasonable time for a fix before public disclosure.

### In scope
Admin/agent/customer privilege escalation, customer-portal OTP bypass, access to another customer's leads, messages or documents, JWT flaws, injection, insecure file upload/download, secrets exposure.

### Out of scope
Issues that exist only because development defaults were left in production (see checklist), denial of service by volume, social engineering, and flaws in third-party services (SMTP providers, hosting).

## Security measures in the project

- Admin authentication with email + password (bcryptjs) and JWT; role-based access (`super_admin`, `admin`, `agent`).
- Separate JWT flow for the customer portal (mobile OTP); customers can only access records tied to their own verified mobile number.
- OTPs are 6 digits and valid for 10 minutes.
- Helmet security headers and `express-rate-limit` (general API: 300 requests / 15 min; login: 10 requests / 10 min).
- `.env` files are gitignored.
- Least-privilege MySQL user recommended for production.

## Production deployment checklist

The repository ships with **development defaults**. Before going live:

- [ ] **Fix CORS**: `backend-node/server.js` currently uses `origin: true` (any origin, with credentials). Replace it with an allowlist from `ALLOWED_ORIGINS`, and set `ALLOWED_ORIGINS` on the Python service too.
- [ ] Set strong, **different** values for `JWT_SECRET` and `CUSTOMER_JWT_SECRET` (the customer secret falls back to `JWT_SECRET` if unset).
- [ ] **Configure SMTP.** Without it, `request-otp` returns the OTP in the API response — an authentication bypass in production. Also gate the `dev_otp` field behind `NODE_ENV !== 'production'`.
- [ ] Set `NODE_ENV=production`.
- [ ] Change the bootstrap `ADMIN_PASSWORD` right after first login and remove or rotate it from `.env`.
- [ ] Review upload validation in `backend-node/middleware/upload.js` (file type and size) — uploaded files are served statically from `/uploads`.
- [ ] Use a dedicated MySQL user instead of `root`.
- [ ] Serve everything over HTTPS; run the Node API under PM2/systemd/Docker and the FastAPI service under a production ASGI setup.
- [ ] Review rate-limit thresholds for your traffic.
- [ ] Check git history to confirm no `.env` file was ever committed; rotate secrets if it was.
- [ ] Protect customer data according to applicable law (e.g. India's DPDP Act 2023).

## Known limitations

- No automated test suite is documented yet.
- The analytics service is called directly from the browser, so it must have its own CORS configuration.
