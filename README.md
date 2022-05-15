# Mongo Omar

```sh
docker-compose up -d # Run the cluster in background
docker-compose logs -f # Show docker compose logs (live)
docker-compose down # Shutdown cluster

mongosh --host 127.0.0.1 --port 27017 --authenticationDatabase admin --username root --password OmarIsVerySecure # Connect to mongodb master as root

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

## Stress Testing

https://github.com/johnlpage/POCDriver/tree/885f4e7f67a43fec69a29c9334344419c65839b4

## Doc

- https://github.com/google/cadvisor

## Additionall resources

https://github.com/nthieu29/mongodb-sharding-demo
