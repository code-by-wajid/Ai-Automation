# Day 04 — APIs & API Key Authentication

## Table of Contents

1. What an API actually is
2. Why APIs need authentication at all
3. What an API key actually is
4. How the key proved access today — 200 vs 401
5. Query param auth vs. header auth
6. What ".env" means and why it exists
7. Keeping your key private with a GitHub-only workflow
8. What happens if a key leaks — a real scenario
9. Glossary
10. Self-check questions

---

## 1. What an API actually is

**API** stands for **Application Programming Interface**. Strip away the
jargon and it means exactly this: a defined way for one piece of software
to ask another piece of software to do something, or hand over data —
without either side needing to know how the other is built internally.

When you called OpenWeatherMap's API today, you didn't need to know
anything about how their servers store weather data, what database they
use, or what language their backend is written in. You just needed to
know the "interface" — the specific URL, the parameters it accepts, and
the format it returns data in. That agreed-upon interface *is* the API.

Every automation you'll build going forward is, at its core, using APIs
to make different systems (n8n, an LLM, a client's CRM, your own scripts)
talk to each other without any of them needing to understand each other's
internals.

## 2. Why APIs need authentication at all

Most useful APIs cost the provider real money to run — server time,
processing power, sometimes third-party data licensing. Without
authentication, anyone could call the API unlimited times, for free,
anonymously. Authentication solves three problems at once:

- **Identity** — the provider knows *who* is making each request
- **Rate limiting** — the provider can cap how many requests *your*
account specifically can make (protecting their servers from abuse)
- **Billing** — for paid APIs, usage gets tracked and billed to the
correct account

This is exactly why your request without the key returned `401 Unauthorized` today — the server has no way to identify you as an
approved caller, so it refuses to process the request at all.

## 3. What an API key actually is

An API key is really just a long, unique, hard-to-guess string of
characters — functionally similar to a password, but usually not tied to
a login session the way a website password is. It's generated once by
the provider (OpenWeatherMap, in today's case) and tied permanently to
your account.

When you include it in a request, the server checks: "does this exact
string match a key I've issued to a real, registered account?" If yes,
it proceeds. If no (or missing), it responds with `401`.

Importantly: **a key isn't inherently more "secure" than a password just
because it looks like random characters.** Anyone who obtains your key
can use it exactly as if they were you — make requests under your
account, consume your rate limit, and if it's a paid API, run up your
bill. This is exactly why protecting it matters as much as protecting a
real password.

## 4. How the key proved access today — 200 vs 401

Your two tests today demonstrated the entire authentication concept in
the simplest possible way:

```
WITH the key:    ?q=Sialkot&appid=your-real-key   → 200 OK (data returned)
WITHOUT the key: ?q=Sialkot                       → 401 Unauthorized
```

Nothing else changed between the two requests — same URL, same city,
same method. The *only* difference was the presence of a valid key, and
that alone was the difference between being let in and being turned
away. This is authentication in its most stripped-down, visible form —
every more complex auth system you'll encounter later (OAuth, bearer
tokens, session cookies) is a more elaborate version of this exact same
underlying idea: prove who you are, then get let in.

## 5. Query param auth vs. header auth

You authenticated today by putting the key directly in the URL as a
query parameter (`&appid=...`). This is one valid method, but not the
only one, and not always the most secure:

- **Query parameter** (what you did) — simple, but the key ends up
visible in browser history, server logs, and anywhere the full URL
gets recorded or shared.
- **Header-based** (more common for production APIs) — the key is sent
in a request header (often `Authorization: Bearer YOUR_KEY`) instead of
the URL itself. This keeps it out of logs that record URLs, and is
generally considered the more secure convention.

OpenWeatherMap's free tier supports the simpler query-param method, which
is exactly why today's lesson could use it — but as you move to other
APIs later in this roadmap (especially AI provider APIs), expect
header-based auth to be the norm.

## 6. What ".env" means and why it exists

`.env` is just a plain text file, conventionally named exactly `.env`
(with the dot, no filename before it), used to store sensitive values —
API keys, passwords, database URLs — **separately from your actual code.**

A `.env` file typically looks like this:

```
OPENWEATHER_API_KEY=YOUR_API_KEY
```

Your code then reads this value at runtime, instead of having the key
typed directly into the script itself:

```javascript
require('dotenv').config();
const apiKey = process.env.OPENWEATHER_API_KEY;
```

**Why this separation matters:** if your key is typed directly inside
`fetch-weather.js`, then the moment that file gets committed to GitHub,
your key is now sitting in that file's history, visible to anyone who can
see the repo — permanently, even if you delete it later (removed files
still exist in git's history unless you take special action to purge
them). Keeping the key in `.env`, and telling git to *never track*
`.env` at all (via `.gitignore`), means the key simply never enters your
repository's history in the first place.

## 7. Keeping your key private with a GitHub-only workflow

Since you're managing this repo directly through GitHub.com rather than
running code locally, here's exactly how to keep this key safe in your
actual situation:

**Right now, at this stage of the roadmap:** you don't need to store the
key in the repo at all. You tested it in Postman, which runs entirely on
your own computer — nothing about that test needs to touch GitHub. The
correct move today is simply: **don't paste your real key into any file
you commit.** Your Day 4 notes file (below) should describe *what you
tested*, not include the actual key value.

**When you eventually write real scripts that need this key** (later
phases), and you're doing that work in Cursor locally, the `.env` +
`.gitignore` pattern above is exactly what protects you — the key lives
only on your computer, never in anything pushed to GitHub.

**If you ever need a script to use a secret while running *through*
GitHub itself** (for example, a GitHub Action that runs code
automatically), GitHub has a dedicated, built-in feature for this called
**GitHub Secrets** — found in your repo under **Settings → Secrets and
variables → Actions**. It lets you store a key encrypted, directly on
GitHub, without it ever appearing in your visible code. You won't need
this yet, but it's the correct tool for that specific future situation,
rather than ever pasting a real key into a regular file.

**The one absolute rule regardless of workflow:** if a real key ever
ends up pasted into a committed file, treat it as compromised —
regenerate a new key from the provider's dashboard immediately, rather
than just deleting the file (the old key remains valid, and visible in
history, until you explicitly revoke/regenerate it).

## 8. What happens if a key leaks — a real scenario

This isn't hypothetical caution — it's a well-documented, common incident
pattern: developers accidentally commit a real API key to a public
GitHub repo, automated bots that constantly scan public GitHub
repositories for exposed keys find it within minutes to hours, and the
key gets used for unauthorized access or, for paid APIs, to run up
massive unexpected bills on the original owner's account before they
even notice. This is precisely why the `.env` + `.gitignore` habit is
framed throughout this roadmap as non-negotiable, not optional
best practice — it's the single most common, most avoidable security
mistake in this entire field.

## 9. Glossary

- **API** — Application Programming Interface; a defined way for two
systems to exchange data or trigger actions without needing to know
each other's internals.
- **API key** — a unique string identifying and authenticating a specific
account/application to an API.
- **Authentication** — proving who is making a request.
- **Rate limit** — a cap on how many requests an account can make in a
given time window.
- **.env file** — a plain text file storing sensitive values separately
from code, so they're never hardcoded directly into scripts.
- **.gitignore** — a file listing what git should never track or commit
(including `.env`), keeping secrets out of your repository entirely.
- **GitHub Secrets** — GitHub's built-in, encrypted way to store sensitive
values for use specifically within GitHub Actions.



## 10. Self-check questions

1. What's the actual difference between an API key existing and an API
  key being *valid* for your specific account?
2. Why is putting a key in the URL (query parameter) slightly less secure
  than putting it in a header?
3. If you accidentally commit a real key to GitHub and then delete the
  file in a later commit, is the key actually safe again? Why or why
   not?
4. What's the one thing `.gitignore` needs to list to make sure `.env`
  never gets committed?
5. In your current GitHub-only workflow, where does today's real API key
  actually need to be stored — and where should it definitely NOT
   appear?

---

*Tests run in Postman: same URL with a valid key → 200 OK; same URL
without the key → 401 Unauthorized. Real key value intentionally not
included in this file or committed anywhere in this repo.*
