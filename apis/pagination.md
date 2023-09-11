## Pagination

### Offset Pagination

Achieved using `limit` and `offset` parameters, which are common features in SQL databases.

Pros:
- Simplest strategy to implement (SQL supports `limit` and `offset` out of the box).

Cons (the root for these problems is that we are provided with just `offset` - the number of rows to be dropped - no more context):
- Not performant for large `offset` value.
    - This is due to how the command `OFFSET` works/is implemented in relational databases: with `limit` A and  `offset` B, a database needs to fetch A + B rows, then sorts those rows in the order specified by `ORDER BY` command (if there's any), then drops the first B rows.
- **Page drift** problem: new rows are inserted between fetching 2 pages.

Due to these cons, Offset Pagination does not scale well for large databases.

### Simple Keyset Pagination

This strategy 

### Seek (Cursor) Pagination

The seek method simply doesn’t select already shown values.

Paging requires a deterministic sort order.

Cons:
- you also cannot fetch arbitrary pages.

Keyset pagination provides stable performance regardless of the number of pages we moved forward. To achieve this performance, the paginated query needs an index that covers all the columns in the ORDER BY clause, similarly to the offset pagination.

https://docs.gitlab.com/ee/development/database/keyset_pagination.html
https://www.django-rest-framework.org/api-guide/pagination/#cursorpagination
https://medium.com/swlh/how-to-implement-cursor-pagination-like-a-pro-513140b65f32


