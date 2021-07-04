---
title: Turn Airtable into a Scalable CMS with Sync Inc and Next.js

excerpt: private file - admin access only

status: private

tags: tutorial, cms, nextjs, airtable, syncinc

cover: ./images/000.png

published: 1625492152
---

# Notion API + Next.js

[Getting Started](https://developers.notion.com/docs/getting-started)

Create New [Integration](https://www.notion.so/my-integrations)

![001](https://cdn.jsdelivr.net/gh/gaurangrshah/_shots@master/scrnshots/001.png)

- _Be sure to select the `internal integration`option._

Create new page / database or share an existing one with the new integration![002](/Users/bunty/Downloads/002.png)

    1. Click `Share` on the top right,
    2. then click `Invite` from the dropdown menu

And then you can select the name of your integration from the list.

![003](/Users/bunty/Downloads/003.png)

Lastly, click invite to complete access to the integration.

![004](/Users/bunty/Downloads/004.png)

When successful you'll notice that your integration is listed with _can edit_ access.

![005](/Users/bunty/Downloads/005.png)

```
# .env.local

NOTION_SECRET=
NOTION_DATABSE_ID=
```

```js
// /lib/notion.js

// @link: https://dev.to/geobrodas/how-to-use-notion-api-with-nextjs-5940#making-a-new-integration
import { Client } from "@notionhq/client";

export const notion = new Client({ auth: process.env.NOTION_SECRET });

const database_id = process.env.NOTION_DATABSE_ID;

export const listDatabases = async () => {
  const res = await notion.databases.list();
  console.log(res);
};

export const getPages = async () => {
  const payload = {
    path: `databases/${database_id}/query`,
    method: "POST",
  };

  const { results } = await notion.request(payload);
  return results;
};

export const getPage = async (pageId) => {
  const { results } = await notion.pages.retrieve({ page_id: pageId });
  return results;
};

export const getBlocks = async (blockId) => {
  const { results } = await notion.blocks.children.list({
    block_id: blockId,
    page_size: 10,
  });
  return results;
};

export const ndb = async (config = {}) =>
  // https://developers.notion.com/reference/post-database-query
  await notion.databases.query({
    database_id,
    ...config,
  });
```

```js
// pages/api/notion/dbs.js

import { listDatabases } from "@/lib/notion";

export default async function handler(req, res) {
  if (req?.method === "GET") {
    console.log("🚀 ~ file: index.js ~ line 4 ~ handler ~ req", req.auth);
    // @SCOPE:  Get a list of all notion pages
    return res.json({ data: await listDatabases() });
  }
  return res
    .status(403)
    .json({ error: { message: "sorry this action is not allowed." } });
}
```

```js
// pages/api/notion/pages/index.js

import { getPages } from "@/lib/notion";

export default async function handler(req, res) {
  if (req?.method === "GET") {
    // @SCOPE:  Get a list of all notion pages
    return res.json({ data: await getPages() });
  }

  res
    .status(403)
    .json({ error: { message: "sorry this action is not allowed." } });
}
```

```js
// pages/api/notion/pages/[id].js

import { getPage } from "@/lib/notion";

export default async function handler(req, res) {
  if (req?.method === "GET") {
    // @SCOPE:  Get a list of all notion pages
    return res.json({ data: await getPage(req.params.id) });
  }

  res
    .status(403)
    .json({ error: { message: "sorry this action is not allowed." } });
}
```

```js
// pages/api/notion/content/[id].js

import { getBlocks } from "@/lib/notion";

export default async function handler(req, res) {
  if (req?.method === "GET") {
    // @SCOPE:  Get a list of all notion Blocks
    return res.json({ data: await getBlocks(req.params.id) });
  }

  res
    .status(403)
    .json({ error: { message: "sorry this action is not allowed." } });
}
```
