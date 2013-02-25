---
title: Java properties to enviorment variables
date: '2013-02-25'
description:
categories: Java
---

    #!/bin/bash
 
    property_file=env.properties
 
    get_prop(){
       propfile=$1
       key=$2
       grep  "^${2}=" ${1}| sed "s%${2}=\(.*\)%\1%"
    }
 
    for line in `grep -v "^#" $property_file | sed -e '/^$/d'`
    do
       key=$(echo $line | awk -F "=" '{print $1}')
       export $key=$(get_prop $property_file $key)
    done
