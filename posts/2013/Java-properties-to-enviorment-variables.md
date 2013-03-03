---
title: 将Java的Properties文件转换成环境变量
date: '2013-02-25'
description: Convert Java properties to enviorment variables for shell scripts
permalink: '/2013/02/25/Java%20properties%20to%20enviorment%20variables.html'
categories:
- Blog
tags:
- Java
---

## Overview

在Java程序中使用properties文件很方便，但有时候需要和脚本配合使用时，需要把properties文件内的多个变量转换成环境变量，本文提供一个转换脚本示范：

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
