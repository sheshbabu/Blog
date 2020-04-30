---
title: Minimal Viable Search using Postgres
description: Minimal Viable Search using Postgres
keywords: Postgres, Search, ElasticSearch, MVP
date: 2019-12-01 19:40:19
tags:
  - Postgres
  - Search
---

If you’re building a product, you might have deprioritized building the search feature thinking that it might take a long time to build. If you happen to be using Postgres, let me show you a quick and easy way to implement the search functionality.

## Test drive

Let’s say you’re building an ecommerce app and you want to be able to search on the product descriptions. This can be done using the following query:

```sql
SELECT
  *
FROM
  products
WHERE
  to_tsvector(description) @@ websearch_to_tsquery('chocolate milk') = TRUE
```

If you have a test database lying around, you can quickly try this out by replacing the table name, column name and search query. If you're using Postgres 10 or below, "websearch_to_tsquery" won't work. use "plainto_tsquery" instead.

Now, you might be having a lot of questions like:

- "to_tsvector", "websearch_to_tsquery", "@@" look weird!
- How's this different from "LIKE"?
- How to make this faster?
- What are the tradeoffs compared to ElasticSearch?

## ts_what?

"ts" stands for Text Search.

At the very minimum, you need to only learn four things:

- Use "to_tsvector" function on the columns you're searching on
- Use "websearch_to_tsquery" function for the search query
- Use the match operator "@@" to see if the above two match
- Use "ts_rank" function to sort the results based on relevancy

In simple terms, `to_tsvector` breaks down text into list of keywords and their positions.

Running:

```sql
SELECT to_tsvector('A journey of a thousand miles begins with a single step');
```

gives

```sql
'begin':7 'journey':2 'mile':6 'singl':10 'step':11 'thousand':5
```

Notice that the [words](https://en.wikipedia.org/wiki/Stop_words) "A", "of" and "with" are removed as they're not useful in searching, the word "single" is normalized to its root form "singl" so it appears in more searches, the word "miles" is reduced to its singular form. This also takes care of normalizing the text to lowercase and removing special characters.

The function `websearch_to_tsquery` converts the user submitted search term into something that Postgres can understand. You can use Google style search queries like

```sql
jaguar speed -car

ipad OR iphone

"chocolate chip" recipe
```

You can also try other query functions like "plainto_tsquery" or "phraseto_tsquery" which have their own way of parsing the search queries.

The `@@` operator matches the above search query with text from column. You can also use the `||` operator to concatenate multiple columns together and search on them.

The function ts_rank is used for sorting the search results by relevancy. The way it determines relevancy is by looking at how frequent the search terms appear, how close together they appear, in what position they appear etc.

By now you should have a good idea about how this is different from normal LIKE or pattern matching.

## Making it faster

Instead of building tsvectors everytime we query using to_tsvector, we can store it in a separate column when the record is created/updated. For this, we create the following trigger:

```sql
CREATE OR REPLACE FUNCTION fn_on_product_insert_store_tsv() RETURNS trigger AS
$$
BEGIN
  NEW.tsv := to_tsvector(NEW.description);
  return NEW;
END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER trg_on_product_insert_store_tsv
BEFORE INSERT OR UPDATE ON products
FOR EACH ROW
EXECUTE PROCEDURE fn_on_product_insert_create_tsv();
```

Let's also add an index on this column to make the queries faster:

```sql
CREATE INDEX tsv_idx ON products USING gin(tsv);
```

This should greatly speed up your seach queries.

## Comparison to ElasticSearch

ElasticSearch is synonymous with product search these days so you need to be aware of the tradeoffs:

When Postgres is better than ElasticSearch:

- One less dependency to manage or get approval for
- Faster time to market - can see how your users are using search and decide if you need to use ElasticSearch for more sophisticated search features
- There's a single source of truth for the data - no need to keep multiple datastores in sync

When ElasticSearch is better than Postgres:

- If your team already has expertise in ElasticSearch
- Scale search queries seperately from normal database queries
- You need support for [facets](https://stackoverflow.com/questions/5321595/what-is-faceted-search). Here's a [simple implementation](https://roamanalytics.com/2019/04/16/faceted-search-with-postgres-using-tsvector/) of facets in Postgres
- More flexible and sophisticated search features
