# ideas-of-depth-camera-d455
## Introduction

本项目主要记录作者在如何使用Intel Depth Camera D455相机完成一些简单操作的一些心得和遇到的一些问题。Google上有很多相关的优质文章，作者会根据这些优质的内容进行学习，并引用相关链接。由于作者对于这方面也是初学者，若在描述过程中产生问题，欢迎纠错，您的疑问将是我学习的动力。

## Installation of Depth-Camera-D455

**Software**
1. Ubuntu 20.04
2. Ros-Noetic
3. Intel RealSense SDK

**1.Installation of Ubuntu 20.04**
ubuntu的安装有虚拟机和双系统可以选择。为了后续学习和工作的便利，作者选择安装双系统。相应的教程链接会在下方给出，有需要的读者可以自行安装。需要注意的是，其他版本的ubuntu也可以使用，在此项目中，作者使用的ubuntu版本为20.04。

Link：https://www.bilibili.com/video/BV1554y1n7zv/?spm_id_from=333.999.0.0&vd_source=d933a24fe6d874814e1e9c6a40709b10


**2.Installation of ROS-Noetic**
Ros官方有十分详细的安装文档，需要的读者可以直接Link。需要注意的是，作者在此项目中使用的ubuntu版本为：20.04，因此需要下载对应的ROS版本：Noetic.请在下载时注意版本序号，避免安装错误。

Link: http://wiki.ros.org/noetic/Installation/Ubuntu

**3. Installation of Intel RealSense SOK**
在安装完ROS后，SDK的安装是必须的。以下是安装SOK的步骤：
```
1. 更新商店以及內核
    sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
2. 从github上下载librealsense 资源
    git clone https://github.com/IntelRealSense/librealsense.git
3. 进入librealsense包
    cd ~/librealsense/
4. 安装核心软件包
    sudo apt-get install git libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev
    sudo apt-get install libusb-1.0-0-dev pkg-config
    sudo apt-get install libglfw3-dev
    sudo apt-get install libssl-dev
    sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/
    sudo udevadm control --reload-rules && udevadm trigger 
    mkdir build
    cd build
    cmake ../ -DBUILD_EXAMPLES=true
    sudo make uninstall && make clean && make && sudo make install
```
5. 测试安装结果
```
    realsense-viewer 
```
**4. RealSense的ROS类**
安装完SOK，安装相关的ros类。以下是安装ros类的步骤：
```
1. 创建一个catkin工作空间：
    mkdir -p ~/catkin_ws/src
    cd ~/catkin_ws/src/
2. 从github上下载RealSense资源
    git clone https://github.com/IntelRealSense/realsense-ros.git
    cd realsense-ros/
    git checkout `git tag | sort -V | grep -P "^2.\d+\.\d+" | tail -1`
    cd ..
3. 下载ddynamic_reconfigure资源
    sudo apt-get install ros-noetic-ddynamic-reconfigure
4. 初始化工作空间
    catkin_init_workspace
    cd ..
    catkin_make clean
5. 安装源码
    catkin_make -DCATKIN_ENABLE_TESTING=False -DCMAKE_BUILD_TYPE=Release
    catkin_make install
6. 配置环境
    echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
    source ~/.bashrc
7. 为SLAM（同步定位与建图）安装软件包
    sudo apt-get install ros-melodic-imu-filter-madgwick
    sudo apt-get install ros-melodic-rtabmap-ros
    sudo apt-get install ros-melodic-robot-localization
```
**5. 测试安装的软件**
在安装完所有资源后，接通相机。
```
1. 查询点云
    roslaunch realsense2_camera rs_camera.launch filters:=pointcloud
2. 打开另一个终端
    rviz
```
Rviz启动后，执行以下步骤：
1. 将固定坐标系更改为camera_link
点击Add按钮，就会出现一个设置菜单。然后点击By Topic选项卡。
2. 在/color -> /image_raw下双击Image
3. 在/depth -> /color -> /points下双击PointCloud2
4. 点击Ok
