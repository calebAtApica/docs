---
description: >-
  This document describes the heuristic used by LOGIQ for managing timestamps in
  incoming log data
---

# Timestamp handling

## Which timestamp to use?

Incoming log data streams can have timestamps defined in the following ways

1. Sending agent sends a timestamp
2. Log data has its own timestamp
3. Ingest layer for e.g. LOGIQ adds its own timestamp
4. Log data has non-standard timestamp format

LOGIQ handles the timestamps in the below order

1. User sending agent timestamp
2. Extract any timestamp in log data automatically that is non-ambiguous
3. If a user-defined timestamp is provided, use the extraction rule to get the timestamp 

## User-defined timestamp rules

Users can specify timestamp extraction rules for log data using the LOGIQ data manipulation capabilities. Timestamps are handled by timestamp rules that are defined as follows

```text
- name: custom_timestamp
  namespaces: http.*
  applications: .*
  type: timestamp
  format: "(?P<year>[0-9]{1,4})/(?P<month>[0-9]{1,2})/(?P<day>[0-9]{1,2}) (?P<hour>[0-9]{1,2}):(?P<minute>[0-9]{1,2}):(?P<seconds>[0-9]{1,2}).((?P<milliseconds>[0-9]{1,3})|(?P<microseconds>[0-9]{1,6}))(?P<offset>[+-])((?P<timezone_hour>[0-9]{1,2}):(?P<timezone_minute>[0-9]{1,2}))"
```

Using the example rule above, a logline such as the one below will result in the proper timestamp being parsed and extracted from the log line.  

```text
"2009/06/29 13:30:10.956+05:30 something interesting happened"
```

 

