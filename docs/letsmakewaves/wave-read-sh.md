---
title: Read waves by shell
parent: Let's make waves
layout: default
order: 30
nav_order: 101
---

# Reading data by shell

## SQL Query

The simplest way to print out data using "SQL query".

```sh
machbase-neo shell "select * from EXAMPLE order by time desc"
```
```
 #     NAME      TIME(UTC)            VALUE
────────────────────────────────────────────────
 1     wave.sin  2023-01-28 14:03:59  0.214839
 2     wave.cos  2023-01-28 14:03:59  -0.976649
 3     wave.sin  2023-01-28 14:03:58  0.593504
 4     wave.cos  2023-01-28 14:03:58  -0.804831
  ...
```

![img](../img/shell-sql.gif)

We executed query by `machbase-neo shell` without `sql` sub-command above example.
It properly printed out result of query which is becuase machbase-neo shell takes `sql` sub-command as default as long as there are no other arguments and flags. This means `machbase-neo shell "select..."` is same with `machbase-neo shell sql "select..."`.

So when we use some flags for executing query, explicitly sepcifiy `sql` subcommand like below.

```sh
machbase-neo shell sql \
    --tz America/Los_Angeles \
    "select * from EXAMPLE order by time desc limit 4"
```

```
 #  NAME      TIME(AMERICA/LOS_ANGELES)  VALUE
───────────────────────────────────────────────────
 1  wave.sin  2023-01-28 06:03:59        0.214839
 2  wave.cos  2023-01-28 06:03:59        -0.976649
 3  wave.cos  2023-01-28 06:03:58        -0.804831
 4  wave.sin  2023-01-28 06:03:58        0.593504
```

Machbase treats all time data in UTC as default.
Use `--tz` option to display time in any time-zone other than 'UTC' like above example. 
This flag accepts 'local' and tz database format (eg: 'Europe/Paris').

```sh
machbase-neo shell sql \
    --tz local \
    "select * from EXAMPLE order by time desc limit 4"
```
```
 #  NAME      TIME(LOCAL)          VALUE
─────────────────────────────────────────────
 1  wave.sin  2023-01-28 23:03:59  0.214839
 2  wave.cos  2023-01-28 23:03:59  -0.976649
 3  wave.cos  2023-01-28 23:03:58  -0.804831
 4  wave.sin  2023-01-28 23:03:58  0.593504
 ```

## Table view

It is also possible browsing query result forward/backward with "walk" command like below.

```sh
machbase-neo shell walk "select * from EXAMPLE order by time desc"
```

Then you can scroll up/down with keyboard, press `ESC` to exit table view.

Press `r` to re-execute query to refresh result, it is particularly useful with query was sorted by `order by time desc` to see the latest values when data is continuously being written.

![img](../img/shell-walk.gif)

## Query Output format

### JSON

Use `--format json` option

```sh
machbase-neo shell sql \
    --format json \
    "select * from EXAMPLE order by time desc limit 4"
```

```json
{
  "data": {
    "columns": ["ROWNUM","NAME","TIME(UTC)","VALUE"],
    "types": ["string","string","datetime","double"],
    "rows": [
      [1,"wave.sin","2023-01-28 14:03:59",0.214839],
      [2,"wave.cos","2023-01-28 14:03:59",-0.976649],
      [3,"wave.cos","2023-01-28 14:03:58",-0.804831],
      [4,"wave.sin","2023-01-28 14:03:58",0.593504]
    ]
  }
}
```

### CSV

Use `--format csv` option

```sh
machbase-neo shell sql 
    --format csv \
    "select * from EXAMPLE order by time desc limit 4"
```

```
#,NAME,TIME(UTC),VALUE
1,wave.sin,2023-01-28 14:03:59,0.214839
2,wave.cos,2023-01-28 14:03:59,-0.976649
3,wave.cos,2023-01-28 14:03:58,-0.804831
4,wave.sin,2023-01-28 14:03:58,0.593504
```

Use `--no-heading` option to exclude the first line header 

```sh
machbase-neo shell sql \
    --format csv \
    --no-heading \
    "select * from EXAMPLE order by time desc limit 4"
```

```
1,wave.sin,2023-01-28 14:03:59,0.214839
2,wave.cos,2023-01-28 14:03:59,-0.976649
3,wave.cos,2023-01-28 14:03:58,-0.804831
4,wave.sin,2023-01-28 14:03:58,0.593504
```

## Query Time format

Use `--timeformat` option to sepcify time output format.

Execute `help timeformat` to display pre-defined formats and syntax for custom format.

```sh
machbase-neo shell help timeformat
```

### Pre-defined timeformats

| Name          | Format                                    |
|:--------------|:------------------------------------------|
| Default,-     |    2006-01-02 15:04:05.999                |
| ns, us, ms, s | (UNIX epoch in nano-, mili-, micro-, seconds as int64) |
| Numeric       |    01/02 03:04:05PM '06 -0700             |
| RFC822        |    02 Jan 06 15:04 MST                    |
| RFC850        |    Monday, 02-Jan-06 15:04:05 MST         |
| RFC3339       |    2006-01-02T15:04:05Z07:00              |
| Kitchen       |    3:04:05PM                              |
| Stamp         |    Jan _2 15:04:05                        |
| ...(there are more)...       | Please consult `machbase-neo shell help timeformat` |

Try `--timeformat numeric` format.

```sh
machbase-neo shell sql \
    --timeformat numeric \
    "select * from example where name='wave.sin' order by time desc limit 1"
```

```
 #  NAME      TIME(UTC)                   VALUE
───────────────────────────────────────────────────
 1  wave.sin  01/28 02:03:59PM '23 +0000  0.214839
```

`-t` is a shorten alias of `--timeformat`

```sh
machbase-neo shell sql \
    -t ms \
    "select * from example where name='wave.sin' order by time desc limit 1"
```

```
 #  NAME      TIME(UTC)      VALUE
──────────────────────────────────────
 1  wave.sin  1674914639000  0.214839
```

### Custom time format

It is also possible your own custom format.

```sh
machbase-neo shell sql \
    --timeformat "2006.01.02 (15:04:05.000)" \
    "select * from example where name='wave.sin' order by time desc limit 1"
```

```
 #  NAME      TIME(UTC)                    VALUE
────────────────────────────────────────────────────
 1  wave.sin  "2023.01.28 (14:03:59.000)"  0.214839
```

| Value      | Symbol                                    |
|:-----------|:------------------------------------------|
| year       | 2006                                      |
| month      | 01                                        |
| day        | 02                                        |
| hour       | 03 or 15                                  |
| minute     | 04                                        |
| second     | 05 or with sub-seconds '05.999' or '05.000'|


### Combine time format and time zone

```sh
machbase-neo shell sql \
    --tz Europe/Paris \
    --timeformat "2006.01.02 (15:04:05.000)" \
    "select * from example where name='wave.sin' order by time desc limit 1"
```

```
 #  NAME      TIME(EUROPE/PARIS)           VALUE
────────────────────────────────────────────────────
 1  wave.sin  "2023.01.28 (15:03:59.000)"  0.214839
```

{: .note }
> Since `s`,`ms`,`us` and `ns`  formats are represents UNIX epoch time. 
> If one of these formats are used, `--tz` option is ignored.
> Becuase epoch time is always in UTC.

