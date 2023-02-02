---
title: Let's make waves
parent: Tutorials
layout: default
has_children: true
---

# Let's make waves

Through this tutorial, We are going to cover serveral different ways how to write data into Machbase-neo and read from it.


## Run machbase-neo server

Start machbase-neo server.

```sh
machbase-neo serve
```

## Create example table

Create `EXAMPLE` table for this course if it doesn't exist.

```sh
machbase-neo shell "create tag table EXAMPLE (name varchar(100) primary key, time datetime basetime, value double)"
```

![ex_cre_table](ex_cre_table.gif)

You can delete the table first when you want to create fresh one.

```sh
machbase-neo shell "drop table EXAMPLE"
```

## Make waves - How to write data

- [x] Shell script using `machbase-neo shell` command [🔗](./wave-write-sh.md)
- [ ] Python client program writing data via HTTP API
- [x] Go client program writing data via HTTP API [🔗](./wave-write-go-http.md)
- [x] Go client program writing data via gRPC API [🔗](./wave-write-go-grpc.md)


## Surfing waves - How to query data

- [x] Query table [🔗](./wave-read-sh.md)

## Watch waves - How to visualize data

- [x] Graph on terminal [🔗](./wave-chart-term.md)
- [x] Generate chart HTML [🔗](./wave-chart-genhtml.md)
- [x] Generate chart JSON [🔗](./wave-chart-genjson.md)
- [ ] Using Grapana plugin for Machbase-neo
