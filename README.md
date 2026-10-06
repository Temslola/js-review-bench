# JS Review Bench

A static-review workbench for JavaScript, built for penetration testers and source-code reviewers.
Drop in a bundle or a module and it gives you credentials, injection sinks, endpoints, a logic map,
and a review run-book to work through.

**Everything runs in the browser.** No upload, no API call, no telemetry. The code you analyze never
leaves the machine, which is what makes it safe to use on client source during an engagement.

## Run it

It is one self-contained HTML file with no build step and no dependencies.

```bash
open index.html
```

Or serve it:

```bash
python3 -m http.server 8080
```

## Deploy it

Any static host works. The whole site is `index.html`.

| Host | How |
| --- | --- |
| Netlify | Drag the folder onto <https://app.netlify.com/drop> |
| GitHub Pages | Push the repo, then Settings → Pages → deploy from `main`, folder `/` |
| Cloudflare Pages | Connect the repo; no build command, output directory `/` |
| Vercel | `npx vercel --prod` from this folder |

No build command, no framework, no environment variables.

## What it does

| Section | Purpose |
| --- | --- |
| Overview | Size, minification, inferred stack (React / Vue / Angular / Node / webpack / sourcemap) with stack-specific guidance, severity mix, and the functions to read first |
| Findings | 35 rules across credentials, injection sinks, logic & auth, and attack surface. Each explains what an attacker gains and how to confirm it against the running app. Triage state persists per browser |
| Source → sink | Pairs attacker-controllable values with execution sinks that appear near them — a DOM-XSS candidate list |
| Endpoints | Every URL and path literal, split into notable / API / other |
| Logic map | Named functions grouped by concern: authentication, authorization, money, validation, crypto, non-production |
| Code | Line-numbered source, finding lines tinted, search, and a pretty-printer for minified bundles |
| Run-book | Ten review activities with the questions to answer at each, tracked per file |

"Copy as Markdown" exports findings plus run-book progress for your report.

## Detection coverage

Credentials: AWS access keys and secrets, Google API keys and service accounts, Stripe live and test
keys, GitHub tokens, Slack tokens and webhooks, JWTs, private key blocks, database and broker
connection strings, credentials embedded in URLs, named secret assignments, and Shannon-entropy
scoring with placeholder filtering.

Sinks: `eval`, `Function`, string `setTimeout`, `innerHTML`, `outerHTML`, `insertAdjacentHTML`,
`document.write`, jQuery HTML methods, `dangerouslySetInnerHTML`, Vue `v-html`, Angular
`bypassSecurityTrust*`, dynamic `src`/`href`/`action`, prototype-pollution shapes, `postMessage`
handlers, storage and cookie access, WebSockets.

Logic and surface: client-side authorization checks, feature flags, debug switches, weak or
client-side crypto, permissive CORS and credentialed requests, developer notes, cleartext HTTP,
internal and non-production hostnames, cloud storage buckets, GraphQL operations, upload handling,
and client-side route tables.

## Adding rules

Rules live in the `RULES` array near the top of the `<script>` block in `index.html`. Each one is:

```js
{
  id: "short-slug",
  cat: "secret" | "sink" | "logic" | "surface",
  sev: "critical" | "high" | "medium" | "low" | "info",
  title: "What shows in the findings list",
  re: /a global regex/g,
  why: "What an attacker gains. One or two sentences.",
  verify: ["The step that confirms it.", "What raises or lowers severity."]
}
```

Keep `why` and `verify` substantive — the point of the tool is that it teaches as it detects, so a
rule that only flags a pattern is half a rule.

## Scope

Static output is leads, not findings. Confirm each one against a system you are authorized to test.
Treat discovered credentials as report-and-stop: prove they are live with the smallest possible
read-only call, and do not exercise them further.
