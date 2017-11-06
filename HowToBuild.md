# sdl_implementation_reference
## Build Instruction

### 1. System requirement
Your operating system should be Ubuntu16.04 x64, other OS may be not compatible.

### 2. Install compiler and develop tools.
#### For x86 desktop:
```shell
$sudo apt install git cmake pkg-config build-essential
```
#### For armhf desktop:
```shell
$sudo apt install git cmake pkg-config gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf
```

### 3. Install support packages for 32-bit architecture
```shell
$sudo apt-get install lib32ncurses5 lib32z1 lib32stdc++6
```

### 4. Install all the packages for sdl dependence
#### For x86 desktop:
```shell
$sudo apt install libudev-dev libsqlite3-dev libssl-dev libbluetooth-dev libplist-dev libusbmuxd-dev
```
#### For armhf desktop:
We saved all the packages on the github, you just need download and install as follow.
```shell
$git clone https://github.com/luwanjia/sdl_libraries.git
$cd sdl_libraries
$sudo ./install.sh
```
And you can also uninstall all the packages by uninstall.sh command.
```shell
$sudo ./uninstall.sh
```

### 5. Build and Run

#### a) Get source code.
```shell
$git clone https://github.com/luwanjia/sdl_implementation_reference.git
$cd sdl_implementation_reference
$git checkout feature/tag_4.2.3_develop
```
#### b) cmake && make && make install
##### For x86 desktop:
```shell
$cd sdl_implementation_reference
$mkdir build
$cd build
$cmake -DCMAKE_BUILD_TYPE="Release" ..
$make
$make install
```
##### For armhf desktop:
```shell
$cd sdl_implementation_reference
$mkdir build
$cd build
$cmake \
-DCMAKE_BUILD_TYPE="Release" \
-DCMAKE_CXX_COMPILER="arm-linux-gnueabihf-g++" \
-DCMAKE_C_COMPILER="arm-linux-gnueabihf-gcc" \
-DUDEV_PATH_INCLUDES="/usr/arm-linux-gnueabihf/include" \
-DUDEV_PATH_LIB="/usr/arm-linux-gnueabihf/lib" \
-DOPENSSL_ROOT_DIR="/usr/arm-linux-gnueabihf/lib" \
-DOPENSSL_LIBRARIES="/usr/arm-linux-gnueabihf/lib" \
-DOPENSSL_INCLUDE_DIR="/usr/arm-linux-gnueabihf/include" \
-DSQLITE3_LIBRARIES="/usr/arm-linux-gnueabihf/lib" \
-DSQLITE3_INCLUDE_DIRS="/usr/arm-linux-gnueabihf/include" ..
$make
$make install
```
