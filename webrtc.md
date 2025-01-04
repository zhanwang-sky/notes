# WebRTC

## build from source

**depot_tools**

```
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
echo 'export PATH=/path/to/depot_tools:$PATH' > prepenv
source ./prepenv
```

**fetching**

```
mkdir webrtc-checkout
cd webrtc-checkout
fetch --nohooks webrtc
gclient sync
```

**building**

```
cd src
gn gen out/Default --args='target_os="mac" target_cpu="arm64" is_debug=true'
ninja -C out/Default all
```
