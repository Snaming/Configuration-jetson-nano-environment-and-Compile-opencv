# 配置jetson nano环境
## 1、烧写系统
  访问官方的《Getting Started With Jetson Nano Developer Kit》。完成准备安装----将镜像烧写到microSD卡，开机并设置和首次启动信息。
  网址：https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit

## 2、安装依赖
  打开命令终端窗口并执行以下命令：
  sudo apt-get update
  sudo apt-get upgrade
  sudo apt-get install build-essential python3 python3-dev python3-pip libhdf5-serial-dev hdf5-tools nano ntp

## 3、设定虚拟环境
  pip3 install virtualenv
  python3 -m virtualenv -p python3 env --system-site-packages
  echo "source env/bin/activate" >> ~/.bashrc
  source ~/.bashrc

## 4、安装OpenCV
  要在Jetson Nano上安装Open CV，您需要从源代码构建它。构建OpenCV的第一步是在Jetson Nano上定义交换空间。Jetson Nano具有4GBRAM。这不足以从源代码构建OpenCV。因此，我们需要在Nano上定义交换空间以防止内存崩溃。
  #Allocates 4G of additional swap space at /var/swapfile
  sudo fallocate -l 4G /var/swapfile
  #Permissions
  sudo chmod 600 /var/swapfile
  #Make swap space
  sudo mkswap /var/swapfile
  #Turn on swap
  sudo swapon /var/swapfile
  #Automount swap space on reboot
  sudo bash -c 'echo "/var/swapfile swap swap defaults 0 0" >> /etc/fstab'
  #Reboot
  sudo reboot
  现在您应该有足够的交换空间来构建OpenCV。让我们用构建OpenCV的前提条件设置Jetson Nano。
  #Update
  sudo apt-get update
  sudo apt-get upgrade
  #Pre-requisites
  sudo apt-get install build-essential cmake unzip pkg-config
  sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
  sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
  sudo apt-get install libxvidcore-dev libx264-dev
  sudo apt-get install libgtk-3-dev
  sudo apt-get install libatlas-base-dev gfortran
  sudo apt-get install python3-dev
  现在，您应该拥有所需的所有先决条件。因此，让我们继续下载OpenCV的源代码。
  #Create a directory for opencv
  mkdir -p projects/cv2
  cd projects/cv2

  #Download sources
  wget -O opencv.zip https://github.com/opencv/opencv/archive/4.1.0.zip
  wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.1.0.zip

  #Unzip
  unzip opencv.zip
  unzip opencv_contrib.zip

  #Rename
  mv opencv-4.1.0 opencv
  mv opencv_contrib-4.1.0 opencv_contrib
  让我们env为OpenCV准备好虚拟环境（）。
  #Install Numpy
  pip install numpy
  现在，让我们CMake正确设置，以便它为我们的虚拟环境生成正确的OpenCV绑定。
  #Create a build directory
  cd projects/cv2/opencv
  mkdir build
  cd build

  #Setup CMake
  cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D INSTALL_C_EXAMPLES=OFF \
    -D OPENCV_ENABLE_NONFREE=ON \
    #Contrib path
    -D OPENCV_EXTRA_MODULES_PATH=~/projects/cv2/opencv_contrib/modules \
    #Your virtual environment's Python executable
    #You need to specify the result of echo $(which python)
    -D PYTHON_EXECUTABLE=~/env/bin/python \
    -D BUILD_EXAMPLES=ON ~/projects/cv2/opencv
    该cmake命令应显示配置摘要。确保将Interpreter设置为与您的 virtualenv 关联的Python可执行文件。注意：CMake设置中有几个路径，请确保它们与您下载和保存OpenCV源代码的位置匹配。

  要从build文件夹中编译代码，请发出以下命令。
  make -j2
  #Install OpenCV
  sudo make install
  sudo ldconfig
  最后一步是将构建的OpenCV本机库正确链接到您的virtualenv。
  现在，本机库应安装在类似的位置/usr/local/lib/python3.6/site-packages/cv2/python-3.6/cv2.cpython-36m-xxx-linux-gnu.so。
  #Go to the folder where OpenCV's native library is built
  cd /usr/local/lib/python3.6/site-packages/cv2/python-3.6
  #Rename
  mv cv2.cpython-36m-xxx-linux-gnu.so cv2.so
  #Go to your virtual environments site-packages folder
  cd ~/env/lib/python3.6/site-packages/
  #Symlink the native library
  ln -s /usr/local/lib/python3.6/site-packages/cv2/python-3.6/cv2.so cv2.so
  恭喜你！现在您已经完成了从源代码编译OpenCV的工作。
  快速检查一下您是否正确完成了所有操作
  ls -al
  您应该会看到类似
  total 48
  drwxr-xr-x 10 user user 4096 Jun 16 13:03 .
  drwxr-xr-x  5 user user 4096 Jun 16 07:46 ..
  lrwxrwxrwx  1 user user   60 Jun 16 13:03 cv2.so -> /usr/local/lib/python3.6/site-packages/cv2/python-3.6/cv2.so
  -rw-r--r--  1 user user  126 Jun 16 07:46 easy_install.py
  drwxr-xr-x  5 user user 4096 Jun 16 07:47 pip
  drwxr-xr-x  2 user user 4096 Jun 16 07:47 pip-19.1.1.dist-info
  drwxr-xr-x  5 user user 4096 Jun 16 07:46 pkg_resources
  drwxr-xr-x  2 user user 4096 Jun 16 07:46 __pycache__
  drwxr-xr-x  6 user user 4096 Jun 16 07:46 setuptools
  drwxr-xr-x  2 user user 4096 Jun 16 07:46 setuptools-41.0.1.dist-info
  drwxr-xr-x  4 user user 4096 Jun 16 07:47 wheel
  drwxr-xr-x  2 user user 4096 Jun 16 07:47 wheel-0.33.4.dist-info
  要测试OpenCV安装，请运行python并执行以下操作
  import cv2

  #Should print 4.1.0
  print(cv2.__version__)

