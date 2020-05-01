# Builds a Docker image that contains xid, and that will run the xid
# GSP process.  It will expose the HTTP REST interface as well as the
# ordinary JSON-RPC interface.

FROM xaya/libxayagame AS build
RUN apt-get install -y \
  autoconf \
  autoconf-archive \
  automake \
  build-essential \
  git \
  libtool \
  libcurl4-openssl-dev \
  libmicrohttpd-dev \
  pkg-config \
  protobuf-compiler

# Build and install xid.
WORKDIR /usr/src/xid
COPY . .
RUN make distclean || true
RUN ./autogen.sh && ./configure && make && make install-strip
RUN ldconfig

# Find and extract the shared libraries needed to run xid.  Collect just
# the minimal required stuff to run xid in the final image.
RUN git clone https://github.com/hemanth/futhark /usr/src/scripts
WORKDIR /jail
RUN mkdir bin && cp /usr/local/bin/xid bin/
RUN bash /usr/src/scripts/sh/cpld.bash bin/xid lib64
RUN mkdir xayagame log

# Build the final image.
FROM scratch
COPY --from=build /jail /
ENV LD_LIBRARY_PATH=/lib64/
LABEL description="XAYA ID game-state processor"
VOLUME ["/xayagame", "/log"]
ENV GLOG_log_dir /log
ENTRYPOINT [ \
  "/bin/xid", \
  "--datadir=/xayagame", \
  "--enable_pruning=1000" \
]
CMD []
