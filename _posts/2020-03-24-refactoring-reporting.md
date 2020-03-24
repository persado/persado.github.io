---
layout: post
title: Refactoring our Reporting Section
excerpt: How — and why — we migrated from ActiveRecord and Rollup Tables to Raw SQL and Materialized Views
author: chris_ruenes
---

When I first started working at Persado, there were three main problems with our reporting section:

1. When there were bugs in this section, it often took 1-2 days for a developer to identify and fix them
2. Worst case query time was extremely slow
3. We used a large rollup table to track changes to the aggregate data, which required spaghetti code of Rails callbacks to manage updates

For a while, we made due. But as time went on and we accumulated more and more data, the long execution time for unscoped queries started to cause major performance issues on our production application. This nudged us undertake a comprehensive refactor of the reporting section.

## Step 1: Turn Rollup Table into Materialized Views

Rollup tables and materialized views are both ways of denormalizing data to make that data easier to handle at the aggregate level. Our rollup table represented an aggregation of about 15 other tables, with ~40 distinct columns. Anytime we wrote to one of these dependent tables, we needed to fire off a Rails callback to update the rollup table. Here is an excerpt of a module we included in every related model:

```ruby
included do
  after_save :queue_report_update
end
...
def queue_report_update
  if changes.keys.select { |key| update_triggers.include?(key) }.any?
    self.delay(queue: 'data_sync').update_reports(changes)
  end
end
```

A materialized view, on the other hand, is simply a cached result set from a predefined query. (At the time of this project, we had recently migrated from MySQL to PostgreSQL, which meant that materialized views were a feature newly available to us). Because a materialized view is just a query result set, there is no need to make meticulous updates to it. Simply “refreshing” the materialized view re-runs the query and regenerates the entire result set. The declarative nature of this type of denormalization means that this approach is both sturdy and simple.

Our first step in the refactor process was to rewrite our main rollup table as a collection of materialized views. Anything that could be calculated in a straightforward way went into the top-level materialized view. Anything that required more calculation — especially grouped calculation — we would split out into smaller materialized views which the top-level view could pull from. For example:

```sql
-- top-level mat view
SELECT
CASE
  WHEN results_agg.approved_at IS NULL THEN false
  ELSE true
END AS has_results
FROM shipments
LEFT JOIN results_agg ON results_agg.shipment_id = shipments.id

-- results_agg mat view
SELECT
  uploads.shipment_id,
  min(results.start_date) AS start_date,
  max(results.end_date) AS end_date
FROM results
JOIN uploads ON uploads.id = results.upload_id
JOIN shipments ON uploads.shipment_id = shipments.id
GROUP BY uploads.shipment_id;
```

## Step 2: Convert ActiveRecord to Raw SQL

Our reporting section gives users a ton of autonomy over how they structure their queries, which means there was a lot of branching logic and complexity in our ActiveRecord code. Our choice to move over to raw SQL was not primarily a function of ActiveRecord lacking needed functionality, but rather a function of workflow convenience and code clarity. If there was a bug in the reporting section, our workflow usually required calling `#to_sql` on the final ActiveRecord relation, then copying and pasting the resulting statement into a SQL client like Postico and debugging from there.

While ActiveRecord guarantees efficient SQL statements, it does not guarantee that the statements will be human-readable. This is especially true for queries like ours, which often depended on 2 to 3 layers of subqueries, the columns of which were all assigned terse auto-generated aliases.

So, when we had the opportunity to overhaul this section, we decided that everything would be organized into named methods that lived in service objects and returned raw, formatted SQL. Here is an example of one service's top-level query:

```ruby
def generate_sql
  <<-SQL
    SELECT
      body,
      client_id,
      client_name,
      account_id,
      account_name
    FROM reports
      #{process_joins}
    WHERE
      #{process_filters} AND
      archived_at IS NULL
    #{process_order_params}
    #{process_pagination_params}
  SQL
end
```

The top-level query is formatted as easily-recognizable SQL, and each of the dependent methods returns formatted SQL as well. Again, this change was not primarily because ActiveRecord, as a DSL, seriously lacked the functionality we needed (even though we had to do a bit of monkey-patching that certainly complicated things). It was mostly because our workflow in this section already depended heavily on converting the AR to raw SQL, and the long, unformatted and aliased strings that AR returned were difficult to work with.

## Step 3: Break Complex Subqueries into Materialized Views

Converting everything to raw SQL already revealed some inefficiencies in how we had previously implemented our queries, and we were able to optimize the queries as we converted. However, there was still a major opportunity to make our queries faster. As mentioned above, many of our top level queries were composed of 2-3 layers of nested subqueries, some of which had their own complex and long-running logic. We realized that splitting these subqueries into their own materialized views would both speed up query time immensely and make the final queries still easier to read.

## Drawbacks

Some of the biggest drawbacks to our approach have come along with materialized views. Materialized views represent a snapshot of the data at the time the query was last run. This makes them unworkable in situations where data needs to be available instantaneously. In our reporting section, we came to an agreement with our users about a reasonable refresh interval that kept the data satisfactorily up to date.

Another drawback to materialized views is that the sweeping update to the entire view is much more time- and resource-intensive than the incremental updates required for the rollup table. If a query returns hundreds of thousands or millions of rows, especially if the result needs to be indexed, the refresh can be quite a costly operation. It is possible (and recommended), to run these refreshes with the CONCURRENTLY keyword, which is slower, but prevents any exclusive access locks from being placed during the refresh.

Finally, the fact that this section is in raw SQL does require a bit more specialized knowledge of devs than if it were in ActiveRecord. We found, however, that the previous implementation was sufficiently complex as to cancel most of the ease that ActiveRecord provided.

## Successes

The combined efficiency gains from optimizing the existing queries and cacheing the results of these subqueries in materialized views was pretty exciting. Running a typical query from the frontend now felt nearly instantaneous, and our worst case query time (for those types of queries that prompted the refactor in the first place) was 100x faster.

We have also found that with the raw SQL implementation, it now takes a couple hours instead of a couple days to fix bugs in the reporting section, and a couple days instead of well over a week to implement new features.

One of the nicest things about materialized views is that they are declarative, so it is now easy to see exactly how the result set was generated. This means that the risk of data falling out of sync in our rollup table is completely eliminated by the materialized view.
