FROM amd64/ubuntu:20.04
LABEL com.iximiuz-project="docker-to-linux"
RUN apt-get update -y
RUN apt-get -y install \
  linux-image-virtual \
  systemd-sysv
RUN echo "root:root" | chpasswd

ENV DEBIAN_FRONTEND noninteractive

# install dependencies
RUN apt install -y git python build-essential autoconf libtool scons
RUN apt install -y pkg-config libconfig-dev libelf-dev libglib2.0-dev libboost-log-dev
RUN apt install -y libevent-dev
RUN apt install -y gdb

# copy m5 and set it up
WORKDIR /gem5
RUN git clone https://github.com/gem5/gem5.git --depth=1

WORKDIR /gem5/gem5
RUN scons -C util/m5 build/x86/out/m5
RUN cp util/m5/build/x86/out/m5 /sbin/m5

RUN ln -s /sbin/m5 /sbin/gem5

# download and build Whisper
WORKDIR /work

RUN git clone https://github.com/swapnilh/whisper.git --depth=1

WORKDIR /work/whisper

RUN git submodule update --depth=1 --init nstore

WORKDIR /work/whisper/nstore

# change path because /dev/shm has only 64MB, while we need 1GB
RUN sed -i 's|/dev/shm/zfile|/work/zfile|g' src/main.cpp

# build ycsb with debug option
RUN ./bootstrap
RUN ./configure CFLAGS='-g3 -O0' CXXFLAGS='-g3 -O0'
RUN make

# cleanup
RUN apt purge -y git python build-essential autoconf libtool scons
RUN apt purge -y pkg-config libconfig-dev libelf-dev libglib2.0-dev libboost-log-dev
RUN apt purge -y libevent-dev
RUN apt autoremove -y

RUN rm -rf /gem5

WORKDIR /work/whisper

# ./nstore/src/nstore -x10 -k100 -w -p0.5 -e1 -n0 --ycsb
CMD ["./nstore/src/nstore", "-x10", "-k100", "-w", "-p0.5", "-e1", "-n0", "--ycsb"]
