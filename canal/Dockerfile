FROM openjdk:8-jre-alpine

# 设置工作目录
WORKDIR /opt/canal-adapter

# 将 Canal Adapter 的压缩包复制到容器中
COPY canal.adapter-1.1.7.tar.gz ./

# 安装必要的工具
RUN apk add --no-cache bash file tar

# 解压
RUN tar -zxf canal.adapter-1.1.7.tar.gz -C /opt/canal-adapter

# COPY启动脚本到容器
COPY startup.sh /opt/canal-adapter/bin/startup.sh

# 暴露 Canal Adapter 使用的端口
EXPOSE 8081

# 设置容器的启动命令
CMD ["sh","-c","/opt/canal-adapter/bin/startup.sh && tail -F logs/adapter/adapter.log"]
