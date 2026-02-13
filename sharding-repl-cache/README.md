# PyMongo API with Sharded mongoDB and Redis Cache

## Quick Start

Start all services:

```bash
docker compose up -d
```

## MongoDB Sharded Cluster Setup

### 1. Initialize Config Server

Connect to the config server:

```bash
docker compose exec -T configSrv mongosh --port 27027 <<EOF
rs.initiate({
  _id: "config_server",
  configsvr: true,
  members: [
    { _id: 0, host: "configSrv:27027" }
  ]
});
EOF
```

### 2. Initialize Shard 1 Replica Set

Connect to the first shard node:

```bash
docker compose exec -T shard1a mongosh --port 27028 <<EOF
rs.initiate({
  _id: "shard1rs",
  members: [
    { _id: 0, host: "shard1a:27028" },
    { _id: 1, host: "shard1b:27029" },
    { _id: 2, host: "shard1c:27030" }
  ]
});
EOF
```

### 3. Initialize Shard 2 Replica Set

Connect to the first shard node:

```bash
docker compose exec -T shard2a mongosh --port 27031 <<EOF
rs.initiate({
  _id: "shard2rs",
  members: [
    { _id: 0, host: "shard2a:27031" },
    { _id: 1, host: "shard2b:27032" },
    { _id: 2, host: "shard2c:27033" }
  ]
});
EOF
```

### 4. Configure Mongos Router

Connect to the mongos router:

```bash
docker compose exec -T mongos_router mongosh --port 27034 <<EOF
sh.addShard("shard1rs/shard1a:27028,shard1b:27029,shard1c:27030");
sh.addShard("shard2rs/shard2a:27031,shard2b:27032,shard2c:27033");
EOF
```

### 5. Enable Sharding

Enable sharding for a database and collection:

```bash
docker compose exec -T mongos_router mongosh --port 27034 <<EOF
sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name": "hashed" });
EOF
```

## 6. Fill database

```bash
./scripts/mongo-init.sh
```

## Check app output

```
curl http://localhost:8080
```

## Shutdown the setup

```bash
docker compose down -v
```

