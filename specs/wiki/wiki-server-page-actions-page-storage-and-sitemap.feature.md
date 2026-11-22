# Wiki server page actions, page storage and sitemap

_Extracted from `test`._

The wiki server applies page actions sent to /page/{slug}/action: create, move, add, remove and edit. It records each action in the page journal and refuses to create over a page that already exists. Pages are stored on disk. If a page is not in the store, it is served from the default pages or from plugin pages. The server lists slugs, plugins and factories. It keeps a sitemap that records each page's synopsis and links. Default arguments fill in settings the caller does not give.

## S1: A create action makes a new page @v1 [proposed]

- Given: a running wiki with no page called adsf-test-page
- When: a PUT to /page/adsf-test-page/action sends an action of type create, with a title, a story of four paragraphs and a date
- Then: the response status is 200
- and: /system/slugs.json then lists only adsf-test-page

_Source: `test/server.js`._

## S2: A move action puts the story items in the given order @v1 [proposed]

- Given: a page whose story holds items a1, a2, a3 and a4
- When: a move action with the order a1, a3, a2, a4 is sent to the page's action route
- Then: the response status is 200
- and: the stored story has a3 second and a2 third
- and: the page journal records a move entry

_Source: `test/server.js`._

## S3: An add action inserts an item after the given id @v1 [proposed]

- Given: a page whose story order is a1, a3, a2, a4
- When: an add action with after a2 and a new paragraph item a5 is sent
- Then: the story has 5 items, with a5 directly after a2
- and: the journal records an add entry

_Source: `test/server.js`._

## S4: A remove action deletes the item with the given id @v1 [proposed]

- Given: a page whose story holds a1, a3, a2, a5 and a4
- When: a remove action with id a2 is sent
- Then: the story has 4 items and a2 is gone, so a5 follows a3
- and: the journal records a remove entry

_Source: `test/server.js`._

## S5: An edit action replaces an item in place @v1 [proposed]

- Given: a page whose story contains paragraph a3
- When: an edit action with id a3 and an item whose text is edited is sent
- Then: a3 keeps its position and its text is now edited
- and: the journal records an edit entry

_Source: `test/server.js`._

## S6: An unknown action type is rejected and the page is left unchanged @v1 [proposed]

- Given: an existing page with 4 story items and 5 journal entries
- When: an action of type asdf is sent
- Then: the response status is 500
- and: the story and journal stay as they were

_Source: `test/server.js`._

## S7: Creating over an existing page is refused @v1 [proposed]

- Given: the page adsf-test-page already exists
- When: a create action with the title Doh is sent to that page
- Then: the response status is 409
- and: the stored page title is not Doh

_Source: `test/server.js`._

## S8: A page that is in no store is reported as not found @v1 [proposed]

- Given: the page store, the default pages and the plugin pages hold no page with the slug asked for
- When: that page is requested
- Then: the result is 'Page not found'

_Source: `test/page.js`._

## S9: Pages missing from the store come from the default pages or plugin pages @v1 [proposed]

- Given: the page store holds no welcome-visitors or recent-changes page
- When: those pages are requested
- Then: welcome-visitors is returned from the default pages with the title Welcome Visitors
- and: recent-changes is returned from the activity plugin with the title Recent Changes and plugin set to activity

_Source: `test/page.js`._

## S10: A saved page is written to disk and can be read back @v1 [proposed]

- Given: a page with the title Asdf is saved under the slug asdf
- When: the page asdf is requested
- Then: the page returned has the title Asdf
- and: the file pages/asdf in the data directory holds the same title

_Source: `test/page.js`._

## S11: The sitemap follows page creation, edits and deletion @v1 [proposed]

- Given: a new site whose /system/sitemap.json is an empty list
- When: a page is created whose first paragraph is 'this is the first paragraph' and whose paragraph a3 links to [[third]]
- Then: the sitemap entry for the page has that first paragraph as its synopsis and the links {third: a3}
- and: after an edit to the first paragraph, the synopsis is the edited text
- and: after a DELETE of /adsf-test-page.json returns 200, the sitemap is empty

_Source: `test/sitemap.js`._

## S12: The system endpoints list slugs, plugins and factories @v1 [proposed]

- Given: a running wiki with the activity and video plugins installed
- When: /system/slugs.json, /system/plugins.json and /system/factories.json are requested
- Then: each one answers 200 with JSON
- and: a new site's slug list is empty
- and: the plugin list holds exactly activity and video
- and: the factory list includes Video in the format category

_Source: `test/server.js`._
