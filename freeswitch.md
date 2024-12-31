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

**配置**

**`autoload_configs/unimrcp.conf.xml`**
```xml
<configuration name="unimrcp.conf" description="UniMRCP Client">
  <settings>
    <!-- UniMRCP profile to use for TTS -->
    <param name="default-tts-profile" value="unimrcp-demo2"/>
    <!-- UniMRCP profile to use for ASR -->
    <param name="default-asr-profile" value="unimrcp-demo2"/>
    <!-- UniMRCP logging level to appear in freeswitch.log.  Options are:
         EMERGENCY|ALERT|CRITICAL|ERROR|WARNING|NOTICE|INFO|DEBUG -->
    <param name="log-level" value="DEBUG"/>
    <!-- Enable events for profile creation, open, and close -->
    <param name="enable-profile-events" value="false"/>
    <param name="max-connection-count" value="100"/>
    <param name="offer-new-connection" value="true"/>
    <param name="request-timeout" value="3000"/>
  </settings>

  <profiles>
    <X-PRE-PROCESS cmd="include" data="../mrcp_profiles/*.xml"/>
  </profiles>

</configuration>
```

**`mrcp_profiles/unimrcp-demo2.xml`**
```xml
<include>
  <profile name="unimrcp-demo2" version="2">
    <param name="client-ip" value="$${local_ip_v4}"/>
    <param name="client-port" value="5804"/>
    <param name="server-ip" value="x.x.x.x"/>
    <param name="server-port" value="8060"/>
    <param name="sip-transport" value="udp"/>
    <param name="sip-t1x64" value="3000"/>
    <param name="rtp-ip" value="$${local_ip_v4}"/>
    <param name="rtp-port-min" value="10000"/>
    <param name="rtp-port-max" value="19998"/>
    <param name="codecs" value="PCMU PCMA L16/96/8000 telephone-event/101/8000"/>

    <recogparams>
      <param name="start-input-timers" value="false"/>
    </recogparams>
  </profile>
</include>
```

**`share/freeswitch/grammar/hello.gram`**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<grammar xml:lang="zh-CN" version="1.0" root="ROOT" xmlns="http://www.w3.org/2001/06/grammar">
</grammar>
```

**ESL触发MRCP**

`> telnet ::1 8021`
```
auth ClueCon

sendmsg session-uuid
call-command: execute
execute-app-name: detect_speech
execute-app-arg: unimrcp {CallId=123;CallMethod=inbound;SipFrom=456;SipTo=789;RtpSource=to}hello hello
```
