---
title: User data formats in TQL
parent: Let's make waves
layout: default
order: 30
nav_order: 13
---

# User data formats in TQL
{: .no_toc}

1. TOC
{: toc }

## Decode JSON

It is required to tranform the incoming JSON data that is sent by external system to process into machbase-neo. Using TQL script makes it possible without developing or modifying exsting applictions.

Prepare test data saved in "script-data.json".

```json
{
  "tag": "pump",
  "data": {
    "string": "Hello TQL?",
    "number": "123.456",
    "time": 1687405320,
    "boolean": true
  },
  "array": ["elements", 234.567, 345.678, false]
}
```

Copy the code below into TQL editor and save `script-post-json.tql`.

```js
INPUT( BYTES(CTX.Body) )
SCRIPT({
  json := import("json")
  ctx := import("context")
  val := ctx.value()
  obj := json.decode(val[0])
  // parse a value from json, yield multiple records
  ctx.yieldKey(obj.tag+"_0", obj.data.time*1000000000, obj.data.number)
  ctx.yieldKey(obj.tag+"_1", obj.data.time*1000000000, obj.array[1])
  ctx.yieldKey(obj.tag+"_2", obj.data.time*1000000000, obj.array[2])
})
OUTPUT( CSV() )
```

Post the test data JSON to the tql.

```sh
curl -o - --data-binary @script-data.json http://127.0.0.1:5654/db/tql/script-post-json.tql
```

This example takes a json object via HTTP POST and decode with JSON decoder then produces records by  calling `context.yield()` multiple times.

```sh
$ curl -o - --data-binary @script-data.json http://127.0.0.1:5654/db/tql/script-post-json.tql
name-0,1687405320000000000,123.456
name-1,1687405320000000000,234.567000
name-2,1687405320000000000,345.678000
```

If the script ends with `OUTPUT( APPEND(...) )` or `OUTPUT( INSERT(...) )` instead of `OUTPUT( CSV() )` the final result records will be written into database.

# Decode text

Make test data in 'script-post-lines.txt'.

```
11111
22222
33333
44444
```

Copy the code below into TQL editor and save as `script-post-lines.tql`.
```js
// Produce a {key:lineno, value:string} record per line
INPUT( STRING(CTX.Body, delimiter('\n')) )
SCRIPT({
  text := import("text")
  times := import("times")
  ctx := import("context")
  key := ctx.key()
  values := ctx.value()
  str := text.trim_space(values[0])
  if len(str) == 0 {
    ctx.drop() // ignore empty line
  } else { // parsing
    str = text.substr(str, 0, 2)
    ctx.yieldKey(
      "text_"+key,                // new key
      times.now(),                // time
      text.parse_int(str, 10, 64) // convert to int
    )
  }
})
OUTPUT( CSV() )
```

Send the test data to the *tql* via HTTP POST.

```sh
curl -o - --data-binary @script-post-lines.txt http://127.0.0.1:5654/db/tql/script-post-lines.tql
```

```sh
$ curl -o - --data-binary @script-post-lines.txt http://127.0.0.1:5654/db/tql/script-post-lines.tql
text_0,1687476301286716000,11
text_1,1687476301286750000,22
text_2,1687476301286791000,33
text_3,1687476301286853000,44
```

If the script ends with `OUTPUT( APPEND(...) )` or `OUTPUT( INSERT(...) )` instead of `OUTPUT( CSV() )` the final result records will be written into database.