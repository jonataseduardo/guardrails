How we style our SQL
====================

Basics
-----------------------------------------------------------------------------

-   Indents should be two spaces.

-   Lines of SQL should not exceed 88 characters.

-   Field names, keywords, and function names should all be in lowercase.

-   The `as` keyword should be explicitly used when aliasing a field or table.

Fields, aggregations, and grouping
---------------------------------------------------------------------------------------------------------------------------------------------------------------

-   Fields should be stated before aggregates and window functions.
-   Aggregations should be executed as early as possible (on the smallest data set possible) before joining to another table to improve performance.
-   Ordering and grouping by a number (e.g. group by 1, 2) is preferred over listing the column names (see [this classic rant](https://www.getdbt.com/blog/write-better-sql-a-defense-of-group-by-1) for why). Note that if you are grouping by more than a few columns, it may be worth revisiting your model design.

-   Prefer `union all` to `union` unless you explicitly want to remove duplicates.
-   If joining two or more tables, *always* prefix your column names with the table name. If only selecting from one table, prefixes are not needed.
-   Be explicit about your join type (i.e. write `inner join` instead of `join`).
-   Avoid table aliases in join conditions (especially initialisms) — it's harder to understand what the table called "c" is as compared to "customers".
-   Always move left to right to make joins easy to reason about - `right joins` often indicate that you should change which table you select `from` and which one you `join` to.
 
-   When performance allows, CTEs should perform a single, logical unit of work.
-   CTE names should be as descriptive as necessary to convey their purpose, for example, `events_joined_to_users` instead of `user_events` (although `user_events` could be a good model name, it does not describe a specific function or transformation).
-   CTEs that are duplicated across models should be extracted into their own intermediate models. Look for repeated logic that should be refactored into separate models.
-   The last line of a model should be a `select *` from the final output CTE. This makes it easy to materialize and review the output from different steps in the model during development. Simply change the CTE referenced in the `select` statement to see the output from that step.






### Example SQL

```sql
    with

    my_data as (

        select
            field_1,
            field_2,
            field_3,
            cancellation_date,
            expiration_date,
            start_date

        from {{ ref('my_data') }}

    ),

    some_cte as (

        select
            id,
            field_4,
            field_5

        from {{ ref('some_cte') }}

    ),

    some_cte_agg as (

        select
            id,
            sum(field_4) as total_field_4,
            max(field_5) as max_field_5

        from some_cte

        group by 1

    ),

    joined as (

        select
            my_data.field_1,
            my_data.field_2,
            my_data.field_3,

            -- use line breaks to visually separate calculations into blocks
            case
                when my_data.cancellation_date is null
                    and my_data.expiration_date is not null
                    then expiration_date
                when my_data.cancellation_date is null
                    then my_data.start_date + 7
                else my_data.cancellation_date
            end as cancellation_date,

            some_cte_agg.total_field_4,
            some_cte_agg.max_field_5

        from my_data

        left join some_cte_agg
            on my_data.id = some_cte_agg.id

        where my_data.field_1 = 'abc' and
            (
                my_data.field_2 = 'def' or
                my_data.field_2 = 'ghi'
            )

        having count(*) > 1

    )

    select * from joined

```
