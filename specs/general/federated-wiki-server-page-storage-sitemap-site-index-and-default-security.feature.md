# Federated wiki server: page storage, sitemap, site index and default security

_Extracted from `.`._

The node.js federated wiki server stores pages as JSON flat files and serves them from the store, from default pages or from plugin pages. It moves deleted or unreadable pages to a recycler and keeps a JSON and XML sitemap and a full-text site index up to date. It fills in sane defaults for any missing start options, and its default security module leaves the site read-only.

## S1: A stored page is read from the page store @v1 [proposed]

- Given: a page file with the slug exists in the pages directory
- When: the page is requested by its slug
- Then: the parsed JSON page is returned
- and: it is not marked as coming from a plugin

_Source: `lib/page.js`._

## S2: A missing page falls back to default pages, then plugin pages @v1 [proposed]

- Given: no page with the slug exists in the pages directory
- When: the page is requested
- Then: a page with that name from the configured defaults directory is returned, if one exists
- and: otherwise the server's own default-data pages are tried
- and: otherwise a page shipped in a wiki-plugin's pages directory is returned, with `plugin` set to the plugin name without the `wiki-plugin-` prefix

_Source: `lib/page.js`._

## S3: An unknown page is reported as not found @v1 [proposed]

- Given: the slug is not in the store, in any defaults or in any plugin pages
- When: the page is requested
- Then: the response is `Page not found` with status 404

_Source: `lib/page.js`._

## S4: An unparseable page is moved to the recycler @v1 [proposed]

- Given: a stored page file does not contain valid JSON
- When: the page is requested
- Then: the response is `Error Parsing Page` with status 404
- and: the file is moved into the recycle directory, which is created if it is missing

_Source: `lib/page.js`._

## S5: Deleting a page moves it to the recycler @v1 [proposed]

- Given: a page exists in the store
- When: the page is deleted
- Then: the page file is moved into the recycler directory
- and: deleting a page that does not exist fails with `page does not exist`
- and: deleting a `recycler/` slug removes that file from the recycler for good

_Source: `lib/page.js`._

## S6: Saving a page writes pretty-printed JSON @v1 [proposed]

- Given: a slug and a page object
- When: the page is put
- Then: the page is written as JSON indented by two spaces in the pages directory
- and: any missing directory is created first
- and: reads and writes are handled one at a time, in the order they are received

_Source: `lib/page.js`._

## S7: Sitemap entries record title, last edit, synopsis and links @v1 [proposed]

- Given: a page whose story contains `[[Internal Link]]` text and reference items
- When: the sitemap is updated for that page
- Then: its entry holds the slug, title, synopsis and the date of the last journal action that is not a fork
- and: `links` maps each linked slug to the id of the first item linking to it, and is left out when there are no links
- and: an existing entry for the same slug is replaced, not duplicated

_Source: `lib/sitemap.js`._

## S8: The sitemap is saved as JSON and as XML @v1 [proposed]

- Given: sitemap updates or removals have been processed
- When: the queue is empty
- Then: `sitemap.json` is written to the status directory
- and: `sitemap.xml` lists each page as `<url>/<slug>.html`, with `lastmod` set to the YYYY-MM-DD form of its date when the date is valid

_Source: `lib/sitemap.js`._

## S9: The site index holds searchable text extracted from page stories @v1 [proposed]

- Given: a page with paragraph, markdown, html, code and other text items
- When: the page is added to the site index
- Then: its title and cleaned text are indexed under its slug, with link markup, HTML tags, punctuation and emoji stripped and URLs reduced to their hostname
- and: for audio, video and frame items, lines that start with a URL, `//` or an all-capitals word are left out as markup

_Source: `lib/search.js`._

## S10: The site index is rebuilt at start-up only when needed @v1 [proposed]

- Given: the server starts
- When: there is no `site-index.json`, an `index-updated` flag exists, or the saved index is not serialization version 2
- Then: the index is created again from all pages
- and: the `index-updated` flag is deleted once indexing finishes

_Source: `lib/search.js`._

## S11: Default security makes the site read-only @v1 [proposed]

- Given: the server uses the default security module
- When: authorization or admin rights are checked
- Then: nobody is authorized and nobody is admin
- and: with `security_legacy` on, an unclaimed site can be edited, and admin is granted only when `test` is also set

_Source: `lib/security.js`, `lib/defaultargs.js`._

## S12: Missing start options get sane defaults @v1 [proposed]

- Given: the server is started with no options
- When: the defaults are applied
- Then: the port is 3000, home is `welcome-visitors`, data is `~/.wiki` and the url is `http://localhost:3000`
- and: the upload limit is `5mb`, sessions last 7 days and a random cookie secret is generated
- and: a database type that does not start with `.` gets the prefix `wiki-storage-`, and a non-default security type gets the prefix `wiki-security-`

_Source: `lib/defaultargs.js`._
