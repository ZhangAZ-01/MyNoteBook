# merger_build

## init_repo.sh

用于设置和更新开发环境，管理和更新必要的子模块，保证工作环境干净。

```bash
#! /bin/bash
#
# Copyright 2023 Vipshop Inc. All Rights Reserved.
# Author: Yafei Zhang (yafei06.zhang@vipshop.com)
#

set -e  # 任何命令失败 脚本立即终止执行
cd $(dirname $0)  # 确保脚本在该目录下进行
CWD=$(pwd)  # 路径为CWD

rm -rf ~/.scaffold  # 清除已有的 .scaffold 目录
scaffold init_deps --pull  # 初始化或更新项目的依赖 --pull 表示远程拉取最新依赖

cd $CWD/tesseract/tesseract_entity_idls
git submodule update --init  # 确保所有子模块都被拉取并且更新到适当状态

cd $CWD
rm -rf bazel_clang_tools
git clone git@gitlab.tools.vipshop.com:p13n_cxx/bazel_clang_tools.git
```

## start_staging_server.sh

```bash
#!/bin/bash
#
# Copyright 2023 Vipshop Inc. All Rights Reserved.
# Author: Meng Ding (daemon.d@vipshop.com)
#
# 用于在 10.189.108.33 测试服上启动 staging 服务
#

set -e

# 保证传递了至少两个参数，参数不足则打印使用方法并退出
if [ $# -lt 2 ]; then
  echo "usage: $0 version(v2|v3) scene"
  exit 1
fi

version=$1
scene=$2

# 该机器是公共资源，为了防止影响其他人，这里禁用 7920 端口
export CARBON_METRICS_HTTP_ADDR=""

function StartMerger2() {
  if [ $scene = tess ]; then  #如果场景是 tess，则尝试启动 server_tess；
    if [ ! -f server_$scene ]; then
      echo "server_$scene not found."
      exit 1
    fi
    nohup ./server_tess >/dev/null 2>&1 &
  else
    if [ ! -f server_merger_$scene ]; then  # 否则尝试启动名为 sever_merger_$scence 的服务。
      echo "server_merger_$scene not found."
      exit 1
    fi
    nohup ./server_merger_$scene >/dev/null 2>&1 &
  fi
}

function StartMerger3() {
  if [ ! -f server_$scene ]; then
    echo "server_$scene not found."
    exit 1
  fi

  if [ $scene = featurehub ]; then
    nohup ./server_featurehub >/dev/null 2>&1 &
  else
    nohup ./server_$scene --mock_tracer --resource_config_path= --staging=$scene --health_check_enabled=false >/dev/null 2>&1 &
  fi
}

if [ $version = v2 ]; then
  StartMerger2
else
  StartMerger3
fi

```

## merger3_idls_jar

### deploy.sh

实现自动化部署，流程如下：

1. 创建存放 protobuf IDL 文件的目录。

2. 克隆项目相关的接口定义。

3. 清理冲突文件。

4. 执行 Maven 部署，使用 mvn deploy 命令将构建好的 JAR 包部署到远程 Maven 仓库。

5. 清理工作目录，删除原代码和目标文件。

### pom.xml

- 基本项目信息

- 属性设置

  - 设置编码格式，Google Protocol Buffers 库的版本号，指定 gRPC 版本号。

- 依赖管理

- 分发管理

  - 配置了用于发布项目的构建成果物的仓库地址。

- 构建配置

  - resources: 指定资源文件的存放目录，表示可能包含一些接口定义文件。

  - finalName: 最终构建的 JAR 文件命名。

  - extensions: 引入了操作系统检测插件，用于根据操作系统类型自动配置一些参数。

  - plugins: 包括了编译插件和专用于处理 Protocol Buffers 和 gRPC 的插件：

  - maven-compiler-plugin: 配置了 Java 源代码的编译设置。

  - protobuf-maven-plugin: 自动处理 .proto 文件，生成相应的 Java 类和 gRPC 服务代码。

- 插件具体配置

这两个文件组成了一个完整的自动化构建和部署流程。
