# MYNT小觅相机在TX2上运行的简单DEMO 基于ROS

本项目做了一个简单的demo，能够使用在ros中显示小觅相机的图像信息。

## I. 安装OPENCV+ROS

### i. 安装OPENCV

TX2的操作系统是ubuntu18.04，安装opencv的过程与之前有所不同。过程参考[嵌入式Xavier中配置ROS+OPENCV](http://www.yujincheng.net/2019/03/14/ros+opencv.html)。

1. 安装环境依赖
    ```
    sudo apt-get install build-essential
    sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
    sudo apt-get install python-dev python-numpy python3-dev python3-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev  libdc1394-22-dev
    
    # 注意一定要安装VTK，不然opencv编译时没有viz等模块，影响image_pipeline的编译。
    sudo apt-get install libvtk6-dev
    ```

2. 下载编译OPENCV
   这里安装的是opencv 3.3.1
    ```
    git clone -b 3.3.1 https://github.com/opencv/opencv
    git clone -b 3.3.1 https://github.com/opencv/opencv_contrib
    cd opencv
    mkdir build
    cd build
    # 上述是基本操作
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -DCMAKE_INSTALL_PREFIX=/usr/local/ \
        -DINSTALL_PYTHON_EXAMPLES=ON \
        -DINSTALL_C_EXAMPLES=ON \
        -DPYTHON_EXCUTABLE=/usr/bin/python \
        -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \
        -DWITH_CUDA=OFF \
        -DWITH_CUFFT=OFF \
        -DWITH_CUBLAS=OFF \
        -DWITH_TBB=ON \
        -DWITH_V4L=ON \
        -DWITH_QT=ON \
        -DWITH_GTK=ON \
        -DWITH_OPENGL=ON \
        -DENABLE_PRECOMPILED_HEADERS=OFF \
        -DBUILD_EXAMPLES=ON ..
    make -j8
    sudo make install
    ```
    其中`-DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \`指定opencv_contrib的路径。 `-DENABLE_PRECOMPILED_HEADERS=OFF`很关键，因为ubuntu18.04的编译器为 c++7 版本，如果不加这句话会出现`fatal error: stdlib.h: No such file or directory`的错误。

### ii. 安装ROS
ubuntu18.04需要的ros版本是Melodic，具体安装过程参考[ZCU102上SD卡的制作&ROS安装&SP-SLAM的编译运行教程](https://github.com/xxzzll11111/SDcard_ROS_SPSLAM#ii-安装ros)中安装ROS的部分，或者[Debian install of ROS Melodic](http://wiki.ros.org/melodic/Installation/Debian)。这里不再详细描述。

## II. 安装小觅相机和image_view的package

1. 安装小觅相机
    具体安装过程参考[Ubuntu 源码安装](https://mynt-eye-s-sdk-docs-zh-cn.readthedocs.io/zh_CN/latest/src/sdk/install_ubuntu_src.html)和[ORB_SLAM2 如何整合](https://mynt-eye-s-sdk-docs-zh-cn.readthedocs.io/zh_CN/latest/src/slam/orb_slam2.html)

    ```
    git clone https://github.com/slightech/MYNT-EYE-S-SDK

    # 运行一个简单的demo
    cd MYNT-EYE-S-SDK
    make init
    make samples
    ./samples/_output/bin/camera_with_junior_device_api

    # 编译ros的package
    make ros
    source ./wrappers/ros/devel/setup.bash
    ```

2. 安装image_view
   image_view包含在image_pipeline中，用于显示ros中的图像topic。安装及使用参考[ROS官网](http://wiki.ros.org/image_view?distro=melodic)
    ```
    # 在catkin_workspace的src目录下
    git clone https://github.com/ros-perception/image_pipeline
    cd ..
    catkin_make
    source ./devel/setup.bash
    ```

3. 运行小觅相机和image_view
    ```
    # 运行小觅相机的节点
    roslaunch mynt_eye_ros_wrapper mynteye.launch

    # 打开一个新的terminal，运行image_view
    rosrun image_view image_view image:=/mynteye/left_rect/image_rect
    ```
    运行`rostopic list`可以看到小觅相机所有的topic，运行`rosrun image_view image_view image:=<topic name>`可以看到每个图像的信息。
    ```
    /mynteye/camera_mesh
    /mynteye/depth/camera_info
    /mynteye/depth/image_raw
    /mynteye/disparity/camera_info
    /mynteye/disparity/image_norm
    /mynteye/disparity/image_raw
    /mynteye/imu/data_raw
    /mynteye/left/camera_info
    /mynteye/left/image_raw
    /mynteye/left_rect/camera_info
    /mynteye/left_rect/image_rect
    /mynteye/points/data_raw
    /mynteye/right/camera_info
    /mynteye/right/image_raw
    /mynteye/right_rect/camera_info
    /mynteye/right_rect/image_rect
    /mynteye/temperature/data_raw
    ```

