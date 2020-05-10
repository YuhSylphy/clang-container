FROM ubuntu:20.04

# 近場のミラーに変更 / tzdataがinteractiveになってしまうので先にインストール
RUN sed -i 's@archive.ubuntu.com@ftp.jaist.ac.jp/pub/Linux@g' /etc/apt/sources.list \
	&& apt update \
	&& apt install -y tzdata

# Gitを手動でPPA追加してインストール
RUN apt install -y gnupg \
	&& apt-key adv --keyserver keyserver.ubuntu.com --recv e1dd270288b4e6030699e45fa1715d88e1df1f24 \
	&& echo deb http://ppa.launchpad.net/git-core/ppa/ubuntu focal main >> /etc/apt/sources.list.d/git-core-ubuntu-ppa-focal.list \
	&& echo deb-src http://ppa.launchpad.net/git-core/ppa/ubuntu focal main  >> /etc/apt/sources.list.d/git-core-ubuntu-ppa-focal.list \
	&& apt update \
	&& apt upgrade \
	&& apt install -y git git-flow git-lfs

# SSHD
RUN apt install -y openssh-server
#	&& rm /etc/ssh/ss_host_*key*

RUN useradd ssh-user \
	# && echo "ssh-user:ssh-pass" | chpasswd \
	&& mkdir -p /home/ssh-user/ \
	&& chown ssh-user /home/ssh-user/ \
	&& chmod 755 /home/ssh-user/

USER ssh-user
RUN mkdir -p /tmp/.ssh/
COPY ./.ssh/* /tmp/.ssh/
RUN chmod 755 $HOME \
	&& mkdir -p $HOME/.ssh \
	&& chmod 700 $HOME/.ssh \
	&& cat /tmp/.ssh/*.pub >> $HOME/.ssh/authorized_keys \
	&& chmod 600 $HOME/.ssh/authorized_keys \
	&& cat $HOME/.ssh/authorized_keys
USER root
EXPOSE 22

# LLVM Debian/Ubuntu nightly packages
RUN apt install -y wget
ARG LLVM_VERSION=10
RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - \
	&& echo deb http://apt.llvm.org/focal/ llvm-toolchain-focal-10 main >> /etc/apt/sources.list.d/llvm-toolchain-focal-10.list \
	&& echo deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal-10 main >> /etc/apt/sources.list.d/llvm-toolchain-focal-10.list \
	&& apt update \
	&& apt upgrade -y \
	&& apt install -y cmake \
	&& mkdir -p /usr/local/include/
	# LLVM
RUN apt install -y libllvm-$LLVM_VERSION-ocaml-dev libllvm10 llvm-$LLVM_VERSION llvm-$LLVM_VERSION-dev llvm-$LLVM_VERSION-doc llvm-$LLVM_VERSION-examples llvm-$LLVM_VERSION-runtime \
	# Clang and co
	&& apt install -y clang-$LLVM_VERSION clang-tools-$LLVM_VERSION clang-$LLVM_VERSION-doc libclang-common-$LLVM_VERSION-dev libclang-$LLVM_VERSION-dev libclang1-$LLVM_VERSION clang-format-$LLVM_VERSION clangd-$LLVM_VERSION \
	# libfuzzer
	&& apt install -y libfuzzer-$LLVM_VERSION-dev \
	# lldb
	&& apt install -y lldb-$LLVM_VERSION \
	# lld (linker)
	&& apt install -y lld-$LLVM_VERSION \
	# libc++
	&& apt install -y libc++-$LLVM_VERSION-dev libc++abi-$LLVM_VERSION-dev \
	# OpenMP
	&& apt install -y libomp-$LLVM_VERSION-dev
# google test
RUN cd /tmp \
	&& git clone https://github.com/google/googletest.git \
	&& mkdir -p /tmp/googletest/build \
	&& cd /tmp/googletest/build \
	&& CXX=/usr/bin/clang++-10 CC=/usr/bin/clang-10 cmake .. -DGTEST_CREATE_SHARED_LIBRARY=1 \
	&& make -j2 \
	&& make install
# cpp-yaml
RUN cd /tmp \
	&& git clone https://github.com/jbeder/yaml-cpp \
	&& mkdir -p /tmp/yaml-cpp/build \
	&& cd /tmp/yaml-cpp/build \
	&& CXX=/usr/bin/clang++-10 CC=/usr/bin/clang-10 cmake -DYAML_BUILD_SHARED_LIBS=ON .. \
	&& make -j2 \
	&& make install
# reload ld
RUN ldconfig

# RUN apt install -y wget libtinfo5 xz-utils libc-dev-bin
# COPY ./data/clang+llvm-10.0.0-x86_64-linux-gnu-ubuntu-18.04.tar.xz /tmp/
COPY ./data/boost_1_72_0.tar.bz2 /tmp/
# #RUN wget https://github-production-release-asset-2e65be.s3.amazonaws.com/75821432/498bf100-6f56-11ea-9aa2-65793e6ecf30?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200501%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200501T073312Z&X-Amz-Expires=300&X-Amz-Signature=c01b55727e33ecc7afa8fb257765620550899dc233093ee1021d772fa5b0cc96&X-Amz-SignedHeaders=host&actor_id=0&repo_id=75821432&response-content-disposition=attachment%3B%20filename%3Dclang%2Bllvm-10.0.0-x86_64-linux-gnu-ubuntu-18.04.tar.xz&response-content-type=application%2Foctet-stream
# #RUN wget https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.bz2

# TODO: build boost
RUN mkdir -p /usr/local/boost && tar -jxvf /tmp/boost*.tar.bz2 --strip-components 1 -C /usr/local/boost

RUN touch /root/start.sh \
	&& chmod 744 /root/start.sh \
	&& echo \#!/bin/bash >> /root/start.sh \
	&& echo /etc/init.d/ssh start >> /root/start.sh \
	&& echo /bin/bash >> /root/start.sh

CMD ["/root/start.sh"]
