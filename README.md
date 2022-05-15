# Mongo Omar

![diagram_1](./diagram_1.png)

## Todo

- [x] Implement Shard DB cluster (docker-compose, mongos, shardsvr, configsvr)
- [ ] Manage DB Access
- [ ] Manage System Access
- [ ] Manage Network Access
- [x] DB metrics monitoring (cAdvisor, Prometheus, Grafana)
- [ ] ~~Maybe manage backups~~
- [ ] Application to operate on the DB

## Technicalities

```sh
docker-compose up -d # Run the cluster in background
docker-compose logs -f # Show docker compose logs (live)
docker-compose down # Shutdown cluster

mongosh --host 127.0.0.1 --port 27017 --authenticationDatabase admin --username root --password OmarIsVerySecure # Connect to mongodb master as root

# To run workload tests run
java -jar POCDriver.jar -c "mongodb://root:OmarIsVerySecure@localhost:27017" -d 100

# You can use ps_mem.py to monitore your computer's memory
sudo ./ps_mem.py -w 2
```

```js
// Check cluster status
sh.status();
// Enable sharding for the database sample
sh.enableSharding("sample");
// Shared the collection restaurant
sh.shardCollection("sample.restaurant", { city: 1 });
// Check cluster status
sh.status();
```

### Manage System Access

Unauthenticated users can run multiple commands, but they are useless:

```js
db.runCommand({ buildInfo: 1 });
db.runCommand({ isdbgrid: 1 });
db.runCommand({ isMaster: 1 });
db.runCommand({ whatsmyuri: 1 });
db.runCommand({ _isSelf: 1 });
// etc
```

All the other commands will throw an error

```js
db.runCommand({ listCollections: 1 });
// MongoServerError: command listCollections requires authentication
```

Let's create bunch of users with different roles

```js
use admin // switch to admin db
// A user with the ability to read data on all non-system collections and the system.js collection.
db.createUser({
  user: "UserOne",
  pwd: "VerySecurePassword",
  roles: [{ role: "read", db: "admin" }],
});
// A user with all the privileges of the read role plus ability to modify data on all non-system collections and the system.js collection.
db.createUser({
  user: "UserTwo",
  pwd: "VerySecurePassword",
  roles: [{ role: "readWrite", db: "admin" }],
});
```

Now let's connect as `UserOne` and test some commands that requires `read` privileges

```sh
mongosh --host 127.0.0.1 --port 27017 --authenticationDatabase admin --username UserOne --password VerySecurePassword # Connect to mongodb master as UserOne
```

```js
use admin
db.runCommand({ listCollections: 1 });
// {
//   cursor: {
//     id: Long("0"),
//     ns: 'admin.$cmd.listCollections',
//     firstBatch: [
//       {
//         name: 'system.users',
//         type: 'collection',
// ...
```

It's working as you can see.

Now let's test a write command

```js
use admin
db.collection.insert({ test: "test" });
// MongoBulkWriteError: not authorized on admin to execute command ...
```

It's not working as expected.

Now let's switch to the account `UserTwo` and test the same command.

```sh
mongosh --host 127.0.0.1 --port 27017 --authenticationDatabase admin --username UserTwo --password VerySecurePassword # Connect to mongodb master as UserTwo
```

```js
use admin
db.collection.insert({ test: "test" });
// {
//   acknowledged: true,
//   insertedIds: { '0': ObjectId("6281660b5b9d56ada743c6bb") }
// }
```

It's successful.

## Resources

- MongoDB POCDriver: https://github.com/johnlpage/POCDriver/tree/885f4e7f67a43fec69a29c9334344419c65839b4
- cAdvisor: https://github.com/google/cadvisor
- Demo project: https://github.com/nthieu29/mongodb-sharding-demo

## Workload Help

```sh
# Where:
# -h show help
# -p show what the documents look like in the test
# -t how many threads to run on the client and thus how many connections.
# -s what threshold to consider slow when reporting latency percentages in ms
# -o output stats to a file rather then the screen
# -n use a namespace 'schema.collection' of your choice
# -d how long to run the loader for.
# -q *try* to limit rate to specified ops per second.
# -c a MongoDB connection string (note: you can include write concerns and thread pool size info in this)
# -k Fetch a single document using its primary key
# -r fetch a range of 10 documents
# -u increment an integer field in a random document
# -i add a new document
# -g update a random value in the array (must have arrays enabled)
# -v perform sets of operations on a stack:
# -v iuu will insert then update that document twice
# -v kui will find a document, update it, then insert a new document
#
# The last document is placed on a stack and p pops it off so:
# -v kiippu  Finds a document, adds two, then pops them off and updates the original document found.
# -m when updating a document use findAndModify to fetch a copy of the new incremented value
# -j when updating or querying limit the set to the last N% of documents added
# -b what size to use for operation batches.
# --rangedocs     number of documents to fetch for range queries (default 10)
# --updatefields  number of fields to update (default 1)
# --projectfields number of fields to project in finds (default 0 - return full document)
# -x how many fields to index aside from _id
# -w do not shard this collection on a sharded system
# -e empty this collection at the start of the run.
# -a add an X by Y array of integers to each document using -a X:Y
# -f aside from arrays and _id add f fields to the document, after the first 3 every third is an integer, every fifth a date, the rest are text.
# -l how many characters to have in the text fields
# --depth The depth of the document to create.
# --location Adds a field by name location and provides ISO-3166-2 code. One can provide "random" to fill the field with random values. This field is required for zone sharding with Atlas.
```
