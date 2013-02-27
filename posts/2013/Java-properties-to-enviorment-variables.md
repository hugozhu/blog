---
title: Java properties to enviorment variables
date: '2013-02-25'
description:
categories:
- Blog
tags:
- Java
---

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

    grep -v "^#" $property_file | sed -e '/^$/d' | while read line
    do
        key=$(echo $line | awk -F "=" '{print $1}')
        trimmed_key=$(trim $key)
        export $trimmed_key=$(trim $(get_prop $property_file "$key"))
        echo $trimmed_key=${!trimmed_key}
    done
