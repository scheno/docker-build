FROM centos
# 版本
ARG version=4.9.0

# 环境变量
ENV ROCKETMQ_VERSION ${version}
ENV USER_HOME /home/app
ENV ROCKETMQ_HOME /home/app/rocketmq
ENV CONSOLE_HOME /home/app/console

# namesrv jvm参数
ENV NAMESRV_XMS 1024m
ENV NAMESRV_XMX 1024m
ENV NAMESRV_XMN 256m

# broker jvm参数
ENV BROKER_XMS 1024m
ENV BROKER_XMX 1024m
ENV BROKER_XMN 256m
ENV BROKER_MDM 1024m

# console 参数
ENV NAMESRV_ADDR localhost:9876

# 定义用户信息
ARG user=app
ARG group=app
ARG uid=3000
ARG gid=3000

# 复制rocketmq
COPY ["./run.sh","/tmp/rocketmq/"]

# 添加组
RUN groupadd -g ${gid} ${group} \
    # 添加用户
    && useradd -u ${uid} -g ${gid} -m -s /bin/bash ${user}
    # 更新 yum 源
RUN cd /etc/yum.repos.d/ \
    && sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-* \
    && sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
    # 安装软件包
RUN yum install -y epel-release \
    && yum install -y java htop net-tools vim wget zip lsof unzip \
    && yum clean all -y \
    # 显示vim行号
    && echo ":set number" > ${USER_HOME}/.vimrc
    # 将/root/rocketmq移动到/home/app/
RUN wget https://archive.apache.org/dist/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip \
    && unzip rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip \
    && mv rocketmq-all-${ROCKETMQ_VERSION}-bin-release rocketmq \
    && mv rocketmq ${USER_HOME}/ \
    && mv /tmp/rocketmq/run.sh ${USER_HOME} \
    && mkdir -p ${USER_HOME}/console/config \
    && mkdir -p ${USER_HOME}/console/store \
    && mkdir -p ${USER_HOME}/data/logs \
    # 创建软链接
    && ln -s ${USER_HOME}/data/logs ${USER_HOME}/logs \
    && ln -s ${USER_HOME}/data/rocketmq/conf ${ROCKETMQ_HOME}/conf \
    && ln -s ${USER_HOME}/data/rocketmq/store ${USER_HOME}/store \
    && ln -s ${USER_HOME}/data/console/config ${CONSOLE_HOME}/config \
    && ln -s ${USER_HOME}/data/console/store ${CONSOLE_HOME}/store \
    # 将文件和目录统一设置成644
    && chmod 644 -R ${USER_HOME} \
    # 将目录统一设置成755
    && find ${USER_HOME} -type d -print | xargs chmod 755 \
    # 将/home/app/rocketmq/rocketmq/bin下统一设置成755
    && chmod 755 -R ${ROCKETMQ_HOME}/bin \
    # 将/home/app/run.sh更名为/home/app/.run.sh
    && mv ${USER_HOME}/run.sh ${USER_HOME}/.run.sh \
    # 将/home/app/.run.sh设置为755
    && chmod 755 ${USER_HOME}/.run.sh \
    # 统一将/home/app设置用户和组
    && chown -R ${uid}:${gid} ${USER_HOME}

# 导出端口
EXPOSE 8080 9876 10909 10911 10912

# 工作目录
WORKDIR ${USER_HOME}

# 匿名卷
VOLUME ${USER_HOME}/data

# 切换用户环境
USER ${user}

#执行脚本
CMD ${USER_HOME}/.run.sh