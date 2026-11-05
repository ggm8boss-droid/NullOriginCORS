# AIMP Launcher — `null`-origin CORS vulnerability

A security write-up and **read-only proof of concept** for a CORS misconfiguration in the
[AIMP Launcher](https://github.com/Dropper7856/AIMP) local API.

While the launcher is running, its local API (`http://127.0.0.1:5127`) trusts the opaque
`null` CORS origin. Because browsers send `Origin: null` from sandboxed iframes and `data:`
URLs, **any web page a user visits can read responses from the local API** — for example the
machine's drive and directory listings — a classic drive-by exposure of a localhost service.

## Contents

- [`index.html`](index.html) — the full advisory **and** an in-page, read-only exploit you can run
  against your own launcher (click **Run read-only probe**).

## View it

This is a static page, so [GitHub Pages](https://pages.github.com/) serves it directly:

1. Push this repo to GitHub.
2. **Settings → Pages → Source → Deploy from a branch →** `main` / `/ (root)` → **Save**.
3. Open `https://<your-username>.github.io/<repo-name>/`.

(Viewing `index.html` in the repo file browser shows its source, not the rendered page — use the
Pages URL.)

## Reproduce the exploit

1. Start the AIMP Launcher so its API is listening on `127.0.0.1:5127`.
2. Open the published page (the GitHub Pages URL is ideal — it is a real remote origin, the true
   attacker position).
3. Click **Run read-only probe**.
   - **Vulnerable build:** 🔓 the page prints your local drive list.
   - **Fixed build (or a browser whose Private Network Access policy blocks it):** 🔒 the request is
     refused.

The proof of concept only calls the read-only `GET /api/v1/filesystem/drives` endpoint and **sends
the result nowhere** — the data stays in your browser. The advisory also includes `curl` evidence
showing the server returns `Access-Control-Allow-Origin: null` regardless of the browser.

## The fix (summary)

Stop trusting `null` by default; accept only concrete local origins (loopback, `file:`/`app:`, or an
origin named by `AIMP_FRONTEND_ORIGIN`), with an explicit, off-by-default `AIMP_ALLOW_NULL_ORIGIN`
opt-in for packaged frontends that genuinely report `Origin: null`. Full diff and rationale are in
[`index.html`](index.html).

## Disclaimer

This is independent security research intended for defensive remediation. The proof of concept is
read-only and non-destructive. It is not affiliated with or endorsed by the AIMP project or the
creators of *Alien: Isolation*.
