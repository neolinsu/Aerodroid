#  飞行模拟环境搭建参考文档
------------
## 00.基础环境

>已经通过测试的基础环境配置如下：
```
Ubuntu 16.04.3 LTS
Linux version 4.4.0-104-generic (buildd@lgw01-amd64-022)
gcc version 5.4.0 20160609
gazebo 8.1.1
```
推荐配置：
```
Ubuntu 16.04 LTS amd64
gcc version 5.4 or higher
gazebo 8 or higher
```

模拟环境的搭建对系统环境没有太多需求，只需要是稳定的Ubuntu-16.04-LTS和官方更新源中最新的编译基础环境即可。在终端中输入如下命令：
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential
```

经测试，gazebo官方源中的gazebo8即符合要，下载[Preparation-Script](https://github.com/neolinsu/Aerodroid/tree/master/Preparation-Scripts)中的gazebo8_install.sh文件。终端进入文件所在的文件夹，使用以下命令安装。
```
./gazebo8_install.sh
```
----------  

## 01.Ardupilot

>ArduPilot is an open source, unmanned vehicle Autopilot Software Suite. [Wiki](https://en.wikipedia.org/wiki/ArduPilot)

Ardupilot[[Github](https://github.com/ArduPilot/ardupilot)]是开源的无人机飞行控制项目，主要负责无人机的控制细节。在模拟环境中同样需要这一模块对无人机虚拟模型的细节进行控制,但不必过于注意这一项目的实现细节。为了简化说明，这里以四轴无人机（ArduCopter）为例给出配置Ardupilot的参考命令。

安装git。
```
sudo apt-get -f install git
```

将Ardupilot项目克隆至本地。(本文档中，git保存的位置皆为`~`)
```
cd ~
git clone https://github.com/ArduPilot/ardupilot.git
cd ardupilot
git submodule update --init --recursive
```

初始化Ardupilot编译环境。
```
Tools/scripts/install-prereqs-ubuntu.sh -y
sudo apt-get install gcc-avr avrdude avr-libc binutils-avr
sudo apt-get install python-serial python-wxgtk2.8 python-matplotlib python-opencv python-pexpect python-scipy
```

进入`ardupilot/ArduCopter`文件夹进行编译。
```
cd ArduCopter
make sitl
```

运行以下指令对sitl进行初始化操作。
```
export PATH=$PATH:$HOME/ardupilot/Tools/autotest
export PATH=/usr/lib/ccache:$PATH
. ~/.bashrc
sim_vehicle.py -w
```
编译输出的`arducopter`将在`ardupilot/build/sitl/bin`中。

-------------  

## 02.Intel® RealSense™ SDK

> This project is a cross-platform library (Linux, Windows, Mac) for capturing data from the Intel® RealSense™ F200, SR300, R200, LR200 and the ZR300 cameras.[[Github](https://github.com/IntelRealSense/librealsense/tree/v1.12.1)]

需要特别注意的是目前Intel®提供了最新的RealSense™ SDK支持的是D400系列和SR300传感器，而无人机搭载的是R200传感器，需要使用`v1.12.1`版本的SDK。

安装依赖包。
```
sudo apt-get install libusb-1.0-0-dev pkg-config
sudo apt-get install libglfw3-dev cmake
```

将项目克隆至本地。
```
cd ~
git clone https://github.com/IntelRealSense/librealsense.git
cd librealsense
```

cmake预备，make编译且安装。
```
mkdir build
cd build
cmake ..
make && sudo make install
```
-------------  

## 03.Gazebo Plugins
要在模拟器中运行带RealSense™传感器的无人机模拟模型，需要安装两个插件。  

#### Gazebo SITL Plugin
> A ROS-independent Gazebo plugin for Ardupilot's SITL.[[Github](https://github.com/intel/gazebo-sitl)]

将项目克隆至本地。
```
cd ~
git clone https://github.com/intel/gazebo-sitl.git
cd gazebo-sitl
```

更新下载项目依赖的子模块。
```
git submodule update --init --recursive
```

cmake预备，make编译且安装。
```
mkdir build
cd build
cmake ..
sudo make install
```    

#### Gazebo RealSense™ Plugin
> A RealSense Camera Gazebo plugin.[[Github](https://github.com/intel/gazebo-realsense)]

将项目克隆至本地。
```
cd ~
git clone https://github.com/intel/gazebo-realsense.git
cd gazebo-realsense
```

cmake预备，make编译且安装。
```
mkdir build
cd build
cmake ..
sudo make install
```  
-------------  

## 04.collision-avoidance-library

>A framework for testing and benchmarking collision avoidance strategies. [[Github](https://github.com/neolinsu/collision-avoidance-library)]

安装依赖包。
```
sudo apt-get install git cmake libglm-dev python-future doxygen libusb-1.0-0-dev libglfw3-dev socat
```

将项目克隆至本地。
```
cd ~
git clone https://github.com/neolinsu/collision-avoidance-library.git
cd collision-avoidance-library
git submodule update --init --recursive
```

cmake预备，make编译。编译生成的`collision-avoidance-library/build/tools/coav-control/coav-control`为模拟环境主要要用到的工具。
```
mkdir build
cd build
cmake ../ -DWITH_REALSENSE=OFF -DWITH_GAZEBO=ON -DWITH_VDEBUG=ON -DWITH_TOOLS=ON
make
```
其中cmake的参数（参数名前需要加`-D`）定义为：  

|特征描述|编译参数|默认值|
|-|-|-|
|Intel RealSense support|WITH_REALSENSE|ON|
|Gazebo support|WITH_GAZEBO|OFF|
|Visual Debugger support|WITH_VDEBUG|OFF (depends on Gazebo)|
|Coav Tools|WITH_TOOLS|OFF|
|Compile code samples|WITH_SAMPLES|OFF|

因为目前只在模拟环境中运行故关闭了对Intel RealSense硬件的支持。

-------------  

## 05.运行模拟环境

进入`collision-avoidance-library/testbed`执行`coav-sim.sh`：
```
AUTOPILOT=AP_APM APM_DIR="(改成绝对路径)/ardupilot/build/sitl/bin/" ./coav-sim.sh -a QC_STOP -d DI_POLAR_HIST -s ST_GAZEBO_REALSENSE
```
参数将会被传递至此前编译的`coav-control`工具，参数所控制的内容如下表所示。具体内容请执行`./coav-control --help`查看。  

|运行参数|含义|
|-|-|
|-a|遇到碰撞时应执行的策略|
|-d|检测碰撞的方法|
|-s|使用的传感器|

在另一个终端中使用gazebo可视化界面查看模拟运行的状况。
```
gzclient
```
