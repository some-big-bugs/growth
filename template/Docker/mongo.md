```s
docker run --name some-mongo \
    -v /Users/zhangyunan/docker_data/mongo/data:/data/db \
    -p 27017:27017 \
    -e MONGO_INITDB_ROOT_USERNAME=root \
    -e MONGO_INITDB_ROOT_PASSWORD=123456 \
    -d mongo \
    --wiredTigerCacheSizeGB 1
```