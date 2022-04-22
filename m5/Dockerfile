FROM amd64/ubuntu:20.04 as builder
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update -y

# install dependencies
RUN apt install -y git python build-essential autoconf libtool scons
RUN apt install -y gdb

# copy m5 and set it up
WORKDIR /gem5
RUN git clone https://github.com/gem5/gem5.git --depth=1

WORKDIR /gem5/gem5
RUN scons -C util/m5 build/x86/out/m5

# download and build Whisper
WORKDIR /work

RUN git clone https://github.com/swapnilh/whisper.git --depth=1

WORKDIR /work/whisper

RUN git submodule update --depth=1 --init nstore

WORKDIR /work/whisper/nstore

# build ycsb with debug option
RUN ./bootstrap
RUN ./configure CFLAGS='-O2' CXXFLAGS='-O2'
RUN make

## PRODUCTION
FROM amd64/ubuntu:20.04 as prod
LABEL com.iximiuz-project="docker-to-linux"
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update -y
RUN apt-get -y install \
  linux-image-virtual \
  systemd-sysv
RUN echo "root:root" | chpasswd

COPY --from=builder /gem5/gem5/util/m5/build/x86/out/m5 /sbin/m5
RUN ln -s /sbin/m5 /sbin/gem5

WORKDIR /work/whisper
COPY --from=builder /work/whisper .

RUN apt install -y python

# ./nstore/src/nstore -x10 -k100 -w -p0.5 -e1 -n0 --ycsb
CMD ["./nstore/src/nstore", "-x10", "-k100", "-w", "-p0.5", "-e1", "-n0", "--ycsb"]
