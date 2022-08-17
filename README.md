## 原理
参照
@hts0000
repo
icode-by-pass

docker镜像本质是一堆文件的集合，既然是文件，那就可以直接访问。
```bash
# 查看镜像文件存放的位置
docker inspect -f "lowerdir={{.GraphDriver.Data.LowerDir}},upperdir={{.GraphDriver.Data.UpperDir}},workdir={{.GraphDriver.Data.WorkDir}}" <镜像名> 
```

我们可以把镜像挂载到本地，方便后续操作
```bash
mkdir -p /tmp/coolenv
mount -t overlay overlay -o $(docker inspect -f "lowerdir={{.GraphDriver.Data.LowerDir}},upperdir={{.GraphDriver.Data.UpperDir}},workdir={{.GraphDriver.Data.WorkDir}}" <镜像名>) /tmp/coolenv
```
分析一下镜像启动流程
```bash
# 查看镜像的启动流程
docker history --no-trunc <镜像名>
```
分析后确定是start.sh这个脚本负责项目的启动，执行`find /tmp/coolenv -name start.sh`命令，查找start.sh脚本。脚本内容如下：
```bash
#!/bin/bash
set -eu

echo Starting mongod...
entrypoint-mongo.sh mongod &
echo Starting rabbitmq...
entrypoint-rabbit.sh rabbitmq-server &
echo Starting api server...
coolenv --race_data=/data/coolenv/ningbo.json --grpc_pb_gen=/data/coolenv/pb/coolenv.pb.go --grpc_v2_pb_gen=/data/coolenv/pb/v2
```

可以发现认证是通过coolenv这个二进制文件来做的