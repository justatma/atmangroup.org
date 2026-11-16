# atmangroup.org

Redesign of 2026-11-16. Plain HTML, no JavaScript, self-hosted fonts. This file is not uploaded: the deploy workflow excludes `README.md`.

## What is in this folder

| File | What it is |
|---|---|
| `index.html` | The page. All copy is in the raw HTML. |
| `404.html` | Branded not-found page, marked noindex. |
| `og.jpg` | Share image, 1200 x 630, 48 KB. Replaces `og.png`. |
| `favicon.ico`, `favicon.svg`, `apple-touch-icon.png`, `icon-192.png`, `icon-512.png`, `site.webmanifest` | Icon set. |
| `assets/atma-mark.png` | The Atma mark, carried over from `main` (was `assets/r/d78d95ea1478b734.png`). |
| `assets/fonts/` | Seven woff2 files: Jost 400 to 700, IBM Plex Mono, IBM Plex Serif italic, Noto Sans Devanagari. |
| `robots.txt` | Unchanged from `main`. |
| `sitemap.xml` | Same as `main`, `lastmod` set to 2026-11-16. |
| `.htaccess` | Same as `main`, plus one line: `ErrorDocument 404 /404.html`. |
| `.github/workflows/deploy.yml`, `.gitignore` | Unchanged from `main`. |

Two things on `main` are no longer used and should be deleted from the repo: the folder `assets/r/` and the file `og.png`.

## How to ship it

These steps assume the repo lives at `/Users/atma1/gh_all/sites/atmangroup.org` and the zip was saved to `~/Downloads/atmangroup-org-deploy.zip`. Run every command in Terminal.

**Step 1. Confirm the repo is where I think it is and is clean.**

```
cd /Users/atma1/gh_all/sites/atmangroup.org && git status --short --branch
```

Expected: one line, `## main...origin/main`, and nothing under it.
If you see `No such file or directory`: the repo is somewhere else. Run `ls /Users/atma1/gh_all` and `ls /Users/atma1/gh_all/sites` to find it, then use that path in every step below.
If you see file names under the first line: you have uncommitted work. Stop and deal with that first.

**Step 2. Pull the latest `main`.**

```
cd /Users/atma1/gh_all/sites/atmangroup.org && git pull origin main
```

Expected: `Already up to date.` If it lists changed files instead, someone edited the site after 2026-10-31 and this folder was not built against that version. Stop and tell Claude.

**Step 3. Unpack the zip to a scratch folder.**

```
rm -rf /tmp/atmangroup-new && unzip -q ~/Downloads/atmangroup-org-deploy.zip -d /tmp/atmangroup-new && ls -A /tmp/atmangroup-new/atmangroup.org
```

Expected: a list that starts with `.github`, `.gitignore`, `.htaccess`, `404.html` and includes `index.html` and `og.jpg`.
If `unzip` says it cannot find the file: the download has a different name. Run `ls -t ~/Downloads | head -5` and use the name it shows.

**Step 4. Remove the two things the new page no longer uses.**

```
cd /Users/atma1/gh_all/sites/atmangroup.org && git rm -r -q assets/r og.png && git status --short | head -5
```

Expected: lines beginning with `D ` for `assets/r/...` and `og.png`.

**Step 5. Copy the new files in.**

```
cd /Users/atma1/gh_all/sites/atmangroup.org && rsync -a /tmp/atmangroup-new/atmangroup.org/ ./ && git add -A && git status --short
```

Expected: `M` beside `.htaccess`, `index.html` and `sitemap.xml`; `A` beside `404.html`, `README.md`, `og.jpg`, the icon files, `site.webmanifest` and `assets/atma-mark.png` and the seven files in `assets/fonts/`; `D` beside `og.png` and the old `assets/r/` files. `deploy.yml`, `.gitignore` and `robots.txt` should not appear, because they are identical.
If `deploy.yml` appears as modified: stop. It means `main` changed after this folder was built.

**Step 6. Commit and push. The push deploys.**

```
cd /Users/atma1/gh_all/sites/atmangroup.org && git commit -q -m "Redesign: projector hero, plain HTML, launch kit" && git push origin main
```

Expected: a line ending in `main -> main`.

**Step 7. Watch the deploy.**

Open https://github.com/justatma/atmangroup.org/actions in a browser. The newest run should turn green within a few minutes. The step named "Crawlability gate" should print `every indexable page has readable HTML`.
If the first FTPS step fails and the retry step passes, that is the known GoDaddy socket drop and the deploy is fine.

**Step 8. Check the live site the way a crawler sees it.**

```
curl -s https://atmangroup.org/ | grep -c "Nine years a monk."
```

Expected: `6` (the title, three share tags, the structured data and the headline). `0` means the old page is still being served: wait a minute and run it again.

```
curl -s -o /dev/null -w "%{http_code} %{content_type}\n" https://atmangroup.org/og.jpg
```

Expected: `200 image/jpeg`.

```
curl -s -o /dev/null -w "%{http_code}\n" https://atmangroup.org/no-such-page
```

Expected: `404`. Open that address in a browser and you should see "Nothing here."

**Step 9. Refresh the link previews.**

Paste `https://atmangroup.org/` into LinkedIn Post Inspector and the Facebook Sharing Debugger and re-scrape. Both cache the old monk image until you do.

**Step 10. Run the audit.**

From your `website_ops` folder: `python3 audit/audit.py`. Expected: no FAIL lines for atmangroup.org.
