FROM registry.access.redhat.com/ubi8/ubi:latest


# --build-arg docker-composeのargsを受け取る
ARG arg_resin_guest_port
ARG arg_resin_server_name
ARG arg_jdbc_file_name
ARG arg_jdbc_download_url
ARG arg_im_java_home
ARG arg_resin_home
ARG arg_container_jdk
ARG arg_http_proxy
ARG arg_https_proxy

# 環境変数にHTTPプロキシ情報を設定する
# 定義: ./ap/.env HTTP_PROXY
ENV http_proxy=$arg_http_proxy
ENV HTTP_PROXY=$arg_http_proxy
ENV https_proxy=$arg_https_proxy
ENV HTTPS_PROXY=$arg_https_proxy
ENV CONTAINER_JDK="${arg_container_jdk}"

# resinサーバー名称を設定
ENV RESIN_SERVER_NAME="${arg_resin_server_name}"
# RESIN_HOMEを設定
ENV RESIN_HOME="${arg_resin_home}"
RUN export RESIN_HOME=${RESIN_HOME}

#---------------------------------------------------------------------
# パッケージインストール
#---------------------------------------------------------------------
RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
RUN yum repolist && yum update -y && \
    yum install -y ${CONTAINER_JDK} "${CONTAINER_JDK}-devel" && \
    yum install -y gcc glibc-devel make openssl openssl-devel which tzdata wget && \
    ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

# 設定ファイル: ./ap/.env IM_JAVA_HOMEとOpenJDKの結合
RUN JAVA_HOME=${arg_im_java_home}/`ls /usr/lib/jvm/ | grep ${CONTAINER_JDK}`

ENV CLASS_PATH="$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar"
#---------------------------------------------------------------------
# RESINの設定
# * ホストの$RESIN_SERVER_NAMEをコンテナの$RESIN_HOMEに追加
# * $RESIN_HOME/libにjdbcドライバを追加 
#---------------------------------------------------------------------
ADD "./${RESIN_SERVER_NAME}/" "$RESIN_HOME"

WORKDIR $RESIN_HOME/lib
RUN wget "${arg_jdbc_download_url}/${arg_jdbc_file_name}"

WORKDIR $RESIN_HOME

#---------------------------------------------------------------------
# RESINのインストール
#---------------------------------------------------------------------
RUN chmod 755 "$RESIN_HOME/configure"
RUN $RESIN_HOME/configure --prefix=$RESIN_HOME --enable-64bit  && \
     make && \
     make install

# コンテナのresinポートを開放
EXPOSE ${arg_resin_guest_port}
