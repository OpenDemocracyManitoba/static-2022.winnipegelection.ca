# 2022.winnipegelection.ca (static archive)

Static HTML copy of **2022.winnipegelection.ca**, Open Democracy Manitoba's guide to the 2022 Winnipeg municipal and school board elections. It ran on WordPress 5.9.19, with the ElectionData-V2 theme supplied by the Inform-Your-Vote-Wordpress-Plugin. It was crawled from olddroplet (`odm-nginx-ruby-php-2018`, 159.203.24.194) on 2026-10-24, before the server was retired.

* `site/` holds the crawl: 1,650 files in the raw crawl commit.
* `crawl.log.gz` holds the wget log.

The WordPress source, the DB dump (`2022_winnipegelection_ca`) and the nginx vhost are in the private repo `OpenDemocracyManitoba/archive-2022.winnipegelection.ca`.

## How it was crawled

The crawl ran from a workstation against the origin directly (an `/etc/hosts` entry, bypassing Cloudflare). wp-cron was disabled and the site was frozen first.

```
wget --mirror --page-requisites --adjust-extension --restrict-file-names=windows \
     --no-host-directories --directory-prefix=site \
     -e robots=off --wait=0.1 --user-agent='ODM-archiver/1.0' \
     --reject-regex='(/wp-json|[?&]rest_route=|xmlrpc\.php|wp-login\.php|/wp-admin|[?&]replytocom=|/feed/?$|/trackback/?$|[?&]s=|/candidates/[^/]+/[0-9]+/|[?&]p=[0-9]+)' \
     -o crawl.log https://2022.winnipegelection.ca/
```

* It ran on 2026-10-24, 12:43–12:51 CDT, and downloaded 1,647 files. wget exited 8 because some URLs returned 404.
* The last two reject patterns skip a crawler trap and WordPress `/?p=N` shortlinks:
  * `/candidates/<slug>/<N>/` is WordPress's path form of a paged candidate page. See the 2016 archive for the details. This site's pager uses its own `pagenum` parameter, so nothing links to that form here.
  * `/?p=N` shortlinks are duplicates of pages saved at their real paths, so they aren't archived (owner decision, 2026-10-24).
* There is no `--convert-links`, so the saved HTML keeps its original absolute `https://2022.winnipegelection.ca/…` hrefs. The pagers use root-relative ones.
* `wp-content/debug.log` is not linked from anywhere and is not in the crawl.

Three files are referenced only where wget doesn't look, so they were copied in from the source tree:

* `wp-content/uploads/2022/05/cropped-cropped-favicon-1024-270x270.png` (icon `<meta>` tags);
* `wp-includes/js/wp-emoji-release.min.js` (inline JS);
* `wp-content/themes/ElectionData/ElectionData-V2/js/html5.js` (an IE conditional comment).

On the server, `wp-content/themes/ElectionData` is a symlink to `wp-content/plugins/Inform-Your-Vote-Wordpress-Plugin/theme/`, so `html5.js` was taken from there.

## Changes made on the live site before the crawl

The DB wasn't modified. The private archive repo has the untouched source and dump.

* **wp-cron disabled** (`DISABLE_WP_CRON`, and nginx returns 403 for `/wp-cron.php`). This stopped the news scraper, which had kept running until July 2023.
* **News cutoff** (owner decision, 2026-10-24). The mu-plugin `wp-content/mu-plugins/odm-archive-news-cutoff.php` hides `ed_news_articles` published after **2022-12-26 23:59:59**, two months after election day (2022-10-26). The full news table is still in the DB dump.

No paging fix was needed: this site's news pager uses its own `pagenum` query variable, which WordPress doesn't redirect. There was no page cache.

## Completeness

|Check|Result|
|-|-|
|Candidate pages|164 of 164 published `ed_candidates`. `candidates/804/` is a real candidate (Brad Gross) whose slug is a number.|
|Paginated candidates|41, with 421 extra news pages. None missing, none identical to page 1.|
|News|1,130 of the 1,140 approved news articles up to the cutoff appear on some page (details below). None after the cutoff appear.|
|Questionnaire answers|387 of 387 question/answer pairs, on 101 candidate pages. The DB has 387 non-empty `ed_answers` out of 656, the rest being empty placeholders.|
|Uploads|Every referenced `wp-content/uploads/` file on this host is in the crawl. The crawl holds 946 of the 1,288 files in the source uploads tree; the rest aren't linked from any page.|
|Asset CDNs|no `i0.wp.com` / `c0.wp.com` references|

**News.** There is no paginated news archive on this site: `news_articles/` shows the latest 90 items, and older news is reachable only through the candidate pages.

The plugin moderates news, and only `approved` articles were ever shown. Of the 2,575 published articles, 1,435 are approved; 1,062 are `new` and 78 are `rejected`. 1,141 approved articles fall on or before the cutoff, and 1,140 of those mention a candidate.

1,130 of them appear in the crawl. The other 10 couldn't be reached on the live site either. The cause is in the theme's `display_news_summaries()`, and **it's archived as is** (owner decision, 2026-10-24): the pager computes `total = round(count / 10)`, so a candidate's last page is never linked when it would hold fewer than 5 articles. For example, Brian Mayes has 91 articles but 9 pages, and Cindy Gilroy has 64 but 6. The same function skips an article that has no summary for that candidate, so some pages can show fewer than 10 items.

The candidate pages carry the theme's notice "Showing news articles published after May 1st 2022". That's only text: `get_news()` has no date filter.

**Address lookup.** The ward-finder widget (`wp-content/plugins/Wpg-Council-Address-Lookup-Vue-App/`) looks addresses up in the browser against the City of Winnipeg's open data API (`https://data.winnipeg.ca/resource/cam2-ii3u.json`). It needs nothing from this site's server, so it keeps working as long as the City keeps that dataset.

**404s:** only `wp-content/themes/ElectionData/ElectionData-V2/ipad4/arrow.gif` and `arrowl.gif`, which are referenced by the theme CSS and missing on the live site too.

## Links to other hosts

**Links to `https://www.winnipegelection.ca/…`, the old host.** When this was the current election site, it was served as www.winnipegelection.ca, and some links still use that host. www.winnipegelection.ca is now a separate landing page that doesn't have these paths, so **the links already 404 on the live site**. Every target is in this archive under the same path:

* `frequently-asked-questions/` (and its slashless copies, including `about-winnipegelection-ca/frequently-asked-questions.html`) links to `/about-winnipegelection-ca/#odm` and `/frequently-asked-questions/`.
* `learn-about-elections/` links to `/constituencies/{councillor,mayoral,trustee}-candidates/`.
* `constituencies/mayoral-candidates/` links to `/learn-about-elections/#what_mayor`.
* The address-lookup widget's JavaScript links each school-trustee ward to `/constituencies/<ward>/`: 26 wards (`louis-riel-1`…`4`, `pembina-trails-1`…`3`, `river-east-transcona-1`…`4`, `seine-river-1`, `seven-oaks-2`, `seven-oaks-3`, `st-james-assiniboia-{centre,east,west}-ward`, `winnipeg-1`…`9`).

These are listed for the hosting phase to rewrite, as with 2016. Phase A doesn't edit content.

Other cross-site links, left as they are:

* `learn-about-elections/` links to `http://2018.winnipegelection.ca/constituencies/{councillor,mayoral,trustee}-candidates/`, which are in the 2018 static archive.
* `about-winnipegelection-ca/` links to `http://www.manitobaelection.ca/press-mentions/` and `/press-releases/` (ODM's press pages on the provincial site), and to `http://manitobaelection.ca` and `http://winnipegelection.ca`. `press-mentions/` (and its slashless copy) also link to `http://winnipegelection.ca`.
* `press/WinnipegElection2011By-ElectionPressRelease.pdf`, an old press release, contains links to `http://winnipegelection.ca/about` and `/press`.

There were no references to `upcoming.manitobaelection.ca` or `159.203.24.194`.

## wget quirk, left as is

`frequently-asked-questions.html`, `press-mentions.html` and `press-releases.html` at the root are wget's saves of slashless links (e.g. `/press-releases`). The server 301'd each one to the slash form, and wget saved the redirect target under the slashless name. Static hosting with `auto-trailing-slash` serves `/press-releases` from `press-releases.html`.

`about-winnipegelection-ca/frequently-asked-questions.html` is the same quirk with a twist: a relative link on the About page requested `/about-winnipegelection-ca/frequently-asked-questions`, WordPress 301'd it to `/frequently-asked-questions/`, and wget saved that page under the requested name.

## Commits and URL mapping

1. **Raw crawl.** `--restrict-file-names=windows` saved each URL's `?` as `@`.

## Secret scan

The Phase A §7.2 scan was run over `site/` and found nothing. It checks for:

* no `.git`, `.env`, `.ftpconfig`, `.htpasswd`, `debug.log`, `*.php` or `*.sql*` files;
* no `DB_PASSWORD`, `AUTH_KEY`, `SECURE_AUTH`, private keys, `phpinfo()` or `mysql(i)_connect` in any text file;
* no `*.php.html` pages.
