---
title: 'SQLite自增关键字sqlite(near AUTO_INCREMENT: syntax error)'
date: 2016-08-08 16:05:24
tags: [sqlite,error]
categories: sqlite
---
SQLite中的自增关键字：AUTO_INCREMENT、INTEGER PRIMARY KEY与AUTOINCREMENT

SQLite不支持关键字AUTO_INCREMENT

<!-- more -->

AUTO_INCREMENT不生效的问题

在SQLite中，自增字段需要使用关键字INTEGER PRIMARY KEY

buffer.append("CREATE TABLE userIntegralDetail (_id INTEGER PRIMARY KEY,")
        .append("userid char(32),")
        .append("Num char(11),")
        .append("Type char(11),")
        .append("timestamp char(13))");
db.execSQL(buffer.toString());
