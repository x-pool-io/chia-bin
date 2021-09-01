# chia-cli
chia-cli 为x-pool集成的挖矿，P图软件，检测工具

# 系统要求
Ubuntu 20.04 或 更高版本

# 1 挖矿

## 1.1. 生成配置文件

```
./chia-cli config -f config.yaml
```

* `wallet_addr`: 收款地址
* `worker_name`: 矿工名
* `signatures`: 签名, 签名生成方法见下文
* `final_dir`: P图文件存放的路径，可以填写多个，挖矿软件会搜索最多5层子目录，因此，如果有多个存放P图文件的目录，只需要填写最上层目录即可

## 1.2. 生成签名

```
./chia-cli sign
```
输入助记词，生成签名，助记词不会打印在屏幕上，直接回车即可，会得到如下结果

```
Successfully get your 24 words mnemonic.
pubkey: a4345569c3939cab494f984913a404cfb3b3274d7898d06cc6fbf3cd458ad1e39523e6f5fd5e054840878881d4acca51
figure print: 1186999671
farmer public key: 8376e0e0969badaac03d83ad607ff3f433b7396384b86dbcc81a6c0855916e118316473a30a7ffe134fda53245f7b34b
pool public key: a8a48d3883d27a1874b78bf2ba43e312a62c0a5284e88c7e01ed3aff73c418c1f98c124699edb5687514fd795f38d81c
signature: WCIKgwE2k/+FBeToWwXrNWt1yPD40xkO3LAIBjBdUkZGRlZp5N9AhU7qd+lSmNjGkKS8nNWqoqhyZ1Dk3a6X29/XvWgwzQVJ4ghM5p17JJzE8qsBKMm8ME7y8D/+A6dUfJ2vDKaAJIaVEK4SZ/UtyUuGvqQ7vW5B9jw/+xlXjncLUGmD/UbPJMwJohYygCho6RCyEBtuHtn3/vZzfMSoNWy443dxHgzfNMaRCLPE5WmkR/bo+qY6Bv52r0Uil5zJv7uZdKgEoNhzce27fncthRfcBlpAS2M0rbRtrkoKPsuBJ5jQkeJilHg983pdffmI5G5nhCrxDXdaYFGS38aEzA==
```
注意检查 figure print 是否和官方钱包中的结果一致。

如果有多个助记词，可以多次使用该命令来生成签名，在往配置文件中填写签名格式时，如下：

```yaml
signatures:
  - signature_1
  - signature_2
  - signature_3
```

## 1.3. 启动挖矿

```shell
./chia-cli mine -c config.yaml
```

# 2. P图
`chia-cli`集成了官方的P图软件，只需要更改配置文件config.yaml相应的字段，然后用命令：

```shell
./chia-cli plot -c config.yaml
```

# 3. docker中运行
对于非Ubuntu 20.04 的系统，可以采用docker运行的方式

## 3.1. 进入 docker bash

```shell
docker run -it --rm -v `pwd`:/chia --work-dir=/chia ubuntu:20.04 bash
```

进入docker bash后，同[1.1](), [1.2]()生成配置文件和生成签名方法

## 3.2. 挖矿

### 3.2.1 启动

```shell
export FINAL_DIR_1=/path/to/your/plot_file_dir_1
export FINAL_DIR_2=/path/to/your/plot_file_dir_2
docker run -d --name chia-cli \
  --privileged \
  -v $FINAL_DIR_1:$FINAL_DIR_1 \
  -v $FINAL_DIR_2:$FINAL_DIR_2 \
  -v `pwd`:/chia \
  --restart=always \
  --workdir=/chia \
  ubuntu:20.04 \
  ./chia-cli mine -c ./config.yaml -v
```
注意将 `FINAL_DIR_1`,`FINAL_DIR_2` 设置为自己的存放P盘的路径, 如果有一个或者多个，则增加或者减少docker路径映射。

当日志中出现：`login success`时，说明登录成功

### 3.2.2 重启

```bash
docker restart chia-cli
```

### 3.2.3 查看日志

```bash
docker logs -f chia-cli --tail=100
```

### 3.2.4 升级或者更改配置文件
在宿主机中直接替换掉`chia-cli`或者更改配置文件后，重启docker即可

# 4. 检查P图文件是否正确
直接在宿主机上运行命令或者进入步骤 [3.1]()后运行命令：

```bash
./chia-cli check -c config.yaml
```
结果会将错误文件打印出来
