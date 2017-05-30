Title: mongodb oplog
Date: 2017-5-31 10:20
Category: mongodb

## Replication Internals
Replication gives you hot backups, read scaling, and all sorts of other goodness. If you know how it works you can get a lot more out of it, from how it should be configured to what you should monitor to using it directly in your applications. So, how does it work?

MongoDB’s replication is actually very simple: the master keeps a collection that describes writes and the slaves query that collection. This collection is called the oplog (short for “operation log”).

#### The oplog
Each write (insert, update, or delete) creates a document in the oplog collection, so long as replication is enabled (MongoDB won’t bother keeping an oplog if replication isn’t on). So, to see the oplog in action, start by running the database with the –replSet option:
`$ ./mongod --replSet funWithOplogs`

Now, when you do operations, you’ll be able to see them in the oplog. Let’s start out by initializing out replica set:
`> rs.initiate()`

Now if we query the oplog you’ll see this operation:

```
> use local
switched to db local
> db.oplog.rs.find()
{ 
    "ts" : { "t" : 1286821527000, "i" : 1 }, 
    "h" : NumberLong(0), 
    "op" : "n", 
    "ns" : "", 
    "o" : { "msg" : "initiating set" } 
}
```

This is just an informational message for the slave, it isn’t a “real” operation. Breaking this down, it contains the following fields:

- ts: the time this operation occurred.
- h: a unique ID for this operation. Each operation will have a different value in this field.
- op: the write operation that should be applied to the slave. n indicates a no-op, this is just an informational message.
- ns: the database and collection affected by this operation. Since this is a no-op, this field is left blank.
- o: the actual document representing the op. Since this is a no-op, this field is pretty useless.

To see some real oplog messages, we’ll need to do some writes. Let’s do a few simple ones in the shell:

```
> use test
switched to db test
> db.foo.insert({x:1})
> db.foo.update({x:1}, {$set : {y:1}})
> db.foo.update({x:2}, {$set : {y:1}}, true)
> db.foo.remove({x:1})
```

Now look at the oplog:

```
> use local
switched to db local
> db.oplog.rs.find()
{ "ts" : { "t" : 1286821527000, "i" : 1 }, "h" : NumberLong(0), "op" : "n", "ns" : "", "o" : { "msg" : "initiating set" } }
{ "ts" : { "t" : 1286821977000, "i" : 1 }, "h" : NumberLong("1722870850266333201"), "op" : "i", "ns" : "test.foo", "o" : { "_id" : ObjectId("4cb35859007cc1f4f9f7f85d"), "x" : 1 } }
{ "ts" : { "t" : 1286821984000, "i" : 1 }, "h" : NumberLong("1633487572904743924"), "op" : "u", "ns" : "test.foo", "o2" : { "_id" : ObjectId("4cb35859007cc1f4f9f7f85d") }, "o" : { "$set" : { "y" : 1 } } }
{ "ts" : { "t" : 1286821993000, "i" : 1 }, "h" : NumberLong("5491114356580488109"), "op" : "i", "ns" : "test.foo", "o" : { "_id" : ObjectId("4cb3586928ce78a2245fbd57"), "x" : 2, "y" : 1 } }
{ "ts" : { "t" : 1286821996000, "i" : 1 }, "h" : NumberLong("243223472855067144"), "op" : "d", "ns" : "test.foo", "b" : true, "o" : { "_id" : ObjectId("4cb35859007cc1f4f9f7f85d") } }
```

You can see that each operation has a ns now: “test.foo”. There are also three operations represented (the op field), corresponding to the three types of writes mentioned earlier: i for inserts, u for updates, and d for deletes.

The o field now contains the document to insert or the criteria to update and remove. Notice that, for the update, there are two o fields (o and o2). o2 give the update criteria and o gives the modifications (equivalent to update()‘s second argument).

#### Using this information

MongoDB doesn’t yet have triggers, but applications could hook into this collection if they’re interested in doing something every time a document is deleted (or updated, or inserted, etc.) Part three of this series will elaborate on this idea.


