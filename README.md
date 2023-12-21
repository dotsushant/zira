# Overview

**ZIRA** stands for "Zeppelin and Impala for Reporting and Analytics".


To enable this, data from SQL Server is exported to **HDFS** using **Sqoop** and processed using **Impala** which is a high performant SQL processing engine. The sqoop jobs can be scheduled to run regularly and would only get the incremental updates.

# Getting Started 
----------
## Create SQL views ##
First of all we need to create view in SQL Server to ensure we have all the tables with dAudit field and get all the valid columns.

    Use the v_elt.sql to allow the sqoop to fetch the schema.

## Install Ubuntu 16.04 LTS ##
Install the Ubuntu 16.04 LTS in the Virtualbox from the official website.
  
## Pull latest image from docker hub (Cloudera) ##
    sudo docker pull cloudera/quickstart:latest 
    sudo docker run --hostname=quickstart.cloudera --privileged=true -t -i -p 8080:8080 -p 10000:10000 -p 21050:21050 -p 8888:8888 -p 80:80 -p 7180:7180 cloudera/quickstart /usr/bin/docker-quickstart

## Pull latest image from docker hub (Zeppelin) ##
    sudo docker pull apache/zeppelin:0.7.2
	sudo docker run --privileged=true -t -i -p 8081:8080 --name zeppelin apache/zeppelin:0.7.2

## Initialize the container (Cloudera) ##
    sudo docker exec -t -i <cloudera_docker_container_name> /bin/bash
    cd /home/cloudera
    mkdir reporting
    cd reporting
    scp <username>@<source_ip>:<driver_directory>/sqljdbc_6.0.8112.100_enu/sqljdbc41.jar /var/lib/sqoop
    sqoop import --connect "jdbc:sqlserver://<sql_server_ip>:1433;username=<username>;password=<password>;database=<database>" --table v_elt --delete-target-dir --target-dir /user/hdfs/<instanceid> -m 1
    hadoop fs -cat /user/hdfs/<instanceid>/part-m-00000 | awk -F"," '{if(a[$1]){a[$1]=a[$1]","$2} else {a[$1]=$2}} END {for (i in a) { print "sqoop job --create import"i" -- import --table "i" --columns "a[i]" --connect \"jdbc:sqlserver://<sql_server_ip>:1433;username=<username>;password=<password>;database=<database>\" --target-dir /user/hdfs/<instanceid>/"i" --check-column dAudit --append --incremental append -m 1 -as-avrodatafile; sqoop job --exec import"i"; hadoop fs -copyFromLocal "i".avsc /user/hdfs/<instanceid>/; impala-shell -q \"create external table "i"_ STORED AS AVRO LOCATION '\''/user/hdfs/<instanceid>/"i"'\'' TBLPROPERTIES ('\''avro.schema.url'\''='\''/user/hdfs/<instanceid>/"i".avsc'\'')\";" > "outputfile"}}'
    cat outputfile | while read line; do eval $line; done;

## Initialize the container (Zeppelin) ##
    Url: jdbc:impala://<impala_server_ip>:21050
    Driver class: com.cloudera.impala.jdbc4.Driver
	sudo docker exec -t -i <zeppelin_container_name> /bin/bash
	cd /interpreter/jdbc
    scp <username>@<source_ip>:<driver_directory>/cloudera_impalajdbc4_2.5.5.1007/*.jar .
	exit
	sudo docker restart <zeppelin_container_name>
## Managing cloudera using HUE ##
	http://<cloudera_server_ip>:8888 where cloudera_server_ip is the IP address of the machine hosting the cloudera docker container.
## Managing notebooks using Zeppelin ##
	http://<zeppelin_server_ip>:8081 where zeppelin_server_ip is the IP address of the machine hosting the zeppelin docker container.
