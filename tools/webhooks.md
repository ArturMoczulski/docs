# Webhooks

You can set up webhooks to make Rollbar push data to any arbitrary
external service. Webhooks can be sent for the same triggers as our
other notifications channels:

-   New item (`new_item`)
-   Every occurrence (`occurrence`)
-   Item reactivated (`reactivated_item`)
-   10^nth occurrence (`exp_repeat_item`)
-   High occurrence rate (`item_velocity`)
-   Item resolved (`resolved_item`)
-   Item reopened (`reopened_item`)
-   Deploy (`deploy`)

### Configuration

Configuration is per-project. To set up webhooks, first go to the
Notification settings page for that project: Dashboard -> Settings ->
Notifications -> Webhook.

Enter the full URL where webhooks should be posted and enable the
integration. Once set up, you can add, edit, or remove rules. The main
URL will be used as the default, but you can override by specifying a
different URL on a per-rule basis.

### Expected Response and Retries

The webhook endpoint you provide should respond with a `200` or
`204` response code. If we receive another response code, or the
request times out after 3 seconds, we will retry up to 10 times with
exponential backoff. The first retry is after 1 second, and the 10th
retry is after 16 days (`4 ** num_retries` seconds between
attempts).

You can see the history of each attempt by clicking on the History
button next to each rule on the Webhook Settings page. This will show
all attempts for the selected rule: successes, failures, and pending
retries.

### Payload Format

All payloads are delivered over HTTP/HTTPs as POSTs. Payloads can be
sent in either JSON or XML format. JSON is the default; to use XML,
configure that by editing each rule.

The basic payload format is:

```json
{
    "event_name": "EVENT_NAME",
    "data": {
        "OBJECT_TYPE": {
        }
    }
}
```

EVENT\_NAME will be one of:

-   `new_item`
-   `occurrence`
-   `reactivated_item`
-   `exp_repeat_item`
-   `item_velocity`
-   `resolved_item`
-   `reopened_item`
-   `deploy`

`OBJECT_TYPE` will be one of:

-   `item`(if one of the `*_item events)`
-   `deploy`(if the `deploy` event)

The value of the `OBJECT_TYPE` key will be an object containing all of
the properties of the related item/deploy. This is the same data
returned by our Read API.

`exp_repeat_item` payloads will contain one additional key inside
`data`: `occurrences`, the number of occurrences in the crossed
threshold (e.g. 10, 100, etc.).

`item_velocity` payloads will contain an additional key, `trigger` inside
`data` which holds three additional keys:
`window_size`, the number of seconds in the high occurrence
rate window (e.g. 5, 60, 300, etc.), `window_size_description`, a human-readable
version of the window size (e.g. '100 seconds', '1 hour', etc.) and `threshold`,
how many events were triggered during the window to send the notification.

### Examples

New item (JSON):

```json
{
  "event_name": "new_item",
  "data": {
    "item": {
      "public_item_id": null,
      "integrations_data": {},
      "last_activated_timestamp": 1382655421,
      "unique_occurrences": null,
      "id": 272716944,
      "environment": "production",
      "title": "testing aobg98wrwe",
      "last_occurrence_id": 481761639,
      "last_occurrence_timestamp": 1382655421,
      "platform": 0,
      "first_occurrence_timestamp": 1382655421,
      "project_id": 90,
      "resolved_in_version": null,
      "status": 1,
      "hash": "c595b2ae0af9b397bb6bdafd57104ac4d5f6b382",
      "last_occurrence": {
        "body": {
          "message": {
            "body": "testing aobg98wrwe"
          }
        },
        "uuid": "d2036647-e0b7-4cad-bc98-934831b9b6d1",
        "language": "python",
        "level": "error",
        "timestamp": 1382655421,
        "server": {
          "host": "dev",
          "argv": [
            ""
          ]
        },
        "environment": "production",
        "framework": "unknown",
        "notifier": {
          "version": "0.5.12",
          "name": "pyrollbar"
        },
        "metadata": {
          "access_token": "",
          "debug": {
            "routes": {
              "start_time": 1382212080401,
              "counters": {
                "post_item": 3274122
              }
            }
          },
          "customer_timestamp": 1382655421,
          "api_server_hostname": "web6"
        }
      },
      "framework": 0,
      "total_occurrences": 1,
      "level": 40,
      "counter": 4,
      "first_occurrence_id": 481761639,
      "activating_occurrence_id": 481761639
    }
  }
}
```

Every occurrence (JSON):

```json
{
  "event_name": "new_item",
  "data": {
    "item": {
      "public_item_id": null,
      "integrations_data": {},
      "level_lock": 0,
      "last_activated_timestamp": 1480713852,
      "assigned_user_id": null,
      "hash": "846d53400721434a78442645e0976fc82f009238",
      "id": 418474069,
      "environment": "production",
      "title": "ValueError: Test",
      "last_occurrence_id": 19007601096,
      "last_occurrence_timestamp": 1480713852,
      "platform": 0,
      "first_occurrence_timestamp": 1480713852,
      "project_id": 93341,
      "resolved_in_version": null,
      "status": 1,
      "unique_occurrences": null,
      "title_lock": 0,
      "last_occurrence": {
        "body": {
          "trace": {
            "frames": [
              {
                "filename": "test.py",
                "code": "raise ValueError('Test')",
                "lineno": 5,
                "locals": {
                  "__builtins__": "<type 'module'>",
                  "__file__": "test.py",
                  "__package__": null,
                  "__name__": "__main__",
                  "rollbar": "<type 'module'>",
                  "__doc__": null
                },
                "method": "<module>"
              }
            ],
            "exception": {
              "message": "Test",
              "class": "ValueError"
            }
          }
        },
        "uuid": "5ccf5483-1dfc-453a-aa80-ac857ed5df87",
        "language": "python 2.7.12",
        "level": "error",
        "timestamp": 1480713852,
        "server": {
          "host": "test.local",
          "pid": 11277,
          "argv": ["test.py"]
        },
        "environment": "production",
        "framework": "unknown",
        "notifier": {
          "version": "0.13.8",
          "name": "pyrollbar"
        },
        "metadata": {
          "access_token": "",
          "debug": {
            "routes": {
              "start_time": 1480645795776,
              "counters": {
                "post_item": 475931
              }
            }
          },
        "customer_timestamp": 1480713851,
        "api_server_hostname": "dal05api2",
        "timestamp_ms": 1480713852386
        }
      },
    "framework": 13,
    "total_occurrences": 1,
    "level": 40,
    "counter": 2,
    "last_modified_by": 2147,
    "first_occurrence_id": 19007601096,
    "activating_occurrence_id": 19007601096
    }
  }
}
```

High occurrence rate (JSON):

```json
{
  "event_name": "item_velocity",
  "data": {
    "item": {
      "public_item_id": null,
      "integrations_data": {},
      "last_activated_timestamp": 1382655421,
      "unique_occurrences": null,
      "id": 272716944,
      "environment": "production",
      "title": "testing aobg98wrwe",
      "last_occurrence_id": 481777744,
      "last_occurrence_timestamp": 1382656142,
      "platform": 0,
      "first_occurrence_timestamp": 1382655421,
      "project_id": 90,
      "resolved_in_version": null,
      "status": 1,
      "hash": "c595b2ae0af9b397bb6bdafd57104ac4d5f6b382",
      "last_occurrence": {
        "body": {
          "message": {
            "body": "testing aobg98wrwe"
          }
        },
        "uuid": "fd3a2d6f-3383-42ef-b65f-7d84cfad1b2c",
        "language": "python",
        "level": "error",
        "timestamp": 1382656140,
        "server": {
          "host": "dev",
          "argv": [
            ""
          ]
        },
        "environment": "production",
        "framework": "unknown",
        "notifier": {
          "version": "0.5.12",
          "name": "pyrollbar"
        },
        "metadata": {
          "access_token": "",
          "debug": {
            "routes": {
              "start_time": 1382212089369,
              "counters": {
                "post_item": 3278360
              }
            }
          },
          "customer_timestamp": 1382656140,
          "api_server_hostname": "web5"
        }
      },
      "framework": 0,
      "total_occurrences": 10,
      "level": 40,
      "counter": 4,
      "first_occurrence_id": 481761639,
      "activating_occurrence_id": 481761639
    },
    "occurrences": 10,
    "trigger": {
      "window_size": 300,
      "window_size_description":  "5 minutes",
      "threshold": 100
    }
  }
}
```
10^nth occurrence (JSON):

```json
{
  "event_name": "exp_repeat_item",
  "data": {
    "item": {
      "public_item_id": null,
      "integrations_data": {},
      "last_activated_timestamp": 1382655421,
      "unique_occurrences": null,
      "id": 272716944,
      "environment": "production",
      "title": "testing aobg98wrwe",
      "last_occurrence_id": 481777744,
      "last_occurrence_timestamp": 1382656142,
      "platform": 0,
      "first_occurrence_timestamp": 1382655421,
      "project_id": 90,
      "resolved_in_version": null,
      "status": 1,
      "hash": "c595b2ae0af9b397bb6bdafd57104ac4d5f6b382",
      "last_occurrence": {
        "body": {
          "message": {
            "body": "testing aobg98wrwe"
          }
        },
        "uuid": "fd3a2d6f-3383-42ef-b65f-7d84cfad1b2c",
        "language": "python",
        "level": "error",
        "timestamp": 1382656140,
        "server": {
          "host": "dev",
          "argv": [
            ""
          ]
        },
        "environment": "production",
        "framework": "unknown",
        "notifier": {
          "version": "0.5.12",
          "name": "pyrollbar"
        },
        "metadata": {
          "access_token": "",
          "debug": {
            "routes": {
              "start_time": 1382212089369,
              "counters": {
                "post_item": 3278360
              }
            }
          },
          "customer_timestamp": 1382656140,
          "api_server_hostname": "web5"
        }
      },
      "framework": 0,
      "total_occurrences": 10,
      "level": 40,
      "counter": 4,
      "first_occurrence_id": 481761639,
      "activating_occurrence_id": 481761639
    },
    "occurrences": 10
  }
}
```

Resolved Item (XML):

```xml
<rollbar>
    <event_name>resolved_item</event_name>
    <data>
        <item>
            <public_item_id>None</public_item_id>
            <integrations_data/>
            <last_activated_timestamp>1382655421</last_activated_timestamp>
            <hash>c595b2ae0af9b397bb6bdafd57104ac4d5f6b382</hash>
            <id>272716944</id>
            <environment>production</environment>
            <title>testing aobg98wrwe</title>
            <last_occurrence_id>481777763</last_occurrence_id>
            <last_occurrence_timestamp>1382656142</last_occurrence_timestamp>
            <platform>0</platform>
            <first_occurrence_timestamp>1382655421</first_occurrence_timestamp>
            <project_id>90</project_id>
            <resolved_in_version>None</resolved_in_version>
            <status>2</status>
            <unique_occurrences>None</unique_occurrences>
            <last_occurrence>
                <body>
                    <message>
                        <body>testing aobg98wrwe</body>
                    </message>
                </body>
                <uuid>b5c7ff50-6fff-4ce0-82f8-8a7009227b2e</uuid>
                <language>python</language>
                <level>error</level>
                <timestamp>1382656141</timestamp>
                <server>
                    <host>dev</host>
                    <argv/>
                </server>
                <environment>production</environment>
                <framework>unknown</framework>
                <notifier>
                    <version>0.5.12</version>
                    <name>pyrollbar</name>
                </notifier>
                <metadata>
                    <access_token>REDACTED</access_token>
                    <debug>
                        <routes>
                            <start_time>1382034789251</start_time>
                            <counters>
                                <post_item>4547392</post_item>
                            </counters>
                        </routes>
                    </debug>
                    <customer_timestamp>1382656141</customer_timestamp>
                    <api_server_hostname>web5</api_server_hostname>
                </metadata>
            </last_occurrence>
            <framework>0</framework>
            <total_occurrences>12</total_occurrences>
            <level>40</level>
            <counter>4</counter>
            <first_occurrence_id>481761639</first_occurrence_id>
            <activating_occurrence_id>481761639</activating_occurrence_id>
        </item>
    </data>
</rollbar>
```

Deploy (JSON):

```json
{
  "event_name": "deploy",
  "data": {
    "deploy": {
      "comment": "deploying webs",
      "user_id": 1,
      "finish_time": 1382656039,
      "start_time": 1382656038,
      "id": 187585,
      "environment": "production",
      "project_id": 90,
      "local_username": "brian",
      "revision": "e4b9b7db860b2e5ac799f8c06b9498b71ab270bb"
    }
  }
}
```

------------------------------------------------------------------------

Last updated: March 8, 2017
