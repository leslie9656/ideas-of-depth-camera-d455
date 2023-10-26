# Application of depth-camera-D455
## Introduction

本项目主要记录作者在如何使用Intel Depth Camera D455相机完成一些简单操作的一些心得和遇到的一些问题。Google上有很多相关的优质文章，作者会根据这些优质的内容进行学习，并引用相关链接。由于作者对于这方面也是初学者，若在描述过程中产生问题，欢迎纠错，您的疑问将是我学习的动力。

## Installation of Depth-Camera-D455

**Software**
1. Ubuntu 20.04
2. Ros-Noetic
3. Intel RealSense SDK

**1.Installation of Ubuntu 20.04**
ubuntu的安装有虚拟机和双系统可以选择。为了后续学习和工作的便利，作者选择安装双系统。相应的教程链接会在下方给出，有需要的读者可以自行安装。需要注意的是，其他版本的ubuntu也可以使用，在此项目中，作者使用的ubuntu版本为20.04。
100 Mb/秒s
Link：https://www.bilibili.com/video/BV1554y1n7zv/?spm_id_from=333.999.0.0&vd_source=d933a24fe6d874814e1e9c6a40709b10


**2.Installation of ROS-Noetic**
Ros官方有十分详细的安装文档，需要的读者可以直接Link。需要注意的是，作者在此项目中使用的ubuntu版本为：20.04，因此需要下载对应的ROS版本：Noetic.请在下载时注意版本序号，避免安装错误。

Link: http://wiki.ros.org/noetic/Installation/Ubuntu

**3. Installation of Intel RealSense SDK**
在安装完ROS后，SDK的安装是必须的。以下是安装SDK的步骤：
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
5. 测试安装结果（源神启动）
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
    sudo apt-get install ros-noetic-imu-filter-madgwick
    sudo apt-get install ros-noetic-rtabmap-ros
    sudo apt-get install ros-noetic-robot-localization
```
**5. 测试安装的软件**
在安装完所有资源后，接通相机。
```
1. 查询点云
    roslaunch realsense2_camera rs_camera.launch (后面可以加个filters:=pointcloud来查看点云）
2. 打开另一个终端
    rviz
```
Rviz启动后，执行以下步骤：
1. 将固定坐标系更改为camera_link
点击Add按钮，就会出现一个设置菜单。然后点击By Topic选项卡。
2. 在/color -> /image_raw下双击Image
3. 在/depth -> /color -> /points下双击PointCloud2
4. 点击Ok

## Launching ORB_SLAM2 by D455 at ROS
ORB_SLAM2算法主要用于标定图像内物体的特征点。通过ORB_SLAM2算法，我们可以得到图像内物体的特征点，便于进一步对物体进行识别。以下是实现过程。

```
cd ~/catkin_ws/src/
git clone https://github.com/appliedAI-Initiative/orb_slam_2_ros.git
cd ..
catkin_make
source devel/setup.bash
roslaunch realsense2_camera rs_rgbd.launch
//新增一个窗口
rosrun tf static_transform_publisher 0.011 0.048 0.015 0 0 0 base_link camera_link 100
//新增一个窗口
roslaunch orb_slam2_ros orb_slam2_d435_rgbd.launch
//新增一个窗口
rviz
//新增一个窗口
rosrun rqt_tf_tree rqt_tf_tree
//新增一个窗口
rostopic list
rostopic echo /orb_slam2_rgbd/pose
```

## Launching YOLOV5 algorithm by D455 
YOLOV5 算法用于给图像内物体标定标签和 rrounding box。以下为实现过程。
在这里，我们使用了conda虚拟环境。这个环境可以方便我们对与安装和卸载后续不必要的依赖。以下是配置conda的过程。需要注意，这里已经默认了你已经安装好了accoconda。
```
conda activate -n Yolov5 python=3.8
conda activate Yolov5
```
这样你就进入到一个名为Yolov5的虚拟环境内。接下来是Yolov5的实现。
```
cd ~/catkin_ws/src/
git clone  https://github.com/killnice/yolov5-D435i.git
cd ..
catkin_make
cd yolov5-D435i
conda activate Yolov5
pip install -r requirements.txt
pip install pyrealsense2
python main_debug.py
```

## Recording the ROS bag with D455
在这里，作者希望记录带有特征点的topic，过程如下：
```
cd ~/catkin_ws/src/
roslaunch realsense2_camera rs_rgbd.launch
//新增一个窗口
rosrun tf static_transform_publisher 0.011 0.048 0.015 0 0 0 base_link camera_link 100
//新增一个窗口
roslaunch orb_slam2_ros orb_slam2_d435_rgbd.launch
//新增一个窗口
rviz
//新增一个窗口
rosrun rqt_tf_tree rqt_tf_tree
//新增一个窗口
rostopic list
rostopic echo /orb_slam2_rgbd/pose
//开始录包
rosbag record /orb_slam2_rgbd/debug_image
```

## 对于记录好的包进行抽帧
```
cd ~/catkin_ws/src/
roslaunch out1.launch
```
照片会被保存在~/.ros文件夹中

## 使用d455完成深度相机视频录制
```
cd ~/catkin_ws/src/
rosrun D455_get_RGB_Video_Capture.py
```


## D455 imu topic 调出并结合rtabmap使用
```
roslaunch realsense2_camera rs_camera.launch
roslaunch realsense2_camera rs_camera.launch enable_gyro:=true
roslaunch realsense2_camera rs_camera.launch enable_gyro:=true enable_accel:=true
roslaunch realsense2_camera rs_camera.launch enable_gyro:=true enable_accel:=true unite_imu_method:=linear_interpolation
rostopic echo /camera/imu
```
此时，imu处于打开状态

接下来，将imu转换成odom，为lidar开始mapping作准备.
link:http://wiki.ros.org/rtabmap_ros/Tutorials/HandHeldMapping
```
roslaunch realsense2_camera rs_camera.launch \
    align_depth:=true \
    unite_imu_method:="linear_interpolation" \
    enable_gyro:=true \
     enable_accel:=true

rosrun imu_filter_madgwick imu_filter_node \
    _use_mag:=false \
    _publish_tf:=false \
    _world_frame:="enu" \
    /imu/data_raw:=/camera/imu \
    /imu/data:=/rtabmap/imu

roslaunch rtabmap_launch rtabmap.launch \
    rtabmap_args:="--delete_db_on_start --Optimizer/GravitySigma 0.3" \
    depth_topic:=/camera/aligned_depth_to_color/image_raw \
    rgb_topic:=/camera/color/image_raw \
    camera_info_topic:=/camera/color/camera_info \
    approx_sync:=false \
    wait_imu_to_init:=true \
    imu_topic:=/rtabmap/imu
```
完成imu转odom。
