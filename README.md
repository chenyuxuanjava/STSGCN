#这个是由北京交通大学共享的内容，本人自己进行部署
# STSGCN
AAAI 2020. Spatial-Temporal Synchronous Graph Convolutional Networks: A New Framework for Spatial-Temporal Network Data Forecasting

url: paper/AAAI2020-STSGCN.pdf

# Usage
# 本人在虚拟机上面驱动安装没有成功，不知道是否虚拟机上面不支持，希望有大佬能够给虚拟上面Linux安装驱动的教程
Docker is recommended.
# 安装docker
**参照官网安装*

https://docs.docker.com/engine/install/centos/

**1.卸载之前的版本**
```
      yum remove docker \
      docker-client \
      docker-client-latest \
      docker-common \
      docker-latest \
      docker-latest-logrotate \
      docker-logrotate \
      docker-engine
  ```
  **注意这里有的人会报错，是因为python版本问题，可以卸载python3**
  **2.安装需要的yum**
  ```
  yum install -y yum-utils
  ```
  **3.安装一下使用国内镜像**
  ```
  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```
  **4.安装docker镜像，中间需要点几下 y**
  ```
  yum install docker-ce docker-ce-cli containerd.io
  ```

# 安装nvidia-docker，首先需要安装nvdia驱动

**步骤一**
```
yum install kernel-devel gcc -y
```
**步骤二，保证两个版本一样**
```
ls /boot | grep vmlinu
rpm -aq | grep kernel-devel
```

<img width="328" alt="1" src="https://user-images.githubusercontent.com/36155175/148874166-58c09459-fcfd-41db-9233-06b5ae4eeb35.png">

**步骤三，屏蔽系统自带的nouveau**

```
lsmod | grep nouveau
```

如果什么都没有就直接跳过步骤三，如果有内容需要执行下面内容

<img width="269" alt="1" src="https://user-images.githubusercontent.com/36155175/148874492-58a1da50-7700-44a1-9813-24cb28287c5d.png">

修改dist-blacklist.conf文件

```
vim /lib/modprobe.d/dist-blacklist.conf
```



将nvidiafb注释掉:
#blacklist nvidiafb 
然后添加以下语句：
blacklist nouveau
options nouveau modeset=0



**步骤四：重建initramfs image**
```
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

**步骤五：修改运行级别为文本模式**

```
systemctl set-default multi-user.target
```

**步骤六：重启**
```
reboot
```

**步骤七：需要到官网上面下载驱动**
https://www.nvidia.cn/Download/index.aspx?lang=cn

<img width="332" alt="1" src="https://user-images.githubusercontent.com/36155175/148875271-96d2e914-4a9c-45af-b3cf-349821fd3e1e.png">

**步骤八：给安装包权限**

```
chmod +x NVIDIA-Linux-x86_64-470.94.run
```

**步骤九：执行安装包**

```
./NVIDIA-Linux-x86_64-470.94.run
```
如果这里面报错，执行的时候需要添加--kernel-source-path

如果报此错：unable to load the kernel module 'nvidia.ko' .........
```
执行：./NVIDIA-XXXX.run --kernel-source-path=/usr/src/kernels/内核号  -k $(uname -r)
```

如果报此错： WARNING: You do not appear to have an NVIDIA GPU supported by the 430.34 NVIDIA Linux graph 
```
加上：--add-this-kernel 参数
```


如果报此错： unable to find the kernel source tree for the currently running kernel......... 
```
加上：--kernel-source-path=/usr/src/kernels/内核号(2+Tab键  自动出现)
```


```
./NVIDIA-Linux-x86_64-440.64.run --kernel-source-path=/usr/src/kernels/3.10.0-1062.18.1.el7.x86_64 -k $(uname -r)
```



1. install docker（上面安装docker已经完成）
2. install nvidia-docker
3. build image using `cd docker && docker build -t stsgcn/mxnet_1.41_cu100 .`
4. download the data [STSGCN_data.tar.gz](https://pan.baidu.com/s/1ZPIiOM__r1TRlmY4YGlolw) with code: `p72z`
5. uncompress data file using `tar -zxvf data.tar.gz`
6. modify the term `ctx` in `config/PEMS03/individual_GLU_mask_emb.json` to match your GPU devices
7. run code using `docker run -ti --rm --runtime=nvidia -v $PWD:/mxnet stsgcn/mxnet_1.41_cu100 python3 main.py --config config/PEMS03/individual_GLU_mask_emb.json`

If you are using Microsoft OpenPAI, modify the configurations saved in the folder `pai_jobs` to train STSGCNs on your clusters.

# repo structure
name|description
-|-
config|configurations of STSGCN
docker|dockerfile
models|core of STSGCN
pai_job|Microsoft OpenPAI configurations
paper|paper of STSGCN
test|pytest files
load_params.py|read parameters from local files
main.py|code of training STSGCN
pytest.ini|pytest configurations
requirements.txt|python packages requirements
utils.py|tools
