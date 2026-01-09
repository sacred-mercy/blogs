---
layout: post
title: "Syncing Strapi Collections Across Environments"
date: 2026-01-07 23:33:54 +0530
categories: strapi
---

Strapi is a powerful headless CMS, but moving data between development, staging, and production environments can be tricky. Broadly speaking, there are two types of "syncing" you need to handle: **Schema Syncing** (how your content is structured) and **Content Syncing** (the actual data).

## 1. Syncing Schemas (The Easy Part)
The Content-Type Builder’s collections and single types are stored as JSON files within your project’s `src` folder. Because these are flat files, you can sync them easily using Git.

**The Workflow:**
* Ensure your `src` folder is not in your `.gitignore`.
* When you create a new content type in the builder, Strapi generates a corresponding `.json` file.
* Simply commit these files and push them to your target environment. 
* Once the code is deployed and the server restarts, the new content types will be available.

---

## 2. Syncing Content (The REST API Method)
Unlike schemas, content entries are stored directly in the database. To sync these, you need to use the [Strapi REST API](https://docs.strapi.io/cms/api/rest).

When building a sync script, you’ll typically hit an endpoint like: `http://localhost:1337/api/restaurants`.

### The Upsert Logic
Since database auto-increment `id`s can vary between environments, you should never rely on them for syncing. Instead, use a custom **Unique Identifier** (like a `restaurant_code` or `slug`).



**The Step-by-Step Logic:**
1. **Fetch from Source:** Get the content from your source environment.
2. **Check Target:** Hit the target environment with a filter query to see if the entry exists:
   `GET /api/restaurants?filters[restaurant_code][$eq]=MVH232`
3. **Decide Action:**
   * **If found:** Capture the `documentId` from the response. Perform a **PUT** request to `/api/restaurants/{documentId}` to update it.
   * **If not found:** Perform a **POST** request to `/api/restaurants` to create it.

> **Note:** In Strapi 5, the `id` is the local database primary key, while the `documentId` is the unique string used to address entities across the API. Always target the `documentId` for updates.

### Data Cleaning
When sending data to the target API, you must **unset** (remove) specific fields to avoid validation errors:
* `id`
* `documentId`
* `publishedAt`
* `createdAt`
* `updatedAt`

---

## 3. Handling Deeply Nested Components
If your content types use complex components or nested relations, a standard API call might not return all the data you need.

To solve this, I recommend the [Strapi v5 Deep Populate Plugin](https://github.com/tooonuch/strapi-v5-plugin-populate-deep). This allows you to fetch content to any depth level required for your sync.

**Word of Caution:** Deep population can significantly increase response times. For production environments, it is a good idea to cache the JSON responses at the API Gateway level to maintain performance.

---

## Useful Resources
* [Strapi Introduction](https://docs.strapi.io/cms/intro)
* [Content-Type Builder Guide](https://docs.strapi.io/cms/features/content-type-builder)
* [REST API Filters & Population](https://docs.strapi.io/cms/api/rest/filters)