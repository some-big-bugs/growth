```s
docker run -it --name youtrack-server-instance  \
    -v /Users/zhangyunan/docker_data/youtrack/data:/opt/youtrack/data \
    -v /Users/zhangyunan/docker_data/youtrack/conf:/opt/youtrack/conf  \
    -v /Users/zhangyunan/docker_data/youtrack/logs:/opt/youtrack/logs  \
    -v /Users/zhangyunan/docker_data/youtrack/backups:/opt/youtrack/backups  \
    -p 18000:8080 \
    jetbrains/youtrack:2020.6.5578
```