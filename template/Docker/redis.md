```s
docker run --name redis -p 6379:6379 -v D:/docker/volumns/redis/data:/data -d redis redis-server --appendonly yes
```
