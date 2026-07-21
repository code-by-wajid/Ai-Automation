# Day 02 — HTTP Fundamentals (Comprehensive Notes)

## Table of Contents

1. What HTTP actually is
2. The client-server model, precisely
3. Anatomy of an HTTP request — every part explained
4. Anatomy of an HTTP response — every part explained
5. HTTP methods — all the ones that matter, explained
6. HTTP status codes — the full reference you'll actually use
7. Headers — what they are and why they matter
8. The request/response cycle, end to end
9. GET vs POST — the real distinction
10. Idempotency — a concept worth knowing early
11. HTTP in AI automation — comprehensively
12. What Postman is actually doing behind the UI
13. Common HTTP mistakes and how to catch them
14. My test results, annotated
15. Glossary — every term in one place
16. Self-check questions

---

## 1. What HTTP actually is

HTTP stands for **HyperText Transfer Protocol**. It is the agreed-upon set
of rules that governs how two systems — a client and a server — talk to
each other over a network, most commonly the internet.

The key word is **protocol**: a protocol is just a shared set of rules
both sides agree to follow, so that a message sent by one system can be
correctly understood by the other, regardless of what programming
language, operating system, or platform either side is built on. HTTP is
what makes it possible for a Postman request built by you, a browser
request built by Chrome, and an n8n workflow node — three completely
different pieces of software — to all successfully talk to the exact same
API in the exact same way.

Without a shared protocol, every API would need custom, incompatible ways
of communicating with every different type of client. HTTP is the
universal agreement that eliminates that problem.

---

## 2. The client-server model, precisely

Every HTTP interaction involves exactly two roles:

- **Client** — the system that *initiates* the request. This can be a
  browser, Postman, a mobile app, an n8n workflow, another server, or
  literally any piece of software capable of sending an HTTP request.
- **Server** — the system that *receives* the request, processes it, and
  sends back a response.

A critical, easy-to-miss point from today: **"client" does not mean
"browser."** A browser is just one example of a client. When your n8n
workflow calls an API, n8n is acting as the client in that exact moment.
When one of your future AI agents calls a tool, your own code is the
client. The client/server relationship is a *role*, not a specific piece
of software — and a single system can be a server in one interaction and
a client in another, simultaneously, depending on which direction the
request is flowing.

---

## 3. Anatomy of an HTTP request — every part explained

Every HTTP request, no matter what tool sends it, is built from the same
four components:

### 3.1 Method
A word describing *what kind of action* you want the server to perform
(GET, POST, PUT, DELETE, etc. — see Section 5 for the full breakdown).

### 3.2 URL (the endpoint)
The specific address identifying *what resource* you're interacting with.

```
https://jsonplaceholder.typicode.com/users
└─────────────┬────────────────────┘└──┬──┘
          the domain               the specific
      (which server to reach)     resource/endpoint
```

### 3.3 Headers
Metadata about the request itself — information *about* the request,
separate from the actual data being sent. Covered fully in Section 7.

### 3.4 Body (optional)
The actual data being sent *to* the server — only present on methods like
POST or PUT, where you're sending new information. A GET request
typically has no body, because you're only asking for data, not
submitting any.

**Putting it together** — the POST request you sent today, broken into
its four parts:

```
METHOD:  POST
URL:     https://jsonplaceholder.typicode.com/posts
HEADERS: Content-Type: application/json   (tells the server "the body below is JSON")
BODY:    {"title": "Test Post", "body": "This is a test", "userId": 1}
```

---

## 4. Anatomy of an HTTP response — every part explained

The server's response mirrors the request's structure, with one addition:

### 4.1 Status code
A 3-digit number indicating the outcome of the request (full reference in
Section 6).

### 4.2 Headers
Metadata about the *response* — e.g., what format the body is in, how
large it is, caching rules.

### 4.3 Body
The actual data being returned — in modern APIs, almost always JSON (this
directly connects to everything from Day 1).

**Your GET response today, broken down:**

```
STATUS:  200 OK
HEADERS: Content-Type: application/json; charset=utf-8   (the body is JSON text)
BODY:    [{"id": 1, "name": "Leanne Graham", ...}, {"id": 2, ...}, ...]
```

---

## 5. HTTP methods — all the ones that matter, explained

| Method | Purpose | Has a body? | Example use |
|---|---|---|---|
| **GET** | Retrieve data — asking, never changing anything | No | Fetching a list of users |
| **POST** | Create new data | Yes | Submitting a new form entry |
| **PUT** | Replace an existing resource entirely | Yes | Overwriting a full record |
| **PATCH** | Partially update an existing resource | Yes | Updating just one field |
| **DELETE** | Remove a resource | Usually no | Deleting a record |

For the next stretch of this roadmap (webhooks, n8n, most simple
automations), **GET and POST cover the overwhelming majority of what
you'll actually use.** PUT, PATCH, and DELETE become more relevant once
you're building automations that manage existing records, which comes
later.

**The conceptual rule that matters most:** GET should never change
anything on the server — it is purely a request for information. This is
why GET requests typically don't need a body: you're not sending data to
be stored, you're only asking to receive data.

---

## 6. HTTP status codes — the full reference you'll actually use

Status codes are grouped into five categories by their first digit. You
don't need to memorize every individual code today, but you should
recognize the category meaning on sight, plus the handful of specific
codes that come up constantly.

### 1xx — Informational
Rare in everyday API work; indicates the request was received and
processing continues. You will rarely encounter these directly.

### 2xx — Success
The request worked as intended.
- **200 OK** — standard success response (what you saw on your GET)
- **201 Created** — a new resource was successfully created (what you saw
  on your POST — note the new `id` field the server added)
- **204 No Content** — success, but there's nothing to send back (common
  after a successful DELETE)

### 3xx — Redirection
The resource has moved, and the client should look elsewhere.
- **301 Moved Permanently** — this URL has permanently changed; use the
  new one
- **304 Not Modified** — used for caching; "nothing has changed since you
  last asked"

### 4xx — Client Error
Something is wrong with *the request you sent* — not the server's fault.
- **400 Bad Request** — the request is malformed somehow (often: invalid
  JSON body, missing required field)
- **401 Unauthorized** — you didn't authenticate, or your credentials are
  missing/invalid (this becomes very relevant on Day 4 — API keys)
- **403 Forbidden** — you *are* authenticated, but you don't have
  permission for this specific action (different from 401 — this is "I
  know who you are, but you're not allowed")
- **404 Not Found** — the URL/resource doesn't exist (what you tested
  today)
- **429 Too Many Requests** — you've hit a rate limit; the server is
  telling you to slow down

### 5xx — Server Error
Something is broken on *the server's* side — not your fault as the
client.
- **500 Internal Server Error** — a generic "something broke" on the
  server
- **502 Bad Gateway** — a server acting as a relay got an invalid
  response from an upstream server
- **503 Service Unavailable** — the server is temporarily overloaded or
  down for maintenance

**Why this categorization matters practically:** when an automation
fails, the *first* diagnostic question is "is this a 4xx or a 5xx?"
because it immediately tells you where to look. A 4xx means you need to
fix something in what you're sending (your request, your auth, your
data). A 5xx means the problem is on the other system's end, and no
amount of changing your own request will fix it — you may just need to
retry later, which is exactly why retry logic (covered later in your
roadmap) specifically targets 5xx errors and rate-limit responses, not
4xx ones.

---

## 7. Headers — what they are and why they matter

Headers are key-value pairs sent alongside a request or response,
carrying *metadata* — information *about* the message, rather than the
actual content itself. A few you will see constantly:

- **Content-Type** — tells the receiving side what format the body is in
  (e.g., `application/json` tells the server "parse this body as JSON").
  This is exactly why, in Postman, you had to select **raw → JSON** for
  your POST body — that setting is what sets this header correctly behind
  the scenes.
- **Authorization** — carries credentials (an API key, a token) proving
  who is making the request. This becomes directly relevant on Day 4.
- **Accept** — tells the server what format the *client* wants the
  response in (less commonly needed to set manually, since JSON is the
  default almost everywhere now).
- **User-Agent** — identifies what software is making the request (a
  browser, Postman, a custom script).

A useful mental model: **the body is the letter; the headers are the
information written on the envelope.** The envelope information helps the
receiving system correctly process the letter inside, without the
receiving system needing to open and read the whole letter first just to
know basic facts about it.

---

## 8. The request/response cycle, end to end

Putting every piece together, here is the complete cycle that happened
each time you clicked Send in Postman today:

```
1. You (the client) build a request:
   method + URL + headers + (optional) body

2. Postman sends it over the network to the server

3. The server receives the request, reads the method and URL to
   determine what action is being asked for

4. The server processes the request (looks up data, saves data, etc.)

5. The server builds a response: status code + headers + body

6. The response travels back over the network to Postman

7. Postman displays it to you — status code, headers tab, body panel
```

This exact cycle — every single time, regardless of tool — is what's
happening underneath n8n's HTTP Request node, underneath every webhook
call, underneath every AI API call you'll make later in this roadmap.
Nothing about it changes once you move to a "no-code" platform; the
platform is simply building and displaying this cycle for you instead of
you typing it manually.

---

## 9. GET vs POST — the real distinction

The difference is not primarily "GET is for reading, POST is for
writing" as a rule to memorize — it's rooted in a deeper principle: **GET
requests should be safe to repeat without consequence; POST requests
usually are not.**

- Refreshing a GET request (like loading a webpage) doesn't create
  duplicate data — you're just asking for the same information again.
- Repeating a POST request (like resubmitting a form) can create
  duplicate records — you're asking the server to do something new each
  time.

This is why, in Postman, GET requests don't have a body — you're not
submitting anything to be created or changed, only requesting existing
data — while POST requests carry a body containing the new data to be
created.

---

## 10. Idempotency — a concept worth knowing early

**Idempotent** means: performing the same operation multiple times
produces the same result as performing it once. This is a real
engineering term you'll encounter in API documentation, and understanding
it early will save confusion later.

- **GET is idempotent** — asking for the same data 5 times doesn't change
  anything; you get the same result each time.
- **PUT is idempotent** — replacing a resource with the exact same data 5
  times results in the same final state as doing it once.
- **POST is NOT idempotent** — sending the same "create a new post"
  request 5 times typically creates 5 separate new records (5 different
  IDs), not one.

This matters directly for automation reliability: if a webhook fires
twice due to a network hiccup, a non-idempotent POST-based automation
might create duplicate records, while a well-designed idempotent
operation would safely produce the same end result either way. This
concept resurfaces later in your roadmap when building retry logic —
retrying a failed POST safely often requires extra design (like an
idempotency key) that a simple GET retry never needs to worry about.

---

## 11. HTTP in AI automation — comprehensively

### 11.1 Every AI API call is an HTTP POST request

When you call an LLM API (OpenAI, Groq, Gemini, Claude), you are sending
an HTTP **POST** request — the method is POST because you are asking the
server to *create* something (a new response), not just retrieve
existing data. The request body contains your prompt and parameters; the
response body contains the model's generated text. Every single AI
feature you build for the rest of this roadmap is, underneath, exactly
the request/response cycle from Section 8 — nothing more mysterious than
that.

### 11.2 Status codes tell you exactly what went wrong with an AI call
- A **401** from an LLM API means your API key is missing or invalid
  (directly relevant to Day 4).
- A **429** means you've hit a rate limit — you're sending requests
  faster than the provider allows, and your code needs to slow down or
  implement retry-with-backoff logic (covered later in Phase 4).
- A **500** or **503** means the AI provider's own servers are having
  issues — not something fixable by changing your request; only a retry
  later will help.

Being able to read a status code correctly is the difference between
correctly diagnosing "my API key is wrong" (401) versus "their servers
are down right now, try again shortly" (503) — two completely different
fixes that look identical if you never learn to check the status code.

### 11.3 Webhooks are just inverted HTTP requests
A webhook (which you'll build in n8n starting Day 7) is fundamentally an
HTTP **POST** request — the only conceptual difference from what you did
today is *direction*: instead of you deciding when to send a request
*to* an API, some external event (a form submission, a new lead)
triggers another system to send a POST request *to your* automation.
Same method, same anatomy (headers, body) — just initiated by someone
else's event instead of your own click of "Send."

### 11.4 Content-Type headers matter enormously with AI APIs
Nearly every AI API requires the `Content-Type: application/json` header
to correctly interpret your prompt and parameters as JSON rather than
plain text. Missing or incorrect headers are a very common source of
confusing 400 errors when first integrating a new AI API — exactly the
header concept from Section 7, applied directly.

### 11.5 n8n's HTTP Request node is doing precisely what you did in Postman
When you eventually wire an LLM call into an n8n workflow (Day 11 of your
roadmap), you'll be filling in the exact same four components — method,
URL, headers, body — inside n8n's visual node instead of Postman's
interface. Nothing conceptually changes; only the interface wrapping it
does.

---

## 12. What Postman is actually doing behind the UI

It's worth being explicit about this: Postman has no special, secret
power. When you click **Send**, Postman is simply constructing an actual
HTTP request (method + URL + headers + body) using its own internal code,
sending it over the network exactly like any script or app would, and
then displaying the raw response back to you in a readable interface.

This is precisely why the skills from today transfer directly to writing
actual request-sending code later (Day 23's Python glue scripts, Day 31's
Express middleware) — you are not learning "how Postman works," you are
learning how HTTP itself works, using Postman purely as a friendly window
into a process that's identical everywhere else.

---

## 13. Common HTTP mistakes and how to catch them

| Mistake | Symptom | How to catch it |
|---|---|---|
| Wrong method (GET instead of POST, or vice versa) | Unexpected 404 or 405 (Method Not Allowed) | Check the API docs for the exact required method per endpoint |
| Missing `Content-Type: application/json` header | Server can't parse your body correctly, often a 400 | Explicitly set the header, or confirm your tool sets it automatically for JSON bodies |
| Sending invalid JSON in the body | 400 Bad Request | Validate your JSON (Day 1 skill) before sending |
| Typo in the URL | 404 Not Found | Double-check the exact endpoint path against documentation |
| Missing authentication | 401 Unauthorized | Confirm the required header/param (relevant from Day 4 onward) is present |
| Assuming a 5xx is your fault | Wasted time changing your request when the issue is server-side | Recognize 5xx as "their problem," and consider retry logic instead |

---

## 14. My test results, annotated

**GET https://jsonplaceholder.typicode.com/users**
- Status: `200 OK`
- Result: returned a full array of user objects — confirms a successful,
  read-only request returning existing data.

**GET https://jsonplaceholder.typicode.com/nonexistent**
- Status: `404 Not Found`
- Result: confirms what a broken/incorrect URL looks like from the
  server's perspective — the resource simply doesn't exist at that path.

**POST https://jsonplaceholder.typicode.com/posts**
- Body sent: `{"title": "Test Post", "body": "This is a test", "userId": 1}`
- Status: `201 Created`
- Result: server echoed the submitted data back, plus added a new
  `"id": 101` field — demonstrating what a successful "create new
  resource" response looks like. (Note: jsonplaceholder is a fake/test
  API — it doesn't actually persist this new post anywhere; the 201 and
  new ID are simulated for practice purposes.)

---

## 15. Glossary — every term in one place

- **HTTP** — HyperText Transfer Protocol; the shared rules governing how
  clients and servers communicate over a network.
- **Protocol** — an agreed-upon set of rules ensuring different systems
  can understand each other's messages.
- **Client** — the system initiating a request (a role, not a specific
  tool — can be a browser, Postman, n8n, an AI agent, etc.).
- **Server** — the system receiving a request, processing it, and
  returning a response.
- **Method** — the verb describing the intended action of a request (GET,
  POST, PUT, PATCH, DELETE).
- **Endpoint** — the specific URL path identifying a resource on a
  server.
- **Header** — metadata about a request or response, separate from the
  actual body content.
- **Body** — the actual data being sent in a request or returned in a
  response.
- **Status code** — a 3-digit number indicating the outcome of a request
  (grouped 1xx–5xx by category).
- **Idempotent** — an operation that produces the same result no matter
  how many times it's repeated.
- **Rate limit** — a cap on how many requests a client can send in a given
  time period, enforced via a 429 status code when exceeded.

---

## 16. Self-check questions

1. What's the actual difference between a 401 and a 403 status code?
2. Why doesn't a GET request typically include a body?
3. If an automation gets a 500 error, is the fix likely to be "change my
   request" or "try again later"? Why?
4. What does "idempotent" mean, and which of GET/POST/PUT is NOT
   idempotent?
5. Why is calling an AI/LLM API always a POST request, even though you're
   just "asking a question"?

---

*Test requests run in Postman: GET (200), GET on invalid path (404), POST
(201) — all documented above.*
