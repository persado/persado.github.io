---
layout: post
title: Enforcing NOT NULL Constraints on Boolean Columns
excerpt: An example of how we keep our database tidy with database-level tests.
author: polina_vulakh
---

Database cleanup is by definition continuous, but we can enforce certain guidelines to minimize future work. We'll walk through adding a database-level test that ensures we have no boolean columns that unintentinally allow nulls.

### Why are NULL values considered bad in boolean fields?

Boolean columns should be binary systems where values can either be `true` or `false`. By allowing nulls, we end up with a ternary system `(true, false, NULL)`. A boolean column like this is actually a nice, inexpensive way to express a ternary system. However, typically there isn't a ternary system to express: the reason we are allowing nulls here is an oversite that leads to unnecessarily ambiguous data.

With this psuedo-ternary field, we can end up with bugs that slip through the cracks. Imagine code that checks whether a boolean field is `false`, but overlooks entities where the field is `null`. We also end up with SQL code that has to be more verbose:

```sql
SELECT *
FROM table_name
WHERE nullable_bool_column = 'f' OR nullable_bool_column IS NULL;
```

Our application has a fair amount of raw SQL (see how we [refactored our reporting section](https://persado.github.io/2020/03/24/refactoring-reporting.html) for an example of why we choose to do this), so the more readable we can make our queries, the better for everyone involved.

### What does cleanup of this type look like?
Many years into our application's history, we ended up with 94 boolean columns that allowed nulls. In order to determine which of these fields should continue allowing null values, and which fields can be safely converted to having a not-null constraint, a lot of research needed to be done before mass-updating these columns.

- Which of these fields have null values in our production system?
- For a given field, is a null value ever intentionally being set?
- Do we explicitly look for entities with a null value in any of these fields anywhere in our app?

For fields that should have a not-null constraint (in our case, this was all of them!), we also needed to ensure that everywhere these fields were explicitly being set, the values coming in from manual entry, file uploads, or the like were modified to be not null.

### Adding a test to enforce this for us

To save future developers the headache of having to do this again several years down the line, we added a database-level test that checks whether we have any boolean columns that allow nulls. If this test fails, the culpable code cannot be merged in. For context, we use PostgreSQL and RSpec.

First, we find all boolean columns that are nullable. We specify the `table_type` to exclude any duplicates coming from materialized views. Then we make sure that this `nullable_columns` result is empty. We make room for exceptions in case our application does one day contain a ternary system that is best expressed through a nullable boolean column.

```ruby
describe "DB" do
  it "does not have boolean columns that allow nulls" do
    nullable_columns_sql = <<-SQL
      SELECT information_schema.columns.table_name, column_name
      FROM information_schema.columns
      JOIN information_schema.tables ON information_schema.tables.table_name = information_schema.columns.table_name
      WHERE table_type = 'BASE TABLE'
        AND information_schema.columns.table_schema = 'public'
        AND data_type = 'boolean'
        AND is_nullable = 'YES'
      ORDER BY table_name;
    SQL

    nullable_columns = ApplicationRecord.connection.select_all(nullable_columns_sql).rows
    exceptions = [] # Add exceptions here of the form [table_name, column_name], although ideally there should be none.
    expect(nullable_columns - exceptions).to eq([])
  end
end
```

This can be expanded to other database-level integrity tests, such as making sure that every table has `created_at` and `updated_at` columns.