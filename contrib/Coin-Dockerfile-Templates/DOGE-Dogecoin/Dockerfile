FROM ubuntu:22.04 as baseimage2204
LABEL maintainer tpfuemp@github

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    automake autotools-dev build-essential dh-autoreconf \
    ca-certificates git \
    libboost-chrono-dev libboost-filesystem-dev libboost-program-options-dev libboost-system-dev libboost-test-dev libboost-thread-dev libevent-dev \
    libssl-dev libcurl4-openssl-dev libgmp-dev libsodium-dev libz-dev libdb-dev libdb++-dev bsdmainutils curl libsqlite3-dev \
    libtool pkg-config && \
    apt-get install -y --no-install-recommends mc psmisc wget protobuf-compiler libminiupnpc-dev g++ curl cmake bison libevent-dev libboost-dev libxml2-dev python3 flex dumb-init && \
    apt-get clean

FROM baseimage2204 as coinwallet

# Clone the source code.
ARG STRATUM_UPDATEPASS
ARG CURRENT_GITHUB_COMMIT=6c063e714f35cfabae6400fb5cdcb15473b384db

# build db5.3.28 or use --with-incompatible-bdb in configure
WORKDIR /tmp
COPY db53.patch /tmp/
RUN wget "https://github.com/berkeleydb/libdb/releases/download/v5.3.28/db-5.3.28.tar.gz" && \
    tar -xzf db-5.3.28.tar.gz && \
    patch -p0 < /tmp/db53.patch && \
    cd db-5.3.28/build_unix && ../dist/configure --build=x86_64 --prefix=/usr --with-pic --enable-cxx && make && make install && ldconfig  
WORKDIR /

RUN git clone https://github.com/dogecoin/dogecoin
WORKDIR /dogecoin

# use the specific commit known to work
RUN git checkout $CURRENT_GITHUB_COMMIT

# apply patch if needed
#COPY docker.patch /tmp/
#RUN patch -p1 < /tmp/docker.patch

RUN ./autogen.sh && \
    ./configure --with-gui=no --disable-tests --disable-bench && make clean && make && \
    strip src/dogecoind && cp src/dogecoind /usr/bin && \
    cd / && rm /dogecoin -rf && \
    rm ~/.cache -rf

# blocknotify
COPY blocknotify.cpp /tmp/
WORKDIR /tmp
RUN gcc -DSTRATUM_UPDATEPASS=\"$STRATUM_UPDATEPASS\" -o blocknotify blocknotify.cpp && \
    mv blocknotify /usr/bin/ && rm -f /tmp/blocknotify.cpp

ARG walletuserid
ARG walletgroupid
RUN groupadd -g $walletgroupid walletuser && useradd -r -u $walletuserid -g walletuser walletuser
USER walletuser 
