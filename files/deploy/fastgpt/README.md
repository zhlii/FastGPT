### 启动容器
```
openssl rand -base64 128 > ./mongo/keyfile

chmod 600 ./mongo/keyfile
sudo chown 999:root ./mongo/keyfile
```
### 初始化Mongo副本集
FastGPT 4.6.8 后使用了 MongoDB 的事务，需要运行在副本集上。副本集没法自动化初始化，需手动操作。

```
# 查看 mongo 容器是否正常运行
docker ps
# 进入容器
docker exec -it mongo bash

# 连接数据库
mongo -u myname -p mypassword --authenticationDatabase admin

# 初始化副本集。如果需要外网访问，mongo:27017 可以改成 ip:27017。但是需要同时修改 FastGPT 连接的参数（MONGODB_URI=mongodb://myname:mypassword@mongo:27017/fastgpt?authSource=admin => MONGODB_URI=mongodb://myname:mypassword@ip:27017/fastgpt?authSource=admin）
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo:27017" }
  ]
})
# 检查状态。如果提示 rs0 状态，则代表运行成功
rs.status()

```

### 测试m3e模型
```
curl --location --request POST 'https://m3e:6008/v1/embeddings' \
--header 'Authorization: Bearer sk-aaabbbcccdddeeefffggghhhiiijjjkkk' \
--header 'Content-Type: application/json' \
--data-raw '{
  "model": "m3e",
  "input": ["laf是什么"]
}'
```