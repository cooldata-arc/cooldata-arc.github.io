#!/bin/bash

startdate=`date -d "$1" +%Y%m%d`
enddate=`date -d "$2" +%Y%m%d`

while [[ $startdate < $enddate  ]]  
do
    echo "########$startdate#########"  

    startdate=`date -d "+1 day $startdate" +%Y%m%d`
done
