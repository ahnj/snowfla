---
title: "snowsql CLI example config"
date: 2020-10-28T19:55:14-07:00
draft: false

description: "Quickstart command-line interface to snowsql"
featured_image: "/images/snowsql.png"
tags: ["snowsql", "config", "-c"]

---

macOS Snowsql CLI installation and setup in 3 easy steps:

# Install brew

Install brew via one copy-paste from [here](https://brew.sh/)

# Install snowsql binary 

```bash
brew cask install snowflake-snowsql
```

# Configure snowsql

```bash
vi ~/.snowsql/config
```

follow example config [file](https://gist.github.com/ahnj/8d90056deb2860b001a51fadcab03635)

{{< gist ahnj 8d90056deb2860b001a51fadcab03635 >}}

# test and launch

```bash
$ snowsql                 # default set of configurations

$ snowsql -c training     # training set of configurations
```
