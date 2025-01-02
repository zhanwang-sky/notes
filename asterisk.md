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

### basic config

**`asterisk.conf`**
```
debug=1
```

**`cdr.conf`**
```
[radius]
radiuscfg => /etc/radcli/radiusclient.conf
```

**`cel.conf`**
```
radiuscfg => /etc/radcli/radiusclient.conf
```

**`logger.conf`**
```
;console => notice,warning,error
;messages.log => notice,warning,error
full.log => notice,warning,error,debug,verbose,dtmf,fax
```

**`extension.conf`**
```
[from-internal]
exten = 1000,1,Answer()
same = n,Wait(1)
same = n,Playback(hello-world)
same = n,Hangup()
```

**`pjsip.conf`**
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

### music on hold

**ensure audio format:**</br>
`ffmpeg -i the_good_the_bad_and_the_ugly.mp3 -ar 8000 -ac 1 -acodec pcm_s16le the_good_the_bad_and_the_ugly.wav`

**`musiconhold.conf`**
```
[got]
mode=files
directory=/path/to/got

[tgtbtu]
mode=files
directory=/path/to/tgtbtu
```

**`extension.conf`**
```
[from-internal]
exten = 1000,1,Answer()

; will play default moh
same = n,StartMusicOnHold()
same = n,Wait(10)
same = n,Playback(hello-world)

; will play `Game of Thrones`
same = n,StartMusicOnHold(got)
same = n,Wait(10)
same = n,Playback(hello-world)

; will play `the Good the Bad and the Ugly`
same = n,Set(CHANNEL(musicclass)=tgtbtu)
same = n,StartMusicOnHold(got)
same = n,Wait(10)
same = n,Playback(hello-world)

same = n,Hangup()
```
