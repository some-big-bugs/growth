```s
docker run --name mysql5 \
    -p 13306:3306 \
    -v D:/docker/volumns/mysql8/datadir:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -d mysql:8
```