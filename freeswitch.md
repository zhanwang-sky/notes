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
./configure --prefix=/home/work/.local/sofia-sip
make
make install
```

**编译并安装`SpanDSP`**

```
git clone https://github.com/freeswitch/spandsp.git
cd spandsp
./bootstrap.sh
./configure --prefix=/home/work/.local/spandsp
make
make install
```

**编译`FreeSWITCH`**

*由于`sofia-sip`和`spandsp`安装到了自定义目录，故需修改`PKG_CONFIG_PATH`环境变量，使`pkgconfig`能找到*
```
export PKG_CONFIG_PATH=/home/work/.local/sofia-sip/lib/pkgconfig:/home/work/.local/spandsp/lib/pkgconfig
```

```
git clone https://github.com/signalwire/freeswitch.git
cd freeswitch
./bootstrap.sh
./configure --prefix=/home/work/freeswitch
make
make install
```

*下列模块可能编不过，如不需要可先注掉</br>
（从`build/modules.conf.in`中注掉）*
```
mod_verto
mod_signalwire
mod_av
mod_spandsp
```

## post-install

**systemd**

编辑`debian/freeswitch-systemd.freeswitch.service`示例文件，修改如下地方：
```
12c12
< PIDFile=/run/freeswitch/freeswitch.pid
---
> PIDFile=/home/work/freeswitch/var/run/freeswitch/freeswitch.pid
14,19c14,17
< Environment="USER=freeswitch"
< Environment="GROUP=freeswitch"
< EnvironmentFile=-/etc/default/freeswitch
< ExecStartPre=/bin/chown -R ${USER}:${GROUP} /var/lib/freeswitch /var/log/freeswitch /etc/freeswitch /usr/share/freeswitch /var/run/freeswitch
< ExecStart=/usr/bin/freeswitch -u ${USER} -g ${GROUP} -ncwait ${DAEMON_OPTS}
< TimeoutSec=45s
---
> Environment="USER=work"
> Environment="GROUP=work"
> ExecStart=/home/work/freeswitch/bin/freeswitch -u ${USER} -g ${GROUP} -ncwait ${DAEMON_OPTS}
> TimeoutSec=10s
34c32
< UMask=0007
---
> UMask=0002
```

拷贝到system目录
```
sudo cp debian/freeswitch-systemd.freeswitch.service /etc/systemd/system/freeswitch.service
```

重新加载daemon文件
```
sudo systemctl daemon-reload
```

**修改配置**

修改`autoload_configs/modules.conf.xml`，调整需要启用的模块

修改`autoload_configs/logfile.conf.xml`，调整单个日志文件大小

修改`vars.xml`，调整公网SIP和RTP地址

## unimrcp

**下载依赖库源码**

从[unimrcp.org](http://www.unimrcp.org/downloads/dependencies)下载最新依赖库源码包。

**编译并安装`apr`**

```
cd libs/apr
./configure --prefix=/home/work/.local/apr
make
make install
```

**编译并安装`apr-util`**

```
cd libs/apr-util
./configure --prefix=/home/work/.local/apr --with-apr=/home/work/.local/apr
make
make install
```

**编译并安装`unimrcp`**

```
git clone https://github.com/unispeech/unimrcp.git
cd unimrcp
./bootstrap
./configure --prefix=/home/work/.local/unimrcp --with-apr=/home/work/.local/apr --with-apr-util=/home/work/.local/apr --with-sofia-sip=/home/work/.local/sofia-sip
make
make install
```

**编译并安装`mod_unimrcp`**

*设置`PKG_CONFIG_PATH`环境变量*
```
export PKG_CONFIG_PATH=/home/work/freeswitch/lib/pkgconfig:/home/work/.local/unimrcp/lib/pkgconfig
```

```
git clone https://github.com/freeswitch/mod_unimrcp.git
cd mod_unimrcp
./bootstrap.sh
./configure
make
make install
```
