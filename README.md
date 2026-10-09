# SprayScope

A browser based VPN authentication threat detection prototype. Analyze authentication events, investigate suspicious activity, and explore simulated VPN attack patterns through a security analyst dashboard.

> **Prototype notice:** SprayScope is a hackathon demo, not production identity or VPN infrastructure. All bundled activity is illustrative.

**Live demo:** [Open SprayScope on Vercel](https://sprayscope-hackathon-demo.vercel.app)

**Connect:** [Akshat Goyal on LinkedIn](https://www.linkedin.com/in/akshat-goyal-959081247)

[![Open the full size dashboard screenshot](./dashboard-overview.png)](./dashboard-overview.png)

*Select the dashboard image to open it at full size.*

## Explore

| Jump to | What you'll find |
| --- | --- |
| [Quick start](#quick-start) | Run the dashboard and API locally |
| [Dashboard](#dashboard) | Threat detections and analyst views |
| [Project layout](#project-layout) | Frontend, backend, and dataset folders |
| [Datasets](#datasets) | Example authentication logs and ML benchmark |
| [Database and deployment](#database-and-deployment-setup) | Configure PostgreSQL and the Vercel environment |
| [Limitations](#limitations) | Prototype scope and safe-use notes |
| [Project notes](#project-notes) | Architecture, handoff, and evaluator guides |

## Quick start

Requirements: Node.js 20 or newer and a PostgreSQL database.

Copy `.env.example` to `.env` and set `DATABASE_URL` to your PostgreSQL connection URL. The schema is created on first start. Keep `.env` out of Git.

```powershell
npm start
```

Open [http://localhost:4173](http://localhost:4173). The backend serves the dashboard and API from the same local server. Stop it with **Ctrl+C**.

- [Open the analyst dashboard](http://localhost:4173/)
- [Open the simulated VPN user portal](http://localhost:4173/?portal=1)
- [Check backend health](http://localhost:4173/api/health)

Create an analyst account from the sign-in screen. Passwords are scrypt-hashed on the server, and sign-in uses revocable HttpOnly cookies. Events and accounts are stored in PostgreSQL.

## Database and deployment setup

- Set `DATABASE_URL` in `.env` for local work and in the Vercel project’s Environment Variables for deployment. Use a hosted PostgreSQL provider such as Neon or another PostgreSQL service.
- The public Vercel demo link above still serves the previous release until this database variable is configured and the updated source is deployed.
- To create the first admin account, set `BOOTSTRAP_ADMIN_EMAIL` and `BOOTSTRAP_ADMIN_PASSWORD` before the first database initialization. Use a unique password with at least 12 characters. This account is created only when that email is not already present.
- The public VPN simulation portal can submit synthetic events. Reading the shared event feed and accessing the analyst dashboard require an account.
- Vercel Functions need the database configured before deployment. Event storage is persistent in PostgreSQL; the old in-memory demo deployment does not migrate its events or browser-local accounts.

## Checks

Run API and security tests with `npm test`, and JavaScript syntax checks with `npm run check`. GitHub Actions runs both on pushes to `main` and on pull requests.

## Tech stack

| Layer | Technologies |
| --- | --- |
| Frontend | HTML, CSS, vanilla JavaScript modules |
| Backend | Node.js 20+ with built-in HTTP and filesystem modules |
| Optional sign in | Microsoft MSAL Browser and Microsoft Entra ID |
| Offline benchmark | Python and scikit-learn |
| Input data | CSV and JSON authentication event files |

## Dashboard

SprayScope analyzes uploaded CSV or JSON authentication events and the bundled simulated stream. It highlights:

- Brute force: 10 failed events for one user in 5 minutes.
- Password spray: 10 users from one source in 10 minutes, with no more than 5 events per user on average.
- Impossible travel: country and centroid based travel estimates.
- Credential stuffing, abnormal login bursts, and account enumeration when rejection details identify unknown usernames.

The overview includes login and threat metrics, event activity, suspicious sources, and a “Why was this detected?” panel that explains rhythm, context, and shape. Confidence is a rule match strength indicator, not measured model accuracy.

The user portal generates simulated login telemetry for training the same detection flow. It does not authenticate a real account or connect to a VPN. Password values are never stored or sent to the API.

## Project layout

| Path | Purpose |
| --- | --- |
| [`frontend/`](frontend/) | Single page dashboard, styles, and browser side modules |
| [`backend/server.js`](backend/server.js) | Node API and static file server |
| [`dataset/`](dataset/) | Small CSV fixtures and optional offline ML training script |
| [`package.json`](package.json) | One command local launcher: `npm start` |

## Datasets

The Dataset page includes labeled fixtures for [brute force](dataset/brute-force-only.csv), [password spray](dataset/password-spray-only.csv), and [impossible travel](dataset/impossible-travel-only.csv), plus a [mixed authentication sample](dataset/demo-authentication.csv). Files are parsed in the browser for analysis.

The optional LANL benchmark script is at [`dataset/scripts/train_lanl.py`](dataset/scripts/train_lanl.py). Install its dependencies with:

```powershell
python -m pip install -r dataset/requirements-ml.txt
```

Place the LANL `auth.txt.gz` and `redteam.txt.gz` source files under `dataset/lanl/`, then run:

```powershell
python dataset/scripts/train_lanl.py
```

The benchmark writes its model and report under `dataset/models/lanl/`. LANL identities are anonymized and the source logs do not include IP addresses or countries, so benchmark metrics do not validate the dashboard's VPN or impossible travel detections.

## Microsoft sign in

Microsoft SSO is not currently connected to the backend session system. The app uses its server-managed email and password flow; do not enable browser-only Microsoft sign-in until the server verifies Microsoft identity tokens.

## Limitations

This prototype has no analyst account API, VPN control plane, TLS termination, or production grade access controls. Only use synthetic or appropriately sanitized authentication data. Add real identity and VPN integrations and deploy behind HTTPS with suitable access controls before handling real users or authentication traffic.

## Project notes

- [Project architecture, measured fixture metrics, and viva guide](PROJECT_VIVA_GUIDE.md)
- [Five person team handoff and API notes](TEAM_HANDOFF.md)
- [Project overview](SPRAYSCOPE_PROJECT_OVERVIEW.txt)
- [Evaluator questions](SPRAYSCOPE_EVALUATOR_QUESTIONS.txt)
- [What makes SprayScope unique](SPRAYSCOPE_WHATS_UNIQUE.txt)
- [Hackathon source and demo package](SPRAYSCOPE_HACKATHON_PACKAGE.zip)

[Back to top](#sprayscope)

