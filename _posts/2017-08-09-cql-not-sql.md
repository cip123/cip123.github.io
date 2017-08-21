---
layout: post
title: Why CQL is not SQL ?
date: 2017-08-09
categories: cassandra
---


So you start working on a project that uses Cassandra. 
You hear that Cassandra is a NoSQL database that uses CQL but you see a couple of query and create table statements which look very familiar. Let's start with the usual: 


```sql
CREATE TABLE songs(
   id int PRIMARY KEY,
   artist text,
   title text,
   album text,
   track_number int,
   seconds int
 );
```

So far so good, we can insert a couple of songs.  
```sql
INSERT INTO songs (id, artist, title, album, track_number, seconds) VALUES(1, 'AC/DC', 'The Jack', 'T.N.T', 1, 353);
INSERT INTO songs (id, artist, title, album, track_number, seconds) VALUES(2, 'Motörhead', 'Damage Case', 'Overkill', 7, 179);
INSERT INTO songs (id, artist, title, album, track_number, seconds) VALUES(3, 'Motörhead', 'Metropolis', 'Overkill', 9, 214);
INSERT INTO songs (id, artist, title, album, track_number, seconds) VALUES(4, 'AC/DC', 'Live Wire', 'High Voltage', 4, 350);
```

Now let's select all AC/DC songs:

```sql
SELECT * FROM songs where artist='AC/DC';
```  
Didn't work !

<span style="color:red">If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING</span>

Well, let's pretend we resisted the temptation to just add ALLOW FILTERING there and really understand what's happening.

The thing with Cassandra is that it is designed to be distributed. So, those records are possibly distributed like this in a 4 nodes cluster:

![image-title-here](/assets/img/cassandra-songs-first-layout.png){:class="img-responsive"}


What happens is that Cassandra uses the primary key (`id` in our case) to select the node where to store the data. 

So `ALLOW FILTERING` means that cassandra has to take all the records from each node, see if their artist is in fact *AC/DC* and then return the result. For very large amounts of data, like Cassandra was designed to handle, this can become problematic.

Other options? Well, we could add an index, but this is not really a solution. Will talk about indexes later. A better way to do it is to put a little more effort in designing how the data will be stored.  

Let's aim for two Cassandra partitions, one for AC/DC and the other one for Motörhead. And in the meantime we need to find a primary key which guarantees unicity.


### Partition keys and clustering keys


The primary key syntax in Cassandra is special. The first position of the primary key is the partition key which is used to select the cassandra node. The rest of the elements comprise the clustering key. 

Let's start with an example :

```sql
CREATE TABLE songs(
   id int,
   artist text,
   title text,
   album text,
   track_number int,
   seconds int,
   PRIMARY KEY((artist), album, track_number)
 );
```

Cassandra-tools ships with a nice utility `SSTablesDump` which lets you see how the previous data is actually stored.







```json
[
  {
    "partition" : {
      "key" : [ "AC/DC" ],
      "position" : 0
    },
    "rows" : [
      {
        "type" : "row",
        "position" : 64,
        "clustering" : [ "High Voltage", "4" ],
        "liveness_info" : { "tstamp" : "2017-08-13T11:37:02.278647Z" },
        "cells" : [
          { "name" : "id", "value" : "4" },
          { "name" : "seconds", "value" : "350" },
          { "name" : "title", "value" : "Live Wire" }
        ]
      },
      {
        "type" : "row",
        "position" : 64,
        "clustering" : [ "T.N.T", "1" ],
        "liveness_info" : { "tstamp" : "2017-08-13T11:37:02.259084Z" },
        "cells" : [
          { "name" : "id", "value" : "1" },
          { "name" : "seconds", "value" : "353" },
          { "name" : "title", "value" : "The Jack" }
        ]
      }
    ]
  },
  {
    "partition" : {
      "key" : [ "Motörhead" ],
      "position" : 100
    },
    "rows" : [
      {
        "type" : "row",
        "position" : 166,
        "clustering" : [ "Overkill", "7" ],
        "liveness_info" : { "tstamp" : "2017-08-13T11:37:02.265368Z" },
        "cells" : [
          { "name" : "id", "value" : "2" },
          { "name" : "seconds", "value" : "179" },
          { "name" : "title", "value" : "Damage Case" }
        ]
      },
      {
        "type" : "row",
        "position" : 166,
        "clustering" : [ "Overkill", "9" ],
        "liveness_info" : { "tstamp" : "2017-08-13T11:37:02.270004Z" },
        "cells" : [
          { "name" : "id", "value" : "3" },
          { "name" : "seconds", "value" : "214" },
          { "name" : "title", "value" : "Metropolis" }
        ]
      }
    ]
  }
]
```




