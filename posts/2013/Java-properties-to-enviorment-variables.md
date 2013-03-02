---
title: Java properties to enviorment variables
date: '2013-02-25'
description:
permalink: '/2013/02/25/Java%20properties%20to%20enviorment%20variables.html'
categories:
- Blog
tags:
- Java
---

将Java的Properties文件导出成环境变量
====

比如env.properties如下（=附近可以有空格，也可以有空行）

    MYSQL_URL = jdbc:mysql://localhost:3306/test?autoReconnect=true&useUnicode=true&characterEncoding=gbk
    MYSQL_USER = root
    MYSQL_PASS = 

执行下面的脚本后就相当于

    export MYSQL_URL="//localhost:3306/test?autoReconnect=true&useUnicode=true&characterEncoding=gbk"
    export MYSQL_USER="root"
    export MYSQL_PASS="" 

env.sh脚本代码

    #!/bin/bash

    property_file=env.properties

    get_prop(){
        propfile=$1
        key=$2
        grep  "^${2}=" ${1}| sed "s%${2}=\(.*\)%\1%"
    }
    
    trim() {
        trimmed=$1
        trimmed=${trimmed%% }
        trimmed=${trimmed## }
        echo "$trimmed"
    }

    `grep -v "^#" $property_file | sed -e '/^$/d' | while read line
    do
        key=$(echo $line | awk -F "=" '{print $1}')
        trimmed_key=$(trim $key)
        trimmed_val=$(trim $(get_prop $property_file "$key")
        echo "export $trimmed_key=\"$trimmed_val\")"
    done`
