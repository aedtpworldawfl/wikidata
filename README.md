# AWFLWIKIDATAWORLD

| Field | Value |
| --- | --- |
| title | AWFLWIKIDATAWORLD |
| caption | Full documentation of AWFLWIKIWORLD WIKIDATA v25.4 |
| Product | AWFLWIKIWORLD WIKIDATA |
| Version | 25.4 (asset cache-buster v=2541) |
| Publisher | AEDTP WORLD (https://aedtpworld.com) |
| License | AEDTP WORLD FREE LICENSE (AWFL) |
| Runtime | PHP 7.2 or newer, plain JavaScript and CSS |
| Database | None, flat JSON and TXT files in wiki/ |
| Web server | Apache with mod_rewrite |
| Default URL | https://aedtpworld.com/wikidata |
| Package | 44 files, about 260 KB unpacked |



# AWFLWIKIDATAWORLD

**AWFLWIKIDATAWORLD** is the full documentation of **AWFLWIKIWORLD WIKIDATA v25.4**, a free, open, flat-file "wikidata" (structured knowledge base) published by **AEDTP WORLD** under the **AEDTP WORLD FREE LICENSE (AWFL)**. Tagline: *Discover And Connect.*

This page is written in AWFLWIKITEXT, the markup language of AWFLWIKIWORLD. See also [AWFLWIKIWORLD](https://aedtpworld.com/wikiworld/index.php/AWFLWIKIWORLD), the sister wiki engine, and [AWFLWIKIWORLD/AWFLWIKITEXT](https://aedtpworld.com/wikiworld/index.php/AWFLWIKIWORLD/AWFLWIKITEXT).

## 1. Overview
The software lets anyone create and edit **items** (identified as `Q1`, `Q2`, `Q3` and so on), in the style of Wikidata. Each item has:

* a **name** (label) and a short **description**
* **aliases** (alternative names)
* **images** (uploaded files or external URLs)
* **statements** (property and value pairs, for example `country` = `Q5`)
* **identifiers** (links to external databases such as IMDb, Spotify, GitHub)

Besides items there is one special page, the **Main_Page**, which is free-form HTML editable only by the admin.

### 1.1 Key characteristics
* **No database.** Everything is stored as human-readable JSON and TXT, written atomically and guarded by a file lock.
* **Controlled vocabulary.** Users may only choose properties from `label.txt` and identifiers from `id.txt`; the admin curates both lists.
* **Public editing with safeguards.** Anyone can create or edit items, protected by CSRF tokens, a question-and-answer captcha, upload validation and per-IP rate limiting. The admin can protect items from public edits.
* **Public recent-changes log** with character-count differences.
* **SEO-ready.** Canonical URLs, Open Graph and Twitter meta tags, JSON-LD (schema.org), and automatic multi-file sitemaps.
* **Six export formats** per item: HTML, TXT, Markdown, JSON, AEDTPWORLD, PDF.
* **Admin backup history** with one snapshot per Q and one-click restore.
* **Responsive UI** with a card layout for tables on phones and a print stylesheet.

## 2. Requirements and installation
### 2.1 Requirements
* Apache (or compatible) with `mod_rewrite` and `AllowOverride All`.
* PHP 7.2 or newer (`mb_chr` is required) with the extensions mbstring, iconv, fileinfo (for `mime_content_type`), exif, image support for `getimagesize`, and session.
* A directory writable by the PHP user. The app creates `wiki/`, `includes/` and `wiki/uploads/` if missing (mode 0755).

### 2.2 Steps
1. Upload the `wikidata` folder to your web root, for example https://yourdomain.com/wikidata
2. Edit `config.php`: set `$Server` to your public URL (no trailing slash), `$Logo` to your logo URL, and `$Sitename`, `$MetaTitle`, `$MetaDescription` as you like.
3. Change `$Username` and `$Password`. The shipped defaults are `admin` and `adminpassword`, marked "CHANGE THIS" in the file.
4. Make sure the folder is writable by PHP.
5. Open `/dashboard.php`, log in, and click **Rebuild sitemap**.
6. Visit the site root. The first request auto-creates any missing default files (see section 10).

> Security warning: the admin credentials are stored in plain text in `config.php`. Change them before going live and keep the file out of version control.

## 3. Package layout

```
wikidata/
  .htaccess            URL rewrite rules (sitemaps, /Q42 routing)
  config.php           Master config + all shared functions
  index.php            Router, Main_Page and item pages, exports, live search
  create.php           Create a new item
  edit.php             Edit an existing item
  search.php           Full search page + live-search endpoint
  recent.php           Public recent-changes list (latest 100)
  dashboard.php        Admin login and control panel
  history.php          Admin backup history and restore
  upload.php           Image upload (form and AJAX endpoint)
  sitemap.php          Dynamic sitemap index / chunks
  includes/
    .htaccess          Denies direct access to *.php
    header.php         HTML head, SEO meta, JSON-LD, navigation
    footer.php         Footer and site.js loader
    form.php           Item form: validation (parse_post) and rendering
    export.php         HTML/TXT/MD/JSON/AEDTPWORLD/PDF exporters
    pdf.php            Minimal built-in PDF writer
    site.css           Stylesheet (96 lines)
    site.js            Live search + dynamic form widgets
    index.html         Empty (blocks directory listing)
  wiki/                DATA DIRECTORY (flat files)
    .htaccess          Denies web access to txt, json, lock, log, tmp
    q.txt              List of existing Q ids
    name.txt           Q -> name
    description.txt    Q -> description
    value.txt          Q -> list of statements
    aliases.json       Q -> list of aliases
    images.json        Q -> list of images
    identifiers.json   Q -> {identifier name: full URL}
    label.txt          Allowed property names (26 defaults)
    id.txt             Allowed identifier names + link prefixes (23 defaults)
    icon.txt           Keyword -> emoji (463 entries)
    country_flag.json  Country name -> ISO code (455 entries)
    colors.json        Theme colors
    settings.json      Runtime settings (upload_enabled)
    security.txt       Captcha questions "question|answer"
    protected.json     Protected Q list
    recent.json        Public change log (max 1000 rows)
    history.json       Admin backups (one per Q)
    views.json         View counters
    lastmod.json       Last-modified timestamps
    Main_Page.txt      Main page HTML
    index.html         Empty
    uploads/           Uploaded images (+ .htaccess, index.html)
```


## 4. Configuration (config.php)
`config.php` is the master file: every page requires it. It holds settings, default-file generation, storage helpers and almost all business logic.

### 4.1 Settings variables
* `$Sitename`: name shown in the header and footer. Default "AWFLWIKIWORLD WIKIDATA".
* `$MetaTitle`: suffix used in page titles. Default "AWFLWIKIWORLD WIKIDATA".
* `$MetaDescription`: default meta description. Default "A free, open wikidata by AEDTP WORLD. Discover And Connect."
* `$Server`: public base URL without a trailing slash. Default https://aedtpworld.com/wikidata
* `$Logo`: logo, favicon and default Open Graph image. Default ![](https://aedtpworld.com/wikiworld/logo/aedtpworld.svg)
* `$Username`: admin username. Default `admin`.
* `$Password`: admin password. Default `adminpassword`. Change it.
* `$route`: URL style. "0" gives `index.php?title=Q1`; "1" (default) gives `/Q1` and also `/wiki/Q1`.
* `$upload_enabled`: default for public upload (false). The live value is read from `wiki/settings.json`, controlled by the dashboard toggle.

The timezone is fixed to UTC. Derived paths are `$base`, `$wikiDir`, `$incDir` and `$upDir`.

### 4.2 Bootstrapping
On every request `config.php`:

1. creates missing directories;
2. writes any **missing** default file (existing files are never overwritten);
3. starts a session with cookie flags HttpOnly, SameSite=Lax, and Secure when on HTTPS;
4. loads `settings.json` to override `$upload_enabled`.

### 4.3 Function reference
#### Storage and locking
* `h($s)`: HTML-escape (UTF-8, quotes).
* `jread($file,$default)`: read and decode a JSON file from `wiki/`, cached per request.
* `atomic_write($path,$data)`: write to a temp file, then `rename()`; falls back to a locked direct write.
* `jwrite($file,$data)`: pretty-printed, unescaped-Unicode JSON atomic write; updates the cache.
* `with_lock($fn)`: clears caches, takes an exclusive `flock` on `wiki/.lock`, runs the callback, releases. All writes go through it.
* `client_ip()` and `mask_ip($ip)`: visitor IP, and its masked public form (a.b.c.x, or a:b:c::x for IPv6).
* `now_str()`: UTC timestamp, format Y-m-d H:i.

#### Icons and flags
* `flag_from_code($code)`: converts a 2-letter ISO code to a flag emoji.
* `get_icon($val)`: returns an emoji plus a space. Order: exact match in `icon.txt`; exact match in `country_flag.json`; for strings up to 80 characters, a whole-word search for any country name (longest names first). Otherwise an empty string.

#### IDs and URLs
* `valid_q($q)`: matches Q followed by digits.
* `norm_title($t)`: normalises input to `Main_Page` (accepts "main page" and "main_page"), an upper-case Q id, or null.
* `build_item_url($id)`: canonical URL (depends on `$route`); used in sitemaps, canonical tags and exports.
* `q_url($id)`: internal link, always `index.php?title=Qn`.
* `upload_url($name)`: full URL for an uploaded file; passes full http(s) URLs through unchanged.

#### Item list
* `q_list()`, `q_save($list)`, `q_next()` (highest number plus one), `item_exists($q)`.

#### Properties and identifiers
* `label_names()`, `canonical_label($p)` (case-insensitive match returning the canonical spelling), `label_for($p)`.
* `id_map()`, `canonical_id_name($n)`.

#### Items
* `item_get($q)`: assembles an item with q, name, desc, aliases, images, claims, ids, protected.
* `item_text_size($it)`: character count of all text; used for the change difference.
* `map_set($file,$q,$val)`: sets or removes (when empty) one Q's entry in a map file.
* `item_write($q,$it)`: raw write to all data files, adds the Q to `q.txt`, updates `lastmod.json`. The caller holds the lock.
* `empty_item($q)`: blank item structure.
* `item_commit($q,$new,$comment,$isNew)`: locked write, logs the recent change, snapshots history if admin, may rebuild the sitemap.
* `item_delete($q)`: removes the Q from every file, logs "deleted", rebuilds the sitemap.

#### Main page, logging, history, views
* `main_get()` and `main_save($text,$comment)`.
* `recent_log($id,$diff,$comment)`: prepends a row to `recent.json` (kept to 1000). Admin edits show "admin"; others show the masked IP, and the full IP is stored for the admin only.
* `history_snapshot($id)`, `history_restore($id)`, `history_stats()`: see section 7.
* `views_get($id)` and `views_hit($id)`: count **one view per IP per page per hour** (hashed key in `views_ip.json`); HEAD requests are not counted.

#### Search and rendering
* `search_items($term,$limit)`: ranking is described in section 6.5.
* `render_results($ids)`, `render_value($v,$prop)`, `render_gallery($images,$alt)`, `id_display($name,$url)`, `ext_icon()`.

#### Auth, CSRF, captcha, sitemap
* `is_admin()`, `csrf_token()`, `csrf_field()`, `csrf_ok()`.
* `captcha_lines()`, `captcha_question()`, `captcha_ok()`: see section 8.3.
* `sitemap_entries()`, `sitemap_chunks()`, `sitemap_build()`, `sitemap_maybe()`: see section 9.2.

## 5. Data model and file formats
All files live in `wiki/`. JSON is written pretty-printed with unescaped Unicode and slashes.

* `q.txt`: one Q id per line. Example: Q1
* `name.txt`: JSON object, Q to name. Example: `{"Q1":"Universe"}`
* `description.txt`: JSON object, Q to description. Example: `{"Q1":"totality of space and time"}`
* `value.txt`: JSON object, Q to a list of objects with keys p (property) and v (value). Example: `{"Q1":[{"p":"instance of","v":"concept"}]}`
* `aliases.json`: Q to a list of strings. Example: `{"Q1":["cosmos","the universe"]}`
* `images.json`: Q to a list of file names or URLs.
* `identifiers.json`: Q to an object of identifier name and full URL.
* `label.txt`: object of property name and display label (26 defaults).
* `id.txt`: object of identifier name and link prefix (23 defaults).
* `icon.txt`: object of lower-case keyword and emoji (463 entries).
* `country_flag.json`: object of lower-case country name and ISO 3166 code (455 entries, including official long names).
* `colors.json`: six hex colors: header, headerText, button, buttonText, wikiText, links.
* `settings.json`: runtime settings. Example: `{"upload_enabled":false}`
* `security.txt`: one question and answer per line, separated by a vertical bar.
* `protected.json`: list of protected Qs.
* `recent.json`: list, newest first, max 1000 rows, with keys Q, time, ts, by, ip, diff, comment.
* `history.json`: Q to an object with statements, identifiers, time, by (admin backups).
* `views.json`: Q to view count.
* `lastmod.json`: Q or Main_Page to a Unix timestamp.
* `Main_Page.txt`: raw HTML of the main page.
* Created at runtime: `views_ip.json` (hashed IP and page keys, pruned after one hour), `upload_rate.json` (per-IP upload timestamps), `.lock` (lock file for `flock`).

### 5.1 Item field limits (includes/form.php)
* Name: 200 characters (`LIM_NAME`)
* Description: 100 characters (`LIM_DESC`)
* Each alias: 60 characters (`LIM_ALIAS`)
* Aliases per item: 30 (`LIM_ALIASES`)
* Images per item: 20 (`LIM_IMAGES`)
* Statements per item: 100 (`LIM_CLAIMS`)
* Identifiers per item: 100 (`LIM_IDS`)
* Statement value: 1000 characters (`LIM_VALUE`)
* URL or identifier: 500 characters (`LIM_URL`)
* Edit summary: 200 characters

### 5.2 Statement values
A statement value can be:

* **Plain text**, for example `concept`.
* **A Q-id**, typed as `q5` or `Q5` and normalised to upper case. It is shown as a clickable pill with the target item's name, for example "Human (Q5)". If the target does not exist, the pill is red ("missing").
* **A URL** starting with http:// or https://, shown as an external link with rel="noopener noreferrer nofollow ugc".

Duplicate property and value pairs are removed (case-insensitive). Values matching an icon or a country name get an emoji prefix.

### 5.3 Identifiers
Users enter only the **username or ID**, never the link. The app builds the URL as the prefix plus the URL-encoded value (slashes kept). Rules:

* The identifier name must exist in `id.txt` and have a valid http(s) prefix.
* The value may not contain whitespace or look like a link (http:, www., or a slash-colon-slash pattern).
* For prefixes ending in `@` (for example the YouTube prefix), a leading `@` typed by the user is stripped.
* At most 150 characters for the ID and 500 for the final URL, which must pass `FILTER_VALIDATE_URL`.

### 5.4 Images
An image entry must be either a full http(s) URL (3 to 500 characters, no spaces and none of the characters `< > " '`) or the name of a file that already exists in `wiki/uploads/` and matches `^[a-f0-9]{6}_[a-z0-9_-]{0,40}\.(jpg|png|webp)$`.

### 5.5 Example records

```
// recent.json (one row)
{"Q":"Q7","time":"2026-10-03 04:34","ts":1790000040,"by":"203.0.113.x","ip":"203.0.113.9","diff":57,"comment":"created"}

// history.json (one entry)
"Q7": {
  "statements": {"name":"Ghana","description":"country in West Africa","aliases":["GH"],"image":[],"claims":[{"p":"capital","v":"Accra"}]},
  "identifiers": {"Wikipedia":"https://en.wikipedia.org/wiki/Ghana"},
  "time": "2026-10-03 04:40",
  "by": "admin"
}
```


## 6. Pages and features
### 6.1 Routing (.htaccess and index.php)
Rewrite rules apply only when the request is not an existing file or directory:

* `/sitemap.xml` goes to `sitemap.php`.
* `/sitemap-N.xml` goes to `sitemap.php?n=N`.
* `/Q123`, `/wiki/Q123` and `/Main_Page` (optional trailing slash) go to `index.php?title=...`.

`index.php` resolves the id from `?title=` or from the request path; with no title it shows the Main Page. Unknown or deleted items return **HTTP 404** with noindex, follow, and links to create the item.

### 6.2 Main Page
* Shows the admin-edited HTML (`Main_Page.txt`), the export bar, the 20 newest items with the total count, and buttons for Create Item and Recent changes.
* The admin can open `?title=Main_Page&action=edit` to edit the HTML (any HTML allowed, admin only) with an edit summary. Saving is a CSRF-protected POST; non-admins are redirected silently.

### 6.3 Item page
Shows the name with emoji and Q id, a "protected" tag if applicable, the view count, description, aliases ("Also known as"), an image gallery (3 columns on wide screens), the export bar, an Edit button (hidden for protected items unless admin), the **Statements** table and the **Identifiers** table (links open in a new tab with an external-link icon).

Each page sets the title, description, canonical link, Open Graph and Twitter tags, a page-level JSON-LD Thing (name, description, alternateName, image, and sameAs from identifiers) and a Last-Modified header (not sent to admins).

### 6.4 Create and edit (create.php, edit.php, includes/form.php)
* `create.php` assigns the next free Q number inside a lock, writes the item, logs "created" (or your summary), snapshots to history if the editor is admin, rebuilds the sitemap if stale, then redirects to the new item.
* `edit.php?title=Qn` returns to the home page if the item does not exist. For **protected** items it shows a 403 page to non-admins.
* The form has: Name (required), Description, dynamic rows for Aliases, Images (live thumbnail and optional Upload button), Statements (property combo box and value), Identifiers (identifier combo box and username/ID), an Edit summary, and the captcha question for non-admins.
* On a validation error the form is shown again with all messages and the input preserved.
* **Combo boxes** (site.js) show 7 matches, filter as you type, expand with "More (N more)...", and support arrow keys, Enter and Escape. A value not in the list turns the field red and blocks submit.

### 6.5 Search
* **Live search** in the header: after 200 ms of typing it calls `index.php?live=1&q=...` and shows up to 8 results under the header.
* **Search page** `/search.php?q=...` shows up to 100 results (noindex, follow).
* Ranking, lower is better, with newest items winning ties: 0 exact Q id or exact name; 1 name starts with the term; 2 name contains the term; 3 an alias contains the term; 4 the description contains the term; 5 the Q id contains the term.
* Results show a thumbnail (first image) or a letter avatar, the name with icon, the Q id and the description.

### 6.6 Recent changes (recent.php)
A public table of the latest 100 edits: item, change (green +N or red -N characters), time (UTC), editor and summary. Editors appear as masked IPs; logged-in admins see full IPs, and admin edits show "admin". The log keeps 1000 rows. The page is noindex, follow.

### 6.7 Upload (upload.php)
Accepts JPEG, PNG and WebP up to **2 MB**. Public upload must be switched ON in the dashboard (the admin can always upload). The same endpoint serves the standalone form and the AJAX upload button in the editor, returning JSON with ok, name and url, or ok false with an error. Validation is described in section 8.4. On success the stored file name (for example `![](images/a1b2c3_photo.jpg)`) goes into the Images field.

### 6.8 Exports
Every item and the Main Page can be exported from the buttons on the page or directly with `?title=Q1&export=FORMAT`. Responses are downloads with `X-Robots-Tag: noindex`.

* **HTML** (`html`, text/html): standalone styled page with canonical link, statement and identifier tables, image grid.
* **TXT** (`txt`, text/plain): title, description, aliases, URL, statements, identifiers, images, source line.
* **Markdown** (`md`, text/markdown): headings, statements table (pipes escaped), links for Q-values and identifiers, images.
* **JSON** (`json`, application/json): the export model below.
* **AEDTPWORLD** (`aedtpworld`, application/json): the JSON wrapped with format "aedtpworld", version "1.0", generator, export time, license and the item.
* **PDF** (`pdf`, application/pdf): built-in text-only A4 PDF writer, multi-page, Helvetica (Windows-1252 text).

A "Print as PDF" button also opens the browser print dialog; the print stylesheet hides the header, footer and controls.

Export model (JSON):

```
{
  "id": "Q7",
  "name": "Ghana",
  "description": "country in West Africa",
  "aliases": ["GH"],
  "images": ["https://.../![](images/wiki/uploads/a1b2c3_flag.png)"],
  "statements": [
    {"property":"instance of","value":"Q5","item":"Q5","label":"Country","url":"https://.../Q5"},
    {"property":"capital","value":"Accra"}
  ],
  "identifiers": {"Wikipedia":"https://en.wikipedia.org/wiki/Ghana"},
  "url": "https://.../Q7"
}
```


For Main_Page the model holds `content` (the HTML) instead of statements; the TXT, MD and PDF exports convert it to plain text. An unknown format returns HTTP 400.

### 6.9 Views
Each item and the Main Page shows a view counter. A view is counted once per IP per page per hour; the counter is written under lock and old hash keys are pruned.

## 7. Admin dashboard and history
### 7.1 Login
`/dashboard.php` shows a login form when logged out. Credentials are compared with `hash_equals` against `$Username` and `$Password`. Success regenerates the session id and sets the admin flag. All dashboard actions are CSRF-protected POST requests.

### 7.2 Dashboard sections
* **History backup card**: number of backed-up Qs and file size in KB (red above 500 KB), with a link to the history page.
* **Enable Public Upload**: ON/OFF switch stored in `settings.json`.
* **Sitemap**: file count and item count, link to `sitemap.xml`, rebuild button.
* **Most viewed (top 10)**.
* **Properties (label.txt)**: add several properties (one per line, max 100 characters, duplicates ignored case-insensitively) or remove any. Removing a property does not alter existing statements, which keep their text.
* **Identifiers (id.txt)**: add with `Name | https://prefix/` per line. A valid http(s) prefix is required; lines without one are skipped and reported. Saving an existing name updates its link. Any identifier can be removed.
* **Colors**: six color pickers (header, headerText, button, buttonText, wikiText, links), applied as CSS variables on every page.
* **Items** (50 per page): filter by Q or name; Edit, Protect or Unprotect, and Delete per item.
* **Top buttons**: Logout, Edit Main Page, History backup, Recent changes.

### 7.3 Protect and delete
* **Protect**: the item gets a "protected" tag; public edit attempts return a 403 page; only the admin can edit it.
* **Delete**: removes the item from every file, removes protection, logs "deleted" in recent changes (negative size difference), and rebuilds the sitemap. A history backup, if any, is kept so the item can be restored.

### 7.4 History backup (history.php)
* A snapshot of an item (or the Main Page) is stored **only when the admin saves** it, and only **one snapshot per Q** (the newest overwrites).
* The list is sorted by time and shows the Q, name, size summary ("N statements, N identifiers, N images"), time, and a "deleted/missing live" tag when the live item no longer exists.
* **Restore** overwrites the live item with the snapshot (name, description, aliases, images, statements, identifiers; protection state is kept) and logs "restored from backup". Restoring a deleted Q recreates it.
* **Delete** removes one snapshot and **Delete all backups** clears the file. All actions ask for confirmation.
* A capacity warning appears above 500 KB.

## 8. Security model
### 8.1 Authentication and sessions
A single admin account. The session cookie is HttpOnly and SameSite=Lax, plus Secure over HTTPS. The session id is regenerated on login.

### 8.2 CSRF
Every POST form includes a per-session random token (16 random bytes in hex), verified with `hash_equals`. A missing or wrong token gives "Invalid session. Reload and try again."

### 8.3 Captcha
Non-admin create and edit require answering a random question from `security.txt` (one `question|answer` per line, compared case-insensitively and trimmed). The question stays in the session until answered, then is discarded. If `security.txt` is empty the captcha is disabled. Defaults: "What is 2+2?" (4), "What color is sky?" (blue), "Capital of Ghana?" (accra). Add your own lines to make automation harder.

### 8.4 Upload hardening
1. Public uploads must be enabled (the admin is always allowed) and CSRF is required.
2. **Rate limit** for non-admins: 20 uploads per IP per hour (HTTP 429 after that).
3. Size 1 byte to 2 MB, plus an `is_uploaded_file` check.
4. NUL bytes and **double extensions** containing a dangerous segment (for example `![](images/shell.php.jpg)`) are rejected: php, phtml, phar, pl, py, cgi, asp, jsp, sh, exe, htaccess, html, htm, js, svg.
5. The extension must be jpg, jpeg, png or webp.
6. Content must match: `mime_content_type`, `exif_imagetype` and the extension must all agree, and `getimagesize` dimensions must be between 1 and 10000 px.
7. Safe name: 6 random hex characters, an underscore, the sanitised base name (max 40 characters of a-z, 0-9, underscore, dash), and the extension from the verified MIME type. File mode 0644.
8. `wiki/uploads/.htaccess` disables PHP engines and denies php, phtml, phar, pl, py, cgi, sh, txt, json, lock, log and tmp files.

### 8.5 Data protection
* `wiki/.htaccess` denies web access to txt, json, lock, log and tmp files, so raw data (including `history.json`, `security.txt` and full IPs in `recent.json`) is not downloadable. **On a server that ignores .htaccess (nginx, for example) you must add equivalent deny rules** or move `wiki/` outside the web root.
* `includes/.htaccess` denies direct requests to php files; `site.css` and `site.js` stay public. Empty `index.html` files prevent directory listing.
* All output is escaped with `h()`. JSON embedded in script tags uses the JSON_HEX flags for tags, ampersands and quotes. User-supplied links get rel nofollow.
* Writes are atomic and locked, so concurrent edits do not corrupt files.

### 8.6 Privacy
Public pages show only masked IPs. Full IPs are stored in `recent.json` for admin review, and hashed keys are used for the view counter. Operators should mention this in their privacy policy where required.

### 8.7 Known caveats
* Admin credentials are plain text in `config.php`; use a strong password and restrict file permissions.
* Main Page HTML is not sanitised by design (admin only); never give admin access to untrusted people.
* The captcha is a static question list, not a bot-proof system; combine it with server-level rate limiting if abused.
* Flat-file storage suits small to medium datasets; each request reads the JSON maps it needs, so very large item counts will slow down.
* The history file has no automatic size limit, only a warning above 500 KB.
* `$route` set to "1" needs working rewrite rules.

## 9. SEO and sitemaps
### 9.1 Meta data (includes/header.php)
Every page sends: title, description, author, copyright, keywords, license (AEDTP WORLD FREE LICENSE (AWFL)), robots, application-name, canonical link, sitemap link, Open Graph (title, description, type, url, image, site_name), Twitter card (summary_large_image), icon and apple-touch-icon (the logo), an **Organization JSON-LD** block with sameAs links to AEDTP WORLD profiles, and on item pages a **Thing JSON-LD** block.

Default robots: index, follow, max-image-preview:large, max-snippet:-1. Marked noindex: create, edit, search, recent, dashboard, history, upload, 404 pages, exports, and the Main Page editor.

### 9.2 Sitemaps
* `sitemap.xml` is a **sitemap index** pointing to `sitemap-1.xml`, `sitemap-2.xml` and so on. Each chunk is at most **900 KB** (921,600 bytes) or 50,000 URLs.
* Entries: the home page, `recent.php`, and every Q (using `lastmod.json`, with the file time of `q.txt` as a fallback).
* `sitemap_maybe()` rebuilds automatically (when the file is missing or older than 10 minutes) after edits and Main Page visits by non-admins.
* The dashboard button "Rebuild sitemap" forces a rebuild and deletes obsolete chunk files.
* If the folder is not writable, the index points to the dynamic URLs `sitemap.php?n=N` instead.
* `sitemap.php` without parameters serves a cached index (under 60 seconds old) or rebuilds it.

## 10. Default files
When missing, `config.php` creates these. Existing files are never overwritten.

* `q.txt`: Q1
* `name.txt` and `description.txt`: Q1 "Universe" and "totality of space and time"
* `value.txt`: Q1, instance of = concept
* `aliases.json`: Q1, cosmos and the universe
* `label.txt`: 26 properties: instance of, country, capital, occupation, date of birth, place of birth, official website, language, genre, part of, located in, population, founded by, inception, author, creator, member of, spouse, child, parent, educated at, award received, record label, website, title, nickname
* `id.txt`: 23 identifiers (section 10.1)
* `security.txt`: 3 captcha questions
* `colors.json`: header #ffffff, headerText #202122, button #0645ad, buttonText #ffffff, wikiText #202122, links #0645ad
* `settings.json`: upload_enabled false
* `Main_Page.txt`: welcome heading, tagline, links to Q1, recent changes and create item
* `icon.txt` and `country_flag.json`: small starter sets (the shipped package holds the full 463 and 455 entry versions)
* Empty JSON: `images.json`, `identifiers.json`, `history.json`, `views.json`, `lastmod.json` as an empty object; `recent.json` and `protected.json` as an empty list
* The `.htaccess` files and `index.html` placeholders: root rewrite rules and deny rules for `wiki/`, `includes/` and `uploads/`

### 10.1 Default identifiers
* Genius: https://genius.com/artists/
* Google Knowledge Graph: https://g.co/kg/
* IMDb: https://www.imdb.com/name/
* Spotify: https://open.spotify.com/artist/
* YouTube: https://youtube.com/@
* Wikidata: https://www.wikidata.org/wiki/
* Wikipedia: https://en.wikipedia.org/wiki/
* MusicBrainz: https://musicbrainz.org/artist/
* Discogs: https://www.discogs.com/artist/
* Apple Music: https://music.apple.com/artist/
* Deezer: https://www.deezer.com/artist/
* SoundCloud: https://soundcloud.com/
* Last.fm: https://www.last.fm/music/
* Instagram: https://instagram.com/
* X (Twitter): https://x.com/
* Facebook: https://facebook.com/
* TikTok: https://www.tiktok.com/@
* LinkedIn: https://www.linkedin.com/in/
* GitHub: https://github.com/
* Crunchbase: https://www.crunchbase.com/organization/
* Goodreads: https://www.goodreads.com/author/show/
* VIAF: https://viaf.org/viaf/
* Library of Congress: https://id.loc.gov/authorities/names/

## 11. Front-end
### 11.1 Stylesheet (includes/site.css)
* Mobile-first, using CSS variables for header, headerText, button, buttonText, wikiText and links, injected from `colors.json`.
* Components: header, brand and navigation, search results dropdown, item gallery (1 column, 3 from 768 px), tables, "pill" links (blue for an existing Q, pink for a missing one), buttons (default, grey, danger, small), forms, combo dropdowns, notices (error, ok, warning), cards, tags, and change colors (green up, red down).
* Under 640 px, tables of class `rt` become stacked cards using `data-label` attributes.
* Print mode hides the header, footer, export bar and search.

### 11.2 Script (includes/site.js)
One self-executing script with no dependencies:

* **Live search** with a 200 ms debounce.
* **Combo box** widget (filtering, 7-item limit with "More", keyboard control, strict validation against the list).
* **Dynamic rows** for aliases, images, statements and identifiers, each with a remove button.
* **Image rows** with a live thumbnail and, when uploads are available, an Upload button that posts to `upload.php` with `fetch` and fills in the returned file name.
* Blocks form submission if any property or identifier is not in the allowed list.

The page passes data to the script through the globals `WD_SERVER`, `WD_LABELS`, `WD_IDS`, `WD_INIT`, `WD_UPLOAD`, `WD_CSRF`, `WD_UPLOAD_URL` and `WD_UPDIR`.

## 12. Page and URL reference
* `/` and `/Main_Page` (index.php, public): main page.
* `/Q42`, `/wiki/Q42` and `index.php?title=Q42` (index.php, public): item page.
* `?title=Q42&export=html`, `txt`, `md`, `json`, `aedtpworld` or `pdf` (index.php and export.php, public): download an export.
* `index.php?live=1&q=term` (index.php, public): live-search HTML fragment.
* `/create.php` (public, captcha): new item.
* `/edit.php?title=Q42` (public with captcha; admin only if protected): edit item.
* `/search.php?q=term` (public): search results.
* `/recent.php` (public): recent changes.
* `/upload.php` (admin, or public when enabled): image upload.
* `/dashboard.php` (admin): control panel.
* `/history.php` (admin): backups and restore.
* `/sitemap.xml` and `/sitemap-N.xml` (sitemap.php, public): sitemaps.

## 13. Typical workflows
### 13.1 Add an item
1. Click **Create Item**.
2. Enter the name and description; add aliases and images (URL or upload).
3. Add statements: pick a property, then type a value (text, `Q5`, or a link).
4. Add identifiers: pick one (for example GitHub) and type only the username.
5. Optionally write an edit summary, answer the security question, and press **Save**.

### 13.2 Add a new property or identifier type (admin)
1. Log in at `/dashboard.php`.
2. Under **Properties** add lines such as `date of death`; under **Identifiers** add `Bandcamp | https://bandcamp.com/`.
3. They appear in the form drop-downs immediately.

### 13.3 Allow public image uploads
Dashboard, **Enable Public Upload**, then **Turn ON**. Turn it OFF again to restrict uploads to the admin.

### 13.4 Recover from vandalism
1. Open `/recent.php` (the admin sees full IPs) to find the bad edit.
2. If you saved the item as admin earlier, open `/history.php` and click **Restore**.
3. Optionally **Protect** the item from the dashboard.

### 13.5 Move to another domain
Change `$Server` and `$Logo` in `config.php`, move the folder, then rebuild the sitemap. Stored image names are relative, so uploads keep working; identifiers stored as full URLs are unaffected.

### 13.6 Backup
Copy the whole `wiki/` folder, including `uploads/`. That is the entire database.

## 14. Troubleshooting
* **/Q1 gives a server 404:** `mod_rewrite` or `AllowOverride` is off. Enable it, or set `$route` to "0".
* **Cannot save ("Invalid session"):** session cookies are blocked or the page was open too long. Reload and retry.
* **Dropdown says "No match. Ask the admin to add it.":** the property or identifier is not in `label.txt` or `id.txt`. Add it in the dashboard.
* **Identifier rejected, "enter only the username/ID":** remove the link and paste only the ID.
* **Identifier unusable:** it has no link prefix. Save it again as `Name | https://...`.
* **Upload fails:** uploads are OFF, the file is over 2 MB, it is not a real JPG, PNG or WebP, the rate limit was reached, or PHP `upload_max_filesize` or `post_max_size` is below 2 MB.
* **Sitemap shows dynamic URLs:** the folder is not writable. Fix permissions, then rebuild.
* **Data files downloadable in a browser:** the server ignores `.htaccess`. Add deny rules for `wiki/` in the server configuration.
* **History shows a red size warning:** it is over 500 KB. Delete old backups.

## 15. Glossary
* **Q or Q-id:** unique item identifier such as `Q42`.
* **Item:** a record with name, description, aliases, images, statements and identifiers.
* **Statement or claim:** a property and value pair on an item.
* **Property or label:** a name allowed in statements, listed in `label.txt`.
* **Identifier:** a link to an external database, built from a prefix in `id.txt` and a username or ID.
* **Main_Page:** the special admin-edited HTML home page.
* **Protected:** an item only the admin may edit.
* **AWFL:** AEDTP WORLD FREE LICENSE.
* **AEDTPWORLD export:** the JSON export format with a metadata wrapper.
* **AWFLWIKITEXT:** the lightweight markup language of AWFLWIKIWORLD, used to write this page.

## 16. Credits and license
AWFLWIKIWORLD WIKIDATA is created by **AEDTP WORLD** and distributed under the **AEDTP WORLD FREE LICENSE (AWFL)**. Every exported file carries the line "Source: AWFLWIKIWORLD WIKIDATA - AEDTP WORLD FREE LICENSE (AWFL)".

The license text is not included in the package. According to a published summary of the AWFL (https://bebee.com/gb/blog/aedtpworld/aedtp-world-free-license-awfl-VR4qrN9Ihumv):

* Files may be downloaded for personal use and used in AEDTP WORLD-approved projects.
* Modifications, forks, re-uploads and commercial use require written approval from AEDTP WORLD (Emmanuel Deliver Amable).
* Public use must credit the source as "Content provided by AEDTP WORLD (AWFL)".
* The software is provided as is, without warranty.
* Questions and permission requests go to aedtpworld@gmail.com.

Check https://aedtpworld.com for the current official terms.

## 17. See also
* [AWFLWIKIWORLD](https://aedtpworld.com/wikiworld/index.php/AWFLWIKIWORLD)
* [AWFLWIKIWORLD/AWFLWIKITEXT](https://aedtpworld.com/wikiworld/index.php/AWFLWIKIWORLD/AWFLWIKITEXT)
* [Main Page](https://aedtpworld.com/wikiworld/index.php/Main_Page)
* [AWFLWIKIWORLD on GitHub](https://github.com/aedtpworldawfl/awflwikiworld)
