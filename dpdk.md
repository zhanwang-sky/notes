# DPDK

## Build & Install

**Pre-request**

```
apt install build-essential exuberant-ctags doxygen sphinx-common \
pkg-config python3-pip python3.12-venv \
libnuma-dev libfdt-dev libarchive-dev libbsd-dev libjansson-dev libssl-dev libpcap-dev libelf-dev
```

```
pip install meson ninja pyelftools
```

**Build**

```
meson setup -Dplatform=generic build
cd build
ninja
```

**Install**

```
meson install
ldconfig
```

## Play

列出系统中所有可被DPDK接管或特殊照顾的设备：
```
./usertools/dpdk-devbind.py --status
```
