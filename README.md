参考：[入门 — PyTorch/Serve master 文档](https://pytorch.org/serve/getting_started.html)

每次使用prompt记得进入E:/torchserve

记得安装依赖时时cu113

```
REST API文件说明：
densenet161-8d451a50.pth，导入模型的3文件之一
logs:日志文件目录
model-store:建立模型后模型文件mar的存放目录
kitten_small.png:测试预测接口REST API的测试文件
```





```log
Could not fetch URL https://pypi.org/simple/cython/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/cython/ (Caused by SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))) - skipping
ERROR: Could not find a version that satisfies the requirement cython==3.0.5 (from versions: none)
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))) - skipping
ERROR: No matching distribution found for cython==3.0.5
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))': /simple/psutil/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))': /simple/psutil/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))': /simple/psutil/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))': /simple/psutil/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))': /simple/psutil/
Could not fetch URL https://pypi.org/simple/psutil/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/psutil/ (Caused by SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))) - skipping
ERROR: Could not find a version that satisfies the requirement psutil==5.9.8 (from versions: none)
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError(SSLZeroReturnError(6, 'TLS/SSL connection has been closed (EOF) (_ssl.c:1149)'))) - skipping
ERROR: No matching distribution found for psutil==5.9.8
网络问题，直接下载解压，改目录名为serve
```





成功安装：

```
Installing collected packages: torch-workflow-archiver, enum-compat, torchserve, torch-model-archiver
Successfully installed enum-compat-0.0.3 torch-model-archiver-0.12.0 torch-workflow-archiver-0.2.15 torchserve-0.12.0
```



使用过程参见官网： [入门 — PyTorch/Serve master 文档](https://pytorch.org/serve/getting_started.html)

其中使用的报错：

```
#建立过程
1：wget https://download.pytorch.org/models/densenet161-8d451a50.pth
2：torch-model-archiver --model-name densenet161 --version 1.0 --model-file 3：./serve/examples/image_classifier/densenet_161/model.py --serialized-file densenet161-8d451a50.pth --export-path model_store --extra-files ./serve/examples/image_classifier/index_to_name.json --handler image_classifier

#开始启动发生端口占用
netstat ano | findstr 

#重新启动有PID错误
删掉缓存文件C:\Users\wrwut\AppData\Local\Temp

#ModuleNotFoundError: No module named 'nvgpu'
无光紧要，除非如果需要 GPU 相关的系统性能监控，运行以下命令安装缺失的模块pip install nvgpu

#一直返回400
需要API令牌，搜索在 TorchServe 目录或模型存储目录中查找 config.properties 文件（需自己生成）或者命令行指定
检查是否设置了 enable_auth_token=true。如果启用了此设置，访问管理 API 时需要提供令牌。可以禁用，或者：
可以使用 management_api.generate_token() 端点生成令牌：客户端进行
curl -X POST http://127.0.0.1:8081/token \
-H "Content-Type: application/json" \
-d '{"username": "user", "password": "password"}'好像不行！！！
那就禁用吧，命令加个
torchserve --start --ncs --model-store model_store --disable-token-auth

#测试/ping没问题，但返回/models为空
api默认禁用嘛，试一下设置命令行参数设置指定--models呢？可以了，em那就是这样TorchServe 现在默认情况下会禁用使用模型 API，需指定
torchserve --start --ncs --model-store model_store --models densenet161=densenet161.mar --disable-token-auth
或者手动创建config.properties 文件，内容为：
inference_address=http://127.0.0.1:8080
management_address=http://127.0.0.1:8081
enable_envvars_config=true
以后得命令就可以直接使用--ts-config指定该文件就行
torchserve --start --ncs --model-store model_store --ts-config config.properties
```



成功启动：

```
#prompt运行命令：
torchserve --start --ncs --model-store model_store --models densenet161=densenet161.mar --disable-token-auth
#另外开一个终端
C:\Users\Lx>curl http://127.0.0.1:8081/models
{
  "models": [
    {
      "modelName": "densenet161",
      "modelUrl": "densenet161.mar"
    }
  ]
}

C:\Users\Lx>curl http://127.0.0.1:8080/ping
{
  "status": "Healthy"
}

```



预测api：

```
#GRPC API
#依赖安装成功
Installing collected packages: protobuf, grpcio, grpcio-tools
Successfully installed grpcio-1.68.1 grpcio-tools-1.68.1 protobuf-5.29.2
未进行客户端预测测试

#REST API:直接使用成功
E:\torchserver>curl http://127.0.0.1:8080/predictions/densenet161 -T kitten_small.jpg
{
  "tabby": 0.47805070877075195,
  "lynx": 0.20043258368968964,
  "tiger_cat": 0.16802920401096344,
  "tiger": 0.061942897737026215,
  "Egyptian_cat": 0.05109507590532303
}
判断为tabby
```

我先看看torchserve能不能部署类似图像生成的模型吧





其他部署框架：

```
Flask
FastAPI
TorchServe
```

