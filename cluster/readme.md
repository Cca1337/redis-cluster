## Redis cluster

### Configuration
redis.conf

```
#persistence RDB and AOF 
dir /data
dbfilename dump.rdb
appendonly yes
appendfilename "appendonly.aof"

```
### redis-0 Configuration
Master
```
protected-mode no
port 6379

#authentication
masterauth a-very-complex-password-here
requirepass a-very-complex-password-here
```
### redis-1 Configuration
Slave
```
protected-mode no
port 6379
slaveof redis-0 6379

#authentication
masterauth a-very-complex-password-here
requirepass a-very-complex-password-here

```
### redis-2 Configuration
Slave
```
protected-mode no
port 6379
slaveof redis-0 6379

#authentication
masterauth a-very-complex-password-here
requirepass a-very-complex-password-here

```

### Next steps

* create network
```
docker network create redis
```

* run master and slaves

```
cd cluster\

#redis-0
docker run -d --rm --name redis-0 `
    --net redis `
    -v ${PWD}/redis-0:/etc/redis/ `
    redis redis-server /etc/redis/redis.conf

#redis-1
docker run -d --rm --name redis-1 `
    --net redis `
    -v ${PWD}/redis-1:/etc/redis/ `
    redis redis-server /etc/redis/redis.conf

#redis-2
docker run -d --rm --name redis-2 `
    --net redis `
    -v ${PWD}/redis-2:/etc/redis/ `
    redis redis-server /etc/redis/redis.conf

```
## Test Replication

```
# go to one of the clients
docker exec -it redis-2 sh
redis-cli
auth "a-very-complex-password-here"
keys *

```

## Running Sentinels

Documentation [here](https://redis.io/topics/sentinel)

```
#********BASIC CONFIG******sentinel.conf*******************
port 5000
sentinel monitor mymaster redis-0 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster a-very-complex-password-here
#********************************************

```
## Starting Redis in sentinel mode

```

docker run -d --rm --name sentinel-0 --net redis `
    -v ${PWD}/sentinel-0:/etc/redis/ `
    redis redis-sentinel /etc/redis/sentinel.conf

docker run -d --rm --name sentinel-1 --net redis `
    -v ${PWD}/sentinel-1:/etc/redis/ `
    redis redis-sentinel /etc/redis/sentinel.conf

docker run -d --rm --name sentinel-2 --net redis `
    -v ${PWD}/sentinel-2:/etc/redis/ `
    redis redis-sentinel /etc/redis/sentinel.conf

```
## Test Sentinels

```
docker logs sentinel-0
docker exec -it sentinel-0 sh
redis-cli -p 5000
info
sentinel master mymaster
```
