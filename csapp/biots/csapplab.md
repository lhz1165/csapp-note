```bash
# 下载镜像
docker pull linxi177229/csapp:latest
# 查看
docker images
# 启动容器（里面有配置好的环境 和 PDF 资料）
docker run --name csapp -itd linxi177229/csapp 
# 进入容器 
docker attach csapp

# 接下来就和使用 平常的 Ubuntu：20.04 一样了
# 进入 lab1 进行一个简单的测试
cd ~
ls
cd csapplab
cd ~/csapplab/datalab/datalab-handout
make clean && make && ./btest
```