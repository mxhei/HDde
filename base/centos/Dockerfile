ARG VERSION=7
FROM centos:${VERSION}

ARG TIMEZONE="Asia/Shanghai"
ENV GITHUB_PROXY="https://github.com"

RUN VERSION=$(cat /etc/redhat-release  | awk '{print $4}') \
    # set timezone
    && rm -rf /etc/localtime && ln -s /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    # change yum repository
    && {\
        mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup; \
        curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-${VERSION:0:1}.repo; \
        sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo; \
    } \
    # set epel repository
    && [ "${VERSION:0:1}" = "8" ] && { \
        yum install -y  https://mirrors.aliyun.com/epel/epel-release-latest-8.noarch.rpm; \
        sed -i 's|^#baseurl=https://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*; \
        sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*; \
    } || { \
        curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-${VERSION:0:1}.repo; \
    }\
    && yum makecache \
    # install develop tools
    && pkgList="wget gcc gcc-c++ make cmake autoconf libzip libzip-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel readline-devel curl curl-devel openssl openssl-devel libevent-devel vim zip unzip net-tools telnet" \
    && yum -y install ${pkgList} \
    # install Jemalloc
    && wget -4 -O /tmp/jemalloc-5.2.1.tar.bz2 http://mirrors.linuxeye.com/oneinstack/src/jemalloc-5.2.1.tar.bz2 \
    && tar xjf /tmp/jemalloc-5.2.1.tar.bz2  -C /tmp \
    && pushd /tmp/jemalloc-5.2.1 > /dev/null \
    && ./configure && make -j $(nproc) && make install \
    && ln -s /usr/local/lib/libjemalloc.so.2 /usr/lib64/libjemalloc.so.1 \
    && ln -s /usr/local/lib/libjemalloc.so.2 /usr/lib/libjemalloc.so.1 \
    && popd >/dev/null \
    && ldconfig && rm -rf /tmp/jemalloc* \
    # install gosu
    && wget -4 -O /usr/bin/gosu "${GITHUB_PROXY}/tianon/gosu/releases/download/1.12/gosu-amd64" \
    && wget -4 -O /tmp/gosu.asc "${GITHUB_PROXY}/tianon/gosu/releases/download/1.12/gosu-amd64.asc" \
    && gpg --keyserver keyserver.ubuntu.com --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /tmp/gosu.asc /usr/bin/gosu && rm -rf /tmp/gosu.asc \
    && yum clean all

WORKDIR /work