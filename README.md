# Nigeria Electronic Voting System

A secure, full-stack electronic voting (e-voting) web application built for a Nigerian
elections context. It demonstrates voter authentication, an administrator dashboard,
live results with charts, and security/verification features (vote receipts and a
tamper-evident hash chain).

**Stack:** Node.js · Express · SQLite (`better-sqlite3`) · JWT · bcrypt · vanilla JS + Chart.js

---

## 1. Project files

Put all of these files together in **one folder** (e.g. `nigeria-evoting/`):

```
nigeria-evoting/
├── package.json        # dependencies & npm scripts
├── .env.example        # copy to .env and set secrets
├── db.js               # database connection + schema
├── seed.js             # creates admin, sample voters, sample election
├── server.js           # Express API + serves the frontend
├── index.html          # login / registration page
├── voter.html          # voter dashboard (cast vote + verify receipt)
├── admin.html          # admin dashboard (manage elections, audit, integrity)
├── results.html        # public live results with charts
├── styles.css          # shared styling
└── README.md
```

> The files are delivered flat. Just create a folder and move them all into it before running.

---

## 2. Setup & run

You need **Node.js 18+** installed.

```bash
# 1. Install dependencies
npm install

# 2. (Recommended) create your secrets file
cp .env.example .env        # Windows: copy .env.example .env

# 3. Seed the database with demo data
npm run seed

# 4. Start the server
npm start
```

Then open **http://localhost:3000**

To wipe and reseed at any time: `npm run reset`

### Demo accounts
| Role  | Login | Password |
|-------|-------|----------|
| Admin | `admin` | `admin123` |
| Voter | `VIN0000001` … `VIN0000005` | `voter123` |

A sample **Presidential Election 2027 (Demo)** is created and already **open** with four candidates.

---

## 3. How to use it

**As a voter** — log in on the *Voter* tab, pick a candidate in an open election, and
cast your vote. You receive a **receipt code**; save it. You can paste it into the
"Verify a vote receipt" box to confirm your vote was recorded (it never reveals who you
voted for). You cannot vote twice in the same election.

**As an admin** — log in on the *Admin* tab to:
- see live stats (registered voters, votes cast, turnout);
- create a new election (starts as *draft*);
- add/remove candidates while it is a draft;
- **open** an election for voting, then **close** it;
- view the **audit log** of every action;
- run an **integrity check** on the vote chain.

**Anyone** can open the *Live Results* page to watch tallies update (auto-refreshes every 5s).

---

## 4. Security & integrity features

- **Password hashing** — bcrypt (cost 12); plaintext passwords are never stored.
- **Token sessions** — JWT, signed server-side, expire after 2 hours; role-based access
  (voter vs admin) enforced on every protected route.
- **One vote per voter per election** — enforced by a database `UNIQUE` constraint *and*
  re-checked inside a transaction, so it cannot be bypassed by race conditions.
- **Ballot secrecy** — a vote stores a salted SHA-256 hash of the voter's identity, not
  the voter id, so a recorded vote cannot be trivially traced back to a person.
- **Vote receipt** — each voter gets a unique receipt to independently verify their vote
  was counted, without exposing their choice.
- **Tamper-evident hash chain** — every vote is linked to the previous one
  (`prev_hash → vote_hash`, SHA-256), like a lightweight blockchain. The admin
  *Integrity Check* recomputes the whole chain and flags the exact vote where any
  tampering occurred.
- **Audit log** — registrations, logins, votes, and all admin actions are recorded.
- **Hardening** — `helmet` security headers, rate limiting on auth and voting endpoints,
  request size limits, and input validation.

---

## 5. API reference (summary)

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| POST | `/api/auth/register` | – | Voter self-registration |
| POST | `/api/auth/login` | – | Voter login |
| POST | `/api/auth/admin/login` | – | Admin login |
| GET  | `/api/elections` | voter | Open elections + candidates + voted flag |
| POST | `/api/elections/:id/vote` | voter | Cast a vote, returns receipt |
| GET  | `/api/verify/:receipt` | – | Verify a receipt (no choice revealed) |
| GET  | `/api/results` | – | List elections |
| GET  | `/api/results/:id` | – | Live tally for an election |
| GET  | `/api/admin/stats` | admin | Dashboard statistics |
| GET  | `/api/admin/elections` | admin | All elections with counts |
| POST | `/api/admin/elections` | admin | Create election (draft) |
| POST | `/api/admin/elections/:id/status` | admin | Open / close / draft |
| POST | `/api/admin/elections/:id/candidates` | admin | Add candidate (draft only) |
| DELETE | `/api/admin/candidates/:id` | admin | Remove candidate (draft only) |
| GET  | `/api/admin/audit` | admin | Audit log |
| GET  | `/api/admin/integrity` | admin | Verify vote hash chain |

---

## 6. Notes for your write-up / defense

This implementation is a **demonstration prototype**, suitable for a final-year project.
For a production national system you would additionally need: HTTPS/TLS everywhere,
integration with a real voter register / biometric verification (e.g. INEC's BVAS),
a hardened production database (PostgreSQL), independent end-to-end verifiability
(e.g. published cryptographic proofs), professional security auditing, and a legal /
regulatory framework. The hash-chain and receipt mechanisms here illustrate the core
ideas of *verifiability* and *integrity* that such systems rely on.

---

## 7. Troubleshooting

- **`better-sqlite3` install error** — ensure Node.js 18+; on Windows you may need the
  build tools (`npm install --global windows-build-tools`) only if a prebuilt binary
  isn't downloaded automatically. Re-run `npm install`.
- **Port already in use** — change `PORT` in `.env`.
- **Want a clean slate** — `npm run reset`.
