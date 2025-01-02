# Asterisk

## build & install

see [official documentation](https://docs.asterisk.org/Getting-Started/Beginning-Asterisk/)

## configure

```shell
# /etc/asterisk/xxx.conf
sudo make samples
# SysVinit
sudo make config
# logrotate
sudo make install-logrotate
```

**basic config**

`asterisk.conf`
```
debug=1
```

`cdr.conf`
```
[radius]
radiuscfg => /etc/radcli/radiusclient.conf
```

`cel.conf`
```
radiuscfg => /etc/radcli/radiusclient.conf
```

`logger.conf`
```
;console => notice,warning,error
;messages.log => notice,warning,error
full.log => notice,warning,error,debug,verbose,dtmf,fax
```

`extension.conf`
```
[from-internal]
exten = 1000,1,Answer()
same = n,Wait(1)
same = n,Playback(hello-world)
same = n,Hangup()
```

`pjsip.conf`
```
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0

[6000]
type=endpoint
context=from-internal
disallow=all
allow=ulaw
auth=6000
aors=6000

[6000]
type=auth
auth_type=userpass
password=1234
username=6000

[6000]
type=aor
max_contacts=1
```
