# Simple NoSQL Movies Project
Technologies: Bash shell scripting (couchimport, mongoimport/export), IBM Cloudant, Cassandra, MongoDB.

Migrate data between NoSQL databases Cloudant, MongoDB, and Cassandra using shell.

All bash commands have been consolidated into one .sh file for viewing convenience.

Non-Partitioned Replicated movies IBM Cloudant database:
![non-partitioned replicated movies cloudant database](https://user-images.githubusercontent.com/88465305/172264085-a675c3c9-4a91-4831-8887-1f6a98d00fe8.PNG)

```
#!/bin/bash
# setup:
# installs couchimport and mongoimport/mongoexport.
npm install -g couchimport 
wget https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu1804-x86_64-100.3.1.tgz 
tar -xf mongodb-database-tools-ubuntu1804-x86_64-100.3.1.tgz 
export PATH=$PATH:/home/project/mongodb-database-tools-ubuntu1804-x86_64-100.3.1/bin 

# sets CLOUDANTURL environment variable:
export CLOUDANTURL="cloudant_url_here"


# IBM Cloudant:
# exports movie.json file into cloudant database "movies".
curl -XPOST $CLOUDANTURL/movies/_bulk_docs -Hcontent-type:application/json -d @movie.json

# creates index for "director" and "title" in "movies" database using HTTP API.
curl -X POST $CLOUDANTURL/movies/_index -H"Content-Type: application/json" -d'{"index":{"fields":["director"]}}' && curl -X POST $CLOUDANTURL/movies/_index -H"Content-Type: application/json" -d'{"index":{"fields":["title"]}}'

# exports data from "movies" database into a "movies.json" file.
couchexport --url $CLOUDANTURL --db movies --type jsonl > movies.json


# MongoDB:
# imports movies.json into MongoDB server, database "entertainment" collection "movies".
mongoimport -u root -p made_up_password --authenticationDatabase admin --db entertainment --collection movies --file movies.json

# mongo query which finds count of movies released after 1999.
echo 'db.movies.find({"year":{"$gt":1999}}).count()' | mongo -u root -p made_up_password --authenticationDatabase admin local

# exports fields _id, title, year, rating, and director from "movies" collection into a csv.
mongoexport -u root -p made_up_password --authenticationDatabase admin --db entertainment --collection movies --out partial_data.csv --type=csv --fields _id,title,year,rating,director


# Cassandra:
# creates entertainment keyspace.
echo "CREATE KEYSPACE IF NOT EXISTS entertainment WITH REPLICATION = {'class':'SimpleStrategy', 'replication_factor':3};" | cqlsh --username cassandra --password made_up_password

# creates movies table in entertainment keyspace.
echo "CREATE TABLE IF NOT EXISTS entertainment.movies(id INT PRIMARY KEY, title VARCHAR, year INT, rating VARCHAR, director VARCHAR);" | cqlsh --username cassandra --password made_up_password

# imports partial_data.csv into the created Cassandra keyspace/table.
echo "COPY entertainment.movies(id,title,year,rating,director) FROM 'partial_data.csv' WITH DELIMITER=',' AND HEADER=TRUE;" | cqlsh --username cassandra --password made_up_password

```
