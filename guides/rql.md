# RQL

Rollbar Query Language (RQL)  provides a rich SQL-like
interface to the data in Rollbar.

RQL supports running `SELECT` queries on two logical tables,
`item_occurrence` and `deploy`. Basic `GROUP BY`, `ORDER BY`, `LIMIT`, and
aggregation functions are available, as are arbitrary expressions in the `WHERE`
clause.

### Syntax

`SELECT`, `FROM`, and `WHERE` are required.
`GROUP BY`, `ORDER BY`, and `LIMIT` are optional.

`SELECT *` may be used as long as there is no `GROUP BY` clause. It will
return a list of columns similar to the Occurrences tab on Item Detail
pages.

You cannot `GROUP BY` a function. You *can* reference a column by its index in a `GROUP BY`, for example `GROUP BY 1`.

SQL keywords and built-in function names are case-insensitive (i.e.
`SELECT` and `select` are both fine).

Column names should start with a lowercase letter and may
contain letters, numbers, and periods (for specifying a JSON path). If
you need any other characters (i.e. a hyphen, or to start with an
uppercase letter), escape with backticks (i.e.
`` `request.headers.User-Agent` ``)

#### Operators

-   `+`, `-`, `/`, `*`, `DIV`, `MOD`
-   `AND`, `OR`, `NOT`
-   `=`, `!=`,
    `<>`, `>`, `>=`, `<`, `<=`, `IN`, `NOT IN`, `BETWEEN`,
    `NOT BETWEEN`, `LIKE`, `NOT LIKE`

#### Built-in Functions

-   `count(*)`: counts all rows
-   `count(foo)`: counts rows where foo is not null
-   `count_distinct(foo)`: counts distinct rows where foo is not null
-   `sum(foo)`: sums value of foo (for rows where not null)
-   `avg(foo)`: average value of foo (for rows where not null)
-   `min(foo)`: minimum value of foo
-   `max(foo)`: maximum value of foo
-   `unix_timestamp()`: returns the current Unix timestamp, as an
    integer
-   `concat(str1, str2, ...)`: returns the string resulting from
    concatenating all arguments
-   `concat_ws(sep, str1, str2, ...)`: returns the string resulting from
    concatenating the second argument and beyond, separated by the first
    argument
-   `lower(str)`: converts str to lowercase
-   `upper(str)`: converts str to uppercase
-   `left(str, len)`: returns the len leftmost characters of str
-   `right(str, len)`: returns the len rightmost characters of str
-   `substring(str, pos)`: returns the substring from str starting at
    pos (all characters from pos until the end)
-   `substring(str, pos, len)`: returns a substring from str starting at
    pos, at most len characters
-   `locate(substr, str)`: returns the position of the first occurrence
    of substr in str. (1-indexed)
-   `locate(substr, str, pos)`: returns the position of the first
    occurrence of substr in str, starting the search at position pos
-   `char_length(str)`: returns the length of str in characters
-   `length(str)`: returns the length of str in bytes

### Examples

To find all occurrences of item #47, grouped by request.user_ip with the total count, earliest timestamp, and most recent timestamp, ordered by total count descending and limited to the top 10 rows:

```sql
SELECT request.user_ip, min(timestamp), max(timestamp), count(*)
FROM item_occurrence
WHERE item.counter = 47
GROUP BY request.user_ip
ORDER BY count(*) DESC
LIMIT 10
```

To see the timestamp and message for all occurrences for Items #40 to #50:

```sql
SELECT timestamp, body.message.body
FROM item_occurrence
WHERE item.counter BETWEEN 40 AND 50
```

To grab all the occurrences of Items #1, 2, and 3:

```sql
SELECT *
FROM item_occurrence
WHERE item.counter IN (1,2,3)
```

To find items that affected the most IPs in the last 3 days, grouped by the item number and ordered by the distinct count of the number of user_ips descending:

```sql
SELECT item.counter
FROM item_occurrence
WHERE timestamp > unix_timestamp() - 60 * 60 * 24 * 3
GROUP BY item.counter
ORDER BY count_distinct(request.user_ip) desc
```

To see all the items with the User-Agent string "python-requests/2.9.1" in a date range, ordered by timestamps descending:

```sql
SELECT * 
FROM item_occurrence 
WHERE `request.headers.User-Agent` = "python-requests/2.9.1"  
AND timestamp BETWEEN 1507566716 and 1510245116 
ORDER BY timestamp desc
```

To grab all the referrer domains an Item #1234 came from, grouped by the domain and limited to the top 1000 rows:

```sql
SELECT substring(request.headers.Referer, locate('.', request.headers.Referer), locate('.', request.headers.Referer, locate('.', request.headers.Referer)) - locate('.', request.headers.Referer) - 1)
FROM item_occurrence
WHERE item.counter = 1234
GROUP BY 1
LIMIT 1000
```

To find all the occurrences of items that occurred on Safari version 9.x in the last day:

```sql
SELECT *
FROM item_occurrence
WHERE client.javascript.browser LIKE '%Version/9.%.% Safari/%'
AND timestamp > unix_timestamp() - 60 * 60 * 24
```

### Tips

-   Use `SELECT * FROM item_occurrence` to get a display similar to the
    Occurrences tab
-   For better performance, filter by item (i.e.
    `WHERE item.counter = 123`) or by timestamp (i.e.
    `WHERE timestamp > unix_timestamp() - 86400`)
-   RQL will sometimes timeout unless you put a date range, such as `WHERE timestamp > unix_timestamp() - 86400`.
-   When using `GROUP BY` or `ORDER BY`, make sure the group/order clause is
    also present in the `SELECT` clause.
-   You cannot `GROUP BY` a function. 
-   You *can* reference a column by its index in a `GROUP BY`, for example `GROUP BY 1`. 
-   You can share the URL with a co-worker and they'll see the same
    results you do, without having to run the query again.
-   After a query has completed, press `Execute` again to re-run it.

### Limitations

-   At most 1000 rows will be returned per query (though any number of
    rows may be examined)
-   No `DISTINCT`, `HAVING`, subqueries, joins, or unions
-   No `ANY`, `ALL`, `EXISTS`
-   `SELECT *` cannot be combined with `GROUP BY`
-   You cannot `GROUP BY` a function
-   Can only be used to examine data within a single project

### Schema

#### item\_occurrence

`item_occurrence` is a table where each row contains data for a single
occurrence and the item it is associated with. Column names starting with "item."
reference the item, and all other column names reference the occurrence.
Column names that do not exist in a particular occurrence evaluate to NULL.

The following columns exist for every row in `item_occurrence`:

| Name | Description
|-|-
| `item.id` | System-wide Item ID
| `item.counter` | Project-wide Item ID
| `item.environment` | Environment name
| `item.platform` | [Platform ID](#platform-ids)
| `item.framework` | [Framework ID](#framework-ids)
| `item.hash` | Computed fingerprint of the item (controls [grouping](/docs/grouping-algorithm/))
| `item.first_occurrence_id` | ID of the first occurrence
| `item.first_occurrence_timestamp` | Timestamp of the first occurrence
| `item.activating_occurrence_id` | ID of the first occurrence since the item was last resolved
| `item.last_activated_timestamp` | Timestamp the item was last activated
| `item.last_resolved_timestamp` | Timestamp the item was last resolved
| `item.last_muted_timestamp` | Timestamp the item was last muted
| `item.last_occurrence_id` | ID of the most recent occurrence
| `item.last_occurrence_timestamp` | Timestamp of the most recent occurrence
| `item.last_modified_by` | ID of the user who last modified this item
| `item.level` | Item level (50=critical, 40=error, 30=warning, 20=info, 10=debug)
| `item.resolved_in_version` | The revision the item was last marked as resolved in
| `item.status` | Status (as an integer: 1=active, 2=resolved, 3=muted)
| `item.title` | Computed title
| `item.total_occurrences` | The number of occurrences since the last time this item was resolved
| `occurrence_id` | System-wide Occurrence ID
| `timestamp` | Timestamp of the occurrence, as a Unix timestamp

Many virtual columns will usually exist as well, depending on which Rollbar SDK you are using
and what custom data you are sending. Simply use the JSON path to the field
you want to query. To see the JSON structure of one of your occurrences, click the
"Raw JSON" button on an Occurrence page. The structure will follow the
[Rollbar API Item Schema](/docs/api/items_post/).

Here are some common column names, all of which refer to data for the occurrence:

| Name | Description
|-|-
| `body.crash_report.raw` | The raw crash report (if the occurrence is a crash report)
| `body.message.body` | The primary message text (if the occurrence is a message)
| `body.message.foo` | Any arbitrary keys of metadata you sent (if the occurrence is a message)
| `body.trace.exception.class` | The exception class name (if the occurrence is a single exception)
| `body.trace.exception.message` | The exception message (if the occurrence is a single exception)
| `body.trace_chain.0.exception.class` | The first exception class (if the occurrence is a list of nested exceptions)
| `body.trace_chain.0.exception.message` | The first exception message (if the occurrence is a list of nested exceptions)
| `client.javascript.browser` | Raw user agent string (from [rollbar.js](/docs/notifier/rollbar.js/))
| `client.javascript.code_version` | The running code version in JavaScript
| `client.javascript.source_map_enabled` | Whether or not source map deobfuscation is enabled
| `client.javascript.guess_uncaught_frames` | Whether or not frame guessing is enabled
| `client.runtime_ms` | How long the page was open before the event occurred ([rollbar.js](/docs/notifier/rollbar.js/))
| `code_version` | The version of the application code
| `context` | An identifier for which part of your application the error came from
| `custom.foo` | Arbitrary metadata you sent
| `custom.foo.bar` | Nested arbitrary metadata you sent
| `language` | The name of the reported language for the event
| `notifier.name` | Name of the library that sent the item
| `notifier.version` | The version string of the library that sent the item
| `person.id` | A string identifying the user in your system
| `person.username` | A username string
| `person.email` | An email string
| `request.url` | Full URL where the error occurred
| `request.method` | The request method
| `request.headers` | Object containing the request headers
| `request.params` | Any routing parameters
| `request.GET` | Query string parameters
| `request.query_string` | The raw query string
| `request.POST` | POST parameters
| `request.body` | The raw POST body
| `request.user_ip` | The end user's IP address as a string
| `server.host` | The server hostname
| `server.root` | Path to the application code root
| `server.branch` | Name of the checked-out source control branch
| `server.code_version` | String describing the running code version on the server
| `uuid` | A string that uniquely identifies the occurrence. See [UUIDs](/docs/uuids/)

#### deploy

`deploy` is a table where each row represents a single deploy. It has
the following columns:


| Name | Description
|-|-
| `id` | System-wide ID
| `user_id` | Rollbar user\_id of the rollbar\_username recorded for the deploy
| `environment` | Name of the deployed environment
| `revision` | Revision (i.e. git sha or version number) deployed
| `local_username` | Local username recorded for the deploy
| `comment` | The deploy comment
| `timestamp` | Timestamp when the deploy was recorded
    
### Framework IDs

The `item.framework` property is an integer value which maps as follows:

```
    'unknown': 0,
    'rails': 1,
    'django': 2,
    'pyramid': 3,
    'node-js': 4,
    'pylons': 5,
    'php': 6,
    'browser-js': 7,
    'rollbar-system': 8,  # system messages, like "over rate limit"
    'android': 9,
    'ios': 10,
    'mailgun': 11,
    'logentries': 12,
    'python': 13,
    'ruby': 14,
    'sidekiq': 15,
    'flask': 16,
    'celery': 17,
    'rq': 18,
```

### Platform IDs

The `item.platform` property is an integer value which maps as follows:

```
    'unknown': 0,
    'browser': 1,
    'flash': 2,
    'android': 3,
    'ios': 4,
    'heroku': 5,
    'google-app-engine': 6,
    'client': 7,
```

### Future Improvements

-   `DISTINCT`, `HAVING` clauses
-   More performance optimizations
-   More functions
-   Better progress indicators
-   Saved searches
-   More visualization options (i.e. bar graphs, line graphs, etc.)
-   Download results as CSV/JSON dump (with support for large
    resultsets)
-   More data tables
