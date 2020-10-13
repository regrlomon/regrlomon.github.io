---
title: bootstrap.sh为我们做了什么
tag: hyperledger fabric
---
在运行bootstrap.sh安装fabric samples和相关环境时，由于网络问题，很多下了一半就突然停止下载了，结果又要重新安装，所以分析一下这个脚本有哪些功能，帮我们做了哪几件事情。
<!--more-->
## bootstrap.sh的使用
我的配置环境是:Ubuntu 20.04、docker、go
现在fabric的最新版本是2.2.1，fabric-ca的版本是1.4.9，但是最新版本资料较少，而且有bug，这里选择安装1.1.0版本。在命令行输入`./bootstrap.sh 1.1.0 1.1.0`则安装的是1.1.0版本的fabric和fabric-ca；如果输入`./bootstrap.sh`，则默认安装的是最新版的fabric和fabric-ca；如果输入`./bootstrap.sh 1.1.0`，则安装的是1.1.0的fabric和1.4.9的fabric-ca，我一开始是这样输入的，但是找不到1.4.9版本的fabric-ca的docker镜像，安装会报错，我去docker hub上看了一下，没有ubuntu环境的1.4.9版本的fabric-ca镜像，所以选择安装了1.1.0版本的fabric-ca。

## bootstrap.sh安装的部分
## 第一部分，下载fabric-samples的仓库:
这部分我安装的时候网络基本上不会出现问题，很快就能下载好，然后bootstrap.sh目录下面会出现一个fabric-samples的文件夹
```
cloneSamplesRepo() {
    # clone (if needed) hyperledger/fabric-samples and checkout corresponding
    # version to the binaries and docker images to be downloaded
    if [ -d first-network ]; then
        # if we are in the fabric-samples repo, checkout corresponding version
        echo "===> Checking out v${VERSION} of hyperledger/fabric-samples"
        git checkout v${VERSION}
    elif [ -d fabric-samples ]; then
        # if fabric-samples repo already cloned and in current directory,
        # cd fabric-samples and checkout corresponding version
        echo "===> Checking out v${VERSION} of hyperledger/fabric-samples"
        cd fabric-samples && git checkout v${VERSION}
    else
        echo "===> Cloning hyperledger/fabric-samples repo and checkout v${VERSION}"
        git clone -b master https://github.com/hyperledger/fabric-samples.git && cd fabric-samples && git checkout v${VERSION}
    fi
}
```
这部分是完成下载fabric-samples的功能的
## 第二部分，下载并安装fabric和fabric-ca的二进制文件
这里我遇到了网络问题，下载这两个文件特别的慢，校园网内下载这两个每秒只有30kb/s，**可以选择手动下载，解压时两个包内相同的目录直接合并即可，解压完后会有两个文件夹：bin、config，然后将这两个文件夹放在samples文件夹即可**
```
pullBinaries() {
    echo "===> Downloading version ${FABRIC_TAG} platform specific fabric binaries"
    download "${BINARY_FILE}" "https://github.com/hyperledger/fabric/releases/download/v${VERSION}/${BINARY_FILE}"
    if [ $? -eq 22 ]; then
        echo
        echo "------> ${FABRIC_TAG} platform specific fabric binary is not available to download <----"
        echo
        exit
    fi

    echo "===> Downloading version ${CA_TAG} platform specific fabric-ca-client binary"
    download "${CA_BINARY_FILE}" "https://github.com/hyperledger/fabric-ca/releases/download/v${CA_VERSION}/${CA_BINARY_FILE}"
    if [ $? -eq 22 ]; then
        echo
        echo "------> ${CA_TAG} fabric-ca-client binary is not available to download  (Available from 1.1.0-rc1) <----"
        echo
        exit
    fi
}
```
pullBinaries()调用download()进行下载，上半部分是下载fabric，下半部分是下载fabric-ca
```
download() {
    local BINARY_FILE=$1
    local URL=$2
    echo "===> Downloading: " "${URL}"
    curl -L --retry 5 --retry-delay 3 "${URL}" | tar xz || rc=$?
    if [ -n "$rc" ]; then
        echo "==> There was an error downloading the binary file."
        return 22
    else
        echo "==> Done."
    fi
}
```
download()会将github上的二进制文件下载下来并解压
## 第三部分:安装fabric相关的docker镜像
这部分我把docker的源切换到了中科大源，下载速度很快，下面这两个是下载docker镜像的
```
dockerPull() {
    #three_digit_image_tag is passed in, e.g. "1.4.7"
    three_digit_image_tag=$1
    shift
    #two_digit_image_tag is derived, e.g. "1.4", especially useful as a local tag for two digit references to most recent baseos, ccenv, javaenv, nodeenv patch releases
    two_digit_image_tag=$(echo "$three_digit_image_tag" | cut -d'.' -f1,2)
    while [[ $# -gt 0 ]]
    do
        image_name="$1"
        echo "====> hyperledger/fabric-$image_name:$three_digit_image_tag"
        docker pull "hyperledger/fabric-$image_name:$three_digit_image_tag"
        docker tag "hyperledger/fabric-$image_name:$three_digit_image_tag" "hyperledger/fabric-$image_name"
        docker tag "hyperledger/fabric-$image_name:$three_digit_image_tag" "hyperledger/fabric-$image_name:$two_digit_image_tag"
        shift
    done
}
```
```
pullDockerImages() {
    command -v docker >& /dev/null
    NODOCKER=$?
    if [ "${NODOCKER}" == 0 ]; then
        FABRIC_IMAGES=(peer orderer ccenv tools)
        case "$VERSION" in
        2.*)
            FABRIC_IMAGES+=(baseos)
            shift
            ;;
        esac
        echo "FABRIC_IMAGES:" "${FABRIC_IMAGES[@]}"
        echo "===> Pulling fabric Images"
        dockerPull "${FABRIC_TAG}" "${FABRIC_IMAGES[@]}"
        echo "===> Pulling fabric ca Image"
        CA_IMAGE=(ca)
        dockerPull "${CA_TAG}" "${CA_IMAGE[@]}"
        echo "===> List out hyperledger docker images"
        docker images | grep hyperledger
    else
        echo "========================================================="
        echo "Docker not installed, bypassing download of Fabric images"
        echo "========================================================="
    fi
}
```
## 跳过某些安装部分的方法
在命令行输入`./bootstrap.sh --help`查看提示信息
```
printHelp() {
    echo "Usage: bootstrap.sh [version [ca_version]] [options]"
    echo
    echo "options:"
    echo "-h : this help"
    echo "-d : bypass docker image download"
    echo "-s : bypass fabric-samples repo clone"
    echo "-b : bypass download of platform-specific binaries"
    echo
    echo "e.g. bootstrap.sh 2.2.1 1.4.9 -s"
    echo "will download docker images and binaries for Fabric v2.2.1 and Fabric CA v1.4.9"
}
```
可以看到，通过后缀可以跳过上面三个部分其中任意部分的安装，其中-d是跳过docker镜像的安装，-s是跳过samples的安装，-b是跳过fabric和fabric-ca的安装。一开始我某些部分没安装成功就把所有的都重新来一遍，太浪费时间了
## 安装过程出现的问题
1、下载fabric和fabric-ca二进制文件时因为网络原因压根下载不了、下载一半提示错误，这个时候可以去`https://github.com/hyperledger/fabric/releases`下载你需要的版本，然后解压到samples文件夹内。  
2、docker下载镜像很慢，这里可以切换到国内源，修改或者创建`/etc/docker/daemon.json`文件，修改成如下形式：
```
{
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```
也可以多加两个源，如下：
```
{
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn","http://hub-mirror.c.163.com"]
}
```
切记，要重启一下docker
```
service docker restart
```