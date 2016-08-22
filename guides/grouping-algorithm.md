Overview
--------

Rollbar has two key concepts: *Occurrences* and *Items*.
An *occurrence* is a single event instance: an exception or log message,
along with its associated metadata. Rollbar tries to group occurrences
into *items* that represent occurrences having the same source or cause.

Occurrences can either be:

-   *exceptions*, which have an exception class, exception message, and
    stack trace. or
-   *messages*, which have only a string message

### Goals

Rollbar tries to group occurrences such that:

-   Occurrences with the same root cause are grouped together
-   Occurrences with different root causes are grouped separately
-   The grouping is resilient to deploys and code changes

We do this keeping our basic notifications—new item, reactivated item—in
mind, trying to maximize the signal-to-noise ratio. The goal is that
when you see a notification about a "new item", it really is something
new that you haven't seen before, and that if a new issue starts
occuring, it will result in a new item being created in Rollbar.

When the above goals are in conflict, we generally err on the side of
creating more issues. That is, we think it's better to get a few
redundant notifications than to not realize an issue is happening
because it was lumped together with something else.

### Exception Grouping

For exceptions (including PHP and JavaScript errors), our basic
algorithm is:

1.  Combine the filenames and method names from all of the stack frames
2.  Append the exception class name
3.  Take the SHA1 hash of the result

The resulting SHA1 hash is used as the occurrence "fingerprint".
Occurrences with the same fingerprint are grouped together.

Things to note:

-   We use all of the stack frames, not just the frame from the top of
    the stack. While this sometimes means there will be multiple Items
    for the same root cause, it also prevents unrelated issues from
    being grouped together when they happen to call into the same
    library function.
-   We don't use the line numbers in the stack trace, as they often
    change due to unrelated code changes (i.e. someone added a line of
    code at the beginning of the file).
-   We don't use the exception message, as it often contains input that
    varies for each call.

We've added a number of tweaks to this basic algorithm. Here are some of
the big ones:

-   dates and shas are stripped from paths in filenames
-   integers 2 characters or longer are stripped from method names
-   for JavaScript errors, we attempt to match the error message against
    a known database, to include data from the exception message when
    useful, and to group across browsers. Read more about JavaScript
    grouping in [the blog
    post](https://rollbar.com/blog/post/2013/03/21/improved-grouping-for-javascript-errors/).

### Message Grouping

For messages, we apply a number of heuristics on the message text
itself. The basic algorithm is:

1.  Strip out things that look like data
2.  Take the SHA1 hash of what remains

Like exceptions, the resulting SHA1 hash is used as the occurrence
"fingerprint", and occurrences with the same fingerprint are grouped
together.

"Things that look like data" includes:

-   things that look like timestamps
-   numbers with decimal points
-   integers 2 digits or longer
-   hexadecimals 4 digits of longer

### Customizing Grouping with Global Grouping Rules

If our default algorithm is separating things that you would like to see
grouped together, you can set grouping rules to tell Rollbar to group
them together. This can be found in Settings -\> Grouping. Please see
the [Custom Grouping](/docs/custom-grouping/) guide to
learn more.

### Customizing Grouping in Code

If our default algorithm isn't grouping things the way you want, you can
calculate a fingerprint yourself and send it with the occurrence report.
Occurrences with the same fingerprint will be grouped together. The
fingerprint should be a string up to 40 characters long. If you pass a
string longer than 40 characters, we'll use its SHA1 hash instead.

-   [Browser JS](#fingerprint-browserjs)
-   [Python](#fingerprint-python)
-   [Other](#fingerprint-other)

<div class="tab-content">

<div id="fingerprint-browserjs" class="tab-pane active">

```js
Rollbar.scope({fingerprint: "abcdefg"}).error("Something went wrong");
// This will be grouped with the above
Rollbar.scope({fingerprint: "abcdefg"}).error("Something else went wrong");
```

</div>

<div id="fingerprint-python" class="tab-pane">

```python
# exceptions
try:
  do_something()
except:
  rollbar.report_exc_info(sys.exc_info(), payload_data={'fingerprint': 'abcdefg'})

# messages
rollbar.report_message("Something went wrong", payload_data={'fingerprint': 'hijkl'})
```

</div>

<div id="fingerprint-other" class="tab-pane">

Passing custom fingerprints in the other libraries is not supported yet.
If you want this, please email team@rollbar.com or open an issue (or
pull request!) on the GitHub repo for the appropriate library.

</div>

</div>

### Customizing Grouping via the API

If you're using our API directly to send us data, you can set your own
fingerprint by simply passing it in the JSON payload. For example:

```json
{
  "access_token": "your-post_server_item-access-token",
  "data": {
    "environment": "production",
    "body": {
      "message": {
        "body": "Something went wrong"
      }
    },
    "fingerprint": "abcdefg"
  }
}
```

The fingerprint should be a string up to 40 characters long. If you pass
a longer string, we'll use its SHA1 hash.
