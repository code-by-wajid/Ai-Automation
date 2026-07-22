# Day 03 — First AI-Assisted Script

## What I built

A Node.js script (`fetch-users.js`) that fetches users from
`https://jsonplaceholder.typicode.com/users` and prints each one's name
and email.

## The loop I practiced

Prompt → run → predict the failure → observe the actual failure → fix it
→ understand why. Cursor hit its usage limit partway through, so Claude
generated/explained the code directly instead — same exercise either way.

## Key things learned

- `https.get(...)` sends the request (same GET request Postman sent
  manually on Day 2).
- `res.on('data', chunk => ...)` collects the response as it streams in,
  piece by piece — it doesn't all arrive instantly.
- `res.on('end', ...)` fires once the full response has arrived.
- `JSON.parse(data)` converts the raw response text into a usable array
  of objects (same parsing concept from Day 1).

## The bug I found

Pointing the script at a broken URL produced **no output at all** —
a silent failure. Cause: the server returned a 404 with a non-JSON body,
and `JSON.parse()` threw an error with no `try/catch` around it to catch
it, and no status-code check before attempting to parse.

## The fix

- Log `res.statusCode` immediately so failures are visible.
- Check `if (res.statusCode !== 200)` before attempting to parse, and
  exit early with a clear message if the request failed.
- Wrap `JSON.parse()` in a `try/catch` so a malformed response fails
  loudly and clearly instead of silently.

## Takeaway

A script that "does nothing" on failure is worse than one that crashes
loudly — silent failures are the hardest bugs to catch, in a script today
and in a client's automation later.
