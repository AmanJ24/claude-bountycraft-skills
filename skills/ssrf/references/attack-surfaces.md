# SSRF attack surfaces (with example findings)

Each surface lists: what it is, the recon signal, how the response comes back, and the source notes in `../../extracted/ssrf/` it came from. Use this to decide where to look first and what visibility to expect.

## Webhooks and custom integrations
The highest-frequency surface in this set. User supplies an outbound URL the server calls on an event; a "test request"/"send test event" button triggers it on demand; delivery logs often expose the response, making it full-read.
- **Full read via delivery log:** Omise dashboard webhook showed response bodies in "Recent Deliveries", so a 303 redirect to metadata leaked AWS role credentials (note 13). HackerOne org webhook "Test request" showed delivery logs with response headers (note 09).
- **Metadata via integration config:** Dynatrace custom integration webhook reached AWS metadata, rated 10.0 (note 11).
- **Dev-platform command URLs:** Slack slash command request URL, redirect to `[::]` for internal port scan (note 21).
- **Recon:** any "notify a URL", "send to endpoint", "callback", "integration" setting. Look for a log/history view that shows the response.

## PDF / HTML-to-PDF generators
Server renders user-influenced HTML into a PDF. No URL field needed: injected markup makes the fetch from inside the renderer.
- DoD checklist PDF: injected `<script>`/`<iframe>` via a saved JSON name field, read AWS creds, 9.4 (note 05).
- HackerOne analytics export: user value reflected into a "template not found" error that got rendered to PDF, read AWS creds, $25k (note 01). **Test error/not-found paths.**
- PortSwigger research: injection into PDF annotation URIs (unescaped parens) runs PDF JS that POSTs blind, plus DNS exfil, in Acrobat/PDFium (note 26).
- NahamSec talk: engine survey (wkhtmltopdf, headless Chrome, WeasyPrint, Prince); fingerprint engine from callback User-Agent, then use engine-specific fetch features (note 27).
- **Recon:** invoices, receipts, reports, checklists, exports, badge/card renderers. Submit a value, see if it renders as markup in the PDF, then trigger an outbound hit.

## Headless-browser render / screenshot services
A real browser renders a page you control, server-side. The page is the SSRF vehicle; can reach localhost.
- Shopify Exchange store screenshotter: store theme script redirected the renderer to GCP metadata; PNG output showed the token; chained to kube-env, kubelet cert, container root, $25k (note 20).
- h1-CTF: XSS in a PDF converter running headless Chrome with remote debugging reached the local DevTools port and listed other tabs (note 19).
- **Recon:** "preview your store/page", thumbnail, social-card generators. Control the page content; try top-level redirect (iframes to metadata are often frame-blocked).

## Link preview / URL unfurl
Server fetches a user URL to build a preview card. Even a leaked title enumerates internal services.
- Matrix/Reddit `preview_url`: `og:title` of internal addresses leaked service identity, port scan, $6k (note 02).
- **Recon:** chat/message boxes, "paste a link" fields, anything that auto-expands a URL into a card.

## Import-from-URL
Importer fetches a remote resource; if it returns converted content, you get full read.
- GitLab project import: SSRF filter bypass via Ruby Resolv + IP notation, internal port scan (note 22).
- Lark Docs "import as docs": full-read SSRF to internal services, 9.6, $5k (note 04).
- Concrete CMS remote file upload: DNS rebinding past a metadata filter, read IAM keys (note 15).
- Vimeo video import (Google Drive picker): redirect during a ranged download to GCP metadata token, 9.0 (note 18).
- **Recon:** "import from URL/Git/Drive", "upload from link", remote backup, migrate-from.

## "Test connection" / endpoint validation buttons
Admin console feature that fetches an attacker-chosen host and echoes the result. Usually needs an authenticated user with config rights.
- Apache Airflow connection "test" (Slack conn type): non-blind, returned response text; also DoS by repeated tests; fixed in 2.7.0 (note 08).
- **Recon:** DB/API/SMTP/webhook connection setup screens, "validate", "check", "ping" buttons.

## Image / media processors
- **SVG upload:** server-side SVG parsing follows `xlink:href` (HTTP/FTP) and UNC/SMB paths.
  - Shopify product image: SVG disguised with `.png` name + `image/svg+xml` type slipped past a post-processing validator; outbound HTTP/FTP, port reach, file-existence oracle via chained references (note 14).
  - Rockstar emblem editor: SVG referencing a UNC path made ImageMagick authenticate to attacker SMB, leaking the service account NTLMv2 hash, $1.5k (note 10).
- **Video transcode (FFmpeg):** an uploaded file that is really an HLS playlist makes FFmpeg follow URLs and `file://` paths; SSRF + line-by-line local file read, 7.7, $2,727 (note 07).
- **Image-fetch proxy:** fetches and re-encodes a remote image; reached internal "mgmt" hosts; chained into a second internal SSRF (PhantomJS banner service) that accepted `file://` for local image read (note 12).
- **Recon:** any upload that is thumbnailed, converted, or re-encoded. Fingerprint the processor from the callback User-Agent.

## Exposed third-party tooling
- Self-hosted Sentry with "scrape source code" on: send a crafted error event with a stack-frame `filename` URL, server fetches it (note 06). Fingerprint Sentry from the CSP `report-uri`/`sentry_key`.
- **Recon:** read CSP, response headers, JS config for known tools (Sentry, Grafana, Jenkins, Jira, PlantUML, PhantomJS) that have documented SSRF-capable settings.

## Managed cloud control planes
- Managed Kubernetes: a customer-defined StorageClass `resturl` (GlusterFS provisioner) was fetched by the vendor's controller-manager inside its own network; half-blind, upgraded to full via CRLF on old Go, $5k (note 03).
- **Recon:** any field where you supply a URL/endpoint that a *provider-side* component (not your workload) will contact: storage backends, registries, admission webhooks.

## Parameter name quick-list
`url, uri, src, source, dest, destination, target, redirect, redirect_uri, return, returnTo, callback, webhook, webhook_url, endpoint, host, hostname, domain, feed, rss, path, file, filename, image, image_url, img, avatar, logo, icon, resturl, shop_url, data, page, site, link, next, out, view, show, load, fetch, proxy, api, server`
