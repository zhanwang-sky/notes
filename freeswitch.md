# FreeSWITCH

## build from source

**安装依赖**

```
sudo apt install pkgconf libcurl4-openssl-dev libedit-dev \
libldns-dev liblua5.4-dev libmariadb-dev libopus-dev libpcre3-dev libpq-dev \
libsndfile-dev libsqlite3-dev libssl-dev libtiff-dev libtool libtool-bin \
libspeex-dev libspeexdsp-dev uuid-dev \
ffmpeg libavcodec-dev libavdevice-dev libavfilter-dev libavformat-dev libpostproc-dev libswresample-dev libswscale-dev
```

**编译并安装`Sofia-SIP`**

```
git clone https://github.com/freeswitch/sofia-sip.git
cd sofia-sip
./bootstrap.sh
./configure --prefix=/home/work/fs-deps/sofia-sip
make
make install
```

**编译并安装`SpanDSP`**

```
git clone https://github.com/freeswitch/spandsp.git
cd spandsp
./bootstrap.sh
./configure --prefix=/home/work/fs-deps/spandsp
make
make install
```

**编译`FreeSWITCH`**

*由于sofia-sip和spandsp安装到了自定义目录，故需修改PKG_CONFIG_PATH环境变量，使pkgconfig能找到*

```
export PKG_CONFIG_PATH=/home/work/fs-deps/sofia-sip/lib/pkgconfig:/home/work/fs-deps/spandsp/lib/pkgconfig:$PKG_CONFIG_PATH
```

```
git clone https://github.com/signalwire/freeswitch.git
cd freeswitch
./bootstrap.sh
./configure --prefix=/home/work/freeswitch
make
make install
```