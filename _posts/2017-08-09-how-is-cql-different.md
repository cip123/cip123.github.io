---
layout: post
title: How is CQL different?
date: 2017-08-09
categories: cassandra
---


So you start working on a project that uses Cassandra. 
You hear that Cassandra is a NoSQL __distributed__ database that uses CQL, but you see a couple of query and create table statements which look very familiar. 

Let's start with the usual, the way we would have done it in SQL: 


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

So far so good, let's add some songs.  
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
Auch! This query fails but, at least, it gives us the following message:

<span style="color:red">If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING</span>

Well, let's try to resist the temptation to just add `ALLOW FILTERING` there and really understand what's happening.

The thing with Cassandra is that it is designed to be distributed. So, those records are possibly distributed like this in a 4 node cluster:

![cassandra-nodes-first-layout](/assets/img/cassandra-songs-first-layout.png){:class="img-responsive"}

What happens is that Cassandra uses the primary key (`id` in our case) to select the node where to store and where to read the data from. 

So `ALLOW FILTERING` means that cassandra has to take all the records from each node, see if their artist is in fact AC/DC and then return the result. For multiple distant nodes with very large amounts of data, this can become problematic.

Other options? Well, we could add an index, but this is not really a solution. Will talk about indexes later. A better way to do it is to put a little more effort in designing how the data will be stored.  

Since we want to query by artist, let's aim for two Cassandra partitions, one for AC/DC and the other one for Motörhead. Cassandra should just compute the hash of the `artist` and then select the node where to perform the query.

There is one problem though, if we use just the `artist` as the primary key, some records will be overwritten, more precisely song `4` will overwrite song `1` and song `3` will overwrite song `2` because they are played the same `artist`.

### Partition keys and clustering keys

The primary key syntax in Cassandra is special. The first position of the primary key is the `partition key` which is used to select the cassandra node. The rest of the elements comprise the `clustering key`. 

You can think of the clustering key as the primary key inside the partition. 



#### Simple clustering key
Let's exemplify with `artist` at the partitioning key and `album` as the clustering key.


```sql
CREATE TABLE songs(
   id int,
   artist text,
   title text,
   album text,
   track_number int,
   seconds int,
   PRIMARY KEY(artist, album)
 );
```

Here how the same 4 records are stored with this new layout. 

![cassandra-nodes-with-duplicate-primary-key](/assets/img/cassandra-nodes-primary-key-not-unique.png){:class="img-responsive"}

The good news is that now we can do queries like:

```sql
SELECT * FROM songs where artist='AC/DC';
```  

and even:


```sql
SELECT * FROM songs where artist='AC/DC' AND album='T.N.T';
```  

The bad news is that we have only one Motörhead record, because the two songs share the same album. 


So, the trick is that if you are going to filter data, you should always specify the partioning key (or keys, you can have a composite partitioning key) so that Cassandra will know which node to talk to. Then, if you want, you can add some clustering keys to the where clause.


#### Composite clustering key

Let's wrap this up with the final example which will hopefully help the CQL model to sink in. 

As you may have noticed, we could use the `track_number` along with the `album` to uniquely identify a song. We can use then a composite clustering key.

```sql
CREATE TABLE songs(
   id int,
   artist text,
   title text,
   album text,
   track_number int,
   seconds int,
   PRIMARY KEY(artist, album, track_number)
 );
```

So inserting the same 4 records will be distributed like this. 


![cassandra-nodes-with-unique-primary-key](/assets/img/cassandra-nodes-primary-unique.png){:class="img-responsive"}

Now, we can do all the previous queries + one. 

```sql
SELECT * FROM songs where artist='AC/DC';
SELECT * FROM songs where artist='AC/DC' AND album='T.N.T';
SELECT * FROM songs where artist='AC/DC' AND album='T.N.T' AND track_number=1;
```  
Every key that we add to the clustering key can be added to the `WHERE` clause, but you have to make sure you specified all the clustering keys that were declared before it.


### The actual files 

Often, diagrams are not enough. We want to see for ourselves how the data is actually stored. 

Fortunately, Cassandra ships with a nice utility called `SSTablesDump` which lets you see the exact contents of the files: 

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

That was it, thanks for reading so far, you can always reach me for questions and feedback.




