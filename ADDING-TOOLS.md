# Adding a tool to Toolshed

Checklist for putting a new tool on [toolshed.ffff0h.com](https://toolshed.ffff0h.com).
Work through every section. **Step 3 (privacy) is not optional.** The privacy policy makes promises about every tool, so each new tool has to be checked against it.

---

## 1. Host the tool on its own subdomain

Each tool lives in its own GitHub repo, served from an `ffff0h.com` subdomain.

1. Create the repo and push the tool's site to `main`.
2. Add a `CNAME` file containing the subdomain, e.g. `newtool.ffff0h.com`.
3. Turn on GitHub Pages and set the domain:
   ```
   gh api -X POST repos/zohaibhamdulay/<repo>/pages -f "source[branch]=main" -f "source[path]=/"
   gh api -X PUT  repos/zohaibhamdulay/<repo>/pages -f cname=newtool.ffff0h.com
   ```
4. In Cloudflare DNS for `ffff0h.com`, add **CNAME** `newtool` → `zohaibhamdulay.github.io`, **DNS only (grey cloud)**.
   Every subdomain points at the same target; GitHub picks the repo by the domain in its `CNAME` file.
5. Once the certificate is issued, enforce HTTPS:
   ```
   gh api -X PUT repos/zohaibhamdulay/<repo>/pages -F https_enforced=true
   ```
   If the certificate is stuck (`https_certificate` is `null`), remove and re-add the custom domain to restart the request, then `git pull` (GitHub commits its own `CNAME` changes).

## 2. Add the card to the grid

In [index.html](index.html), replace the next empty box (`<div class="box empty">…`) with a tool card. Copy the structure of card 01:

```html
<li><a class="box tool" href="https://newtool.ffff0h.com">
  <span class="n">02</span>
  <svg class="go" viewBox="0 0 24 24" aria-hidden="true"><path d="M7 17L17 7M9 7h8v8"/></svg>
  <span class="art" aria-hidden="true">
    <svg viewBox="0 0 120 92"><!-- illustration --></svg>
  </span>
  <span class="t">Tool Name</span>
  <span class="d">One-line description of what it does.</span>
</a></li>
```

- Keep the illustration in the site palette (`currentColor` / `--accent`, `--accent-soft`, `--paper`).
- Any `id` inside the SVG (e.g. a `clipPath`) must be unique on the page. Card 01 already uses `id="lens"`.
- The description is hidden on phones, so the title must make sense on its own.

## 3. Review and update the privacy policy (required)

The policy is at [privacy/index.html](privacy/index.html) → https://toolshed.ffff0h.com/privacy/

Answer these questions about the new tool:

| Question | If **yes** |
|---|---|
| Does any user input (files, text, images, passwords) leave the device? | Policy must say what is sent, where, why, and how long it's kept. |
| Does the tool store anything on a server or in a database? | Describe what's stored, retention, and how to request deletion. |
| Does it have accounts, logins or email collection? | Describe what's collected and why. |
| Does it use browser storage (`localStorage`, IndexedDB, cookies) for anything beyond preferences? | Mention it. |
| Does it load a new third-party service (API, CDN, analytics, payments, fonts)? | Add it to **"Other services the site relies on"** with a link to its privacy policy. |
| Does it run ads or tracking beyond the existing AdSense? | Add to **"Advertising and cookies"**. |

Then update the policy:

- **"What the tools do with your data"**: if the tool runs fully on-device, the existing text still covers it. Optionally add it as an example. If it does *not* run fully on-device, change the bullets that say "Every tool… processes your data locally". They would no longer be true. Add a per-tool subsection explaining what that tool does with data.
- **"Other services"**: add any new third parties.
- **"Last updated"** date at the top of the page: change it whenever the policy text changes.
- If the tool collects data, its own page must also say so **before** the user gives it anything (the policy promises this under "Future tools").

If every answer is "no", the policy needs no text changes. Still re-read it to confirm nothing it says has become false.

## 4. Ads (AdSense and ads.txt)

**ads.txt is a one-time setup. You do not need to change it for each new tool.**

Ad networks only read `ads.txt` from the **root domain**: `https://ffff0h.com/ads.txt`. A copy on a tool's subdomain (e.g. `newtool.ffff0h.com/ads.txt`) is not enough on its own. Because of how the domain is set up:

- `ffff0h.com` redirects to Toolshed through a Cloudflare redirect rule that **keeps the path**, so `ffff0h.com/ads.txt` serves [ads.txt](ads.txt) from this repo.
- That one file covers every `*.ffff0h.com` tool, because they all use the same AdSense publisher ID.

It currently contains:
```
google.com, pub-4816264619829761, DIRECT, f08c47fec0942fa0
```

For each new tool:
- Add the AdSense `<script>` (same `ca-pub-4816264619829761` client) to the `<head>` of every page of the tool.
- No new AdSense site is needed. The tool runs on a subdomain of `ffff0h.com`, and that domain is already registered in AdSense.
- Leave `ads.txt` alone.

Only edit [ads.txt](ads.txt) if:
- you add a **different ad network** or a second publisher account (add its line), or
- AdSense shows an "ads.txt" warning with a different line.

Don't break the redirect: if the Cloudflare rule for `ffff0h.com` is ever changed, `https://ffff0h.com/ads.txt` must still return this file. Check with:
```
curl -sIL https://ffff0h.com/ads.txt
```
If a tool starts showing ads, it is already covered by the "Advertising and cookies" section of the privacy policy. Step 3 still applies if it adds any *other* ad or tracking service.

## 5. Before pushing

- Check the grid at desktop and phone widths (2 columns under 640px).
- Click through the new card to the live subdomain over `https://`.
- Check the footer **Privacy Policy** link still works.
- Commit and push; GitHub Pages rebuilds in about a minute.
