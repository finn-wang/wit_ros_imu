# Witmotion imu driver for ROS
## 说明
1. 此仓库为维特智能IMU的ROS驱动，版权归维特智能公司所有
2. 详细教程可见维特智能官方教程：https://wit-motion.yuque.com/wumwnr/docs/gpyqkc
3. 因为该公司没有将代码上传到Github，导致学习和阅读非常不方便，所以本人将其上传到github，方便大家git clone
## 测试环境
测试环境：ubuntu20.04 + noetic
**如果你是低版本的Ubuntu系统，请将功能包内scripts目录下的所有文件的Shebang行改为python3**
## IMU软件包使用

### 1. 安装 ROS IMU 依赖
请在终端运行对应的命令

   如果你使用的是 ubuntu 16.04，ROS kinetic，python2 :

   ```
   sudo apt-get install ros-kinetic-imu-tools ros-kinetic-rviz-imu-plugin
   sudo apt-get install python-visual
   ```

   如果你使用的是 ubuntu 18.04，ROS Melodic，python2 :

   ```
   sudo apt-get install ros-melodic-imu-tools ros-melodic-rviz-imu-plugin
   ```

   如果你使用的是 ubuntu 20.04，ROS Noetic，python3 :

   ```
   sudo apt-get install ros-noetic-imu-tools ros-noetic-rviz-imu-plugin
   pip3 install pyserial
   ```

### 2. 建立工作空间
1. 建立工作空间，将代码clone到你的工作空间
2. 为所有的.py文件添加可执行权限

打开命令终端，运行下面指令：
 ```
   mkdir -p your_workspace/src
   catkin_make
   cd your_workspace/src
   git clone https://github.com/Stoner-F/wit_ros_imu.git
   cd wit_ros_imu/src/scripts/
   sudo chmod +x *.py
   ```

## 2.3 ROS 驱动和可视化
以 ubuntu16.04，JY901S，python2.7 为例

1. 查看USB端口号。先不要插 IMU 的 USB ，在终端输入 `ls  /dev/ttyUSB*` 来检测一下，然后在将 USB 插入电脑，再在终端输入 `ls  /dev/ttyUSB*` 来检测一下，多出来的 ttyUSB 设备就是 IMU 的串口。

2. 修改参数配置。需要修改的参数包括设备类型，USB端口号和波特率。进入脚本目录~/wit/wit_ros_imu/src/launch，修改对应的 launch 文件中的配置参数。设备类型如果是modbus协议的就填modbus，使用wit标准协议的填normal,如果是MODBUS高精度协议的填hmodbus，如果是CAN协议的就填can，如果是CAN高精度协议的填hcan。设备号/dev/ttyUSB0（脚本默认用的 /dev/ttyUSB0）为你电脑识别出来的数字。波特率根据实际使用设定，JY6x系列模块默认波特率为115200,CAN模块为230400，其他模块为9600,如果用户通过上位机修改了波特率，需要对应修改成修改后的波特率。

3. 给对应的串口管理员权限，在终端输入：`sudo chmod 777 /dev/ttyUSB0`，提示你输入管理员密码，输入密码后回车即可。注意每次重新插入USB口都需要重新给串口赋管理员权限。

4. 如果使用的产品是Modbus协议的，还需要安装一下Modbus的依赖库，在终端输入：`pip install modbus_tk`

5. 打开终端，运行launch文件
   ```
   roslaunch wit_ros_imu display_and_imu.launch
   ```

![Description](https://witpic-1253369323.cos.ap-guangzhou.myqcloud.com/img-md/2a7ddca1-016c-49d1-a85d-b4568a468027)
   
【注意】：`display_and_imu.launch`文件，即IMU的可视化显示只支持Ubuntu16.04系统，如果您使用的是高版本的ubuntu，则无法使用此功能，但是可以使用`roslaunch wit_ros_imu rviz_and_imu.launch`进行Rviz中的可视化显示。

打开两个新终端输入分别输入下面几行命令

   ```
   rostopic echo /wit/imu
   rostopic echo /wit/mag
   rostopic echo /wit/location     #经纬度解析只有WIT私有协议 即normal才有
   ```

1. 同理，如需要运行其他 launch 文件，需要先确保 launch 文件中的 /dev/ttyUSB0 设备修改对。

2. 相关文件说明

   * display_and_imu.launch，打开打开 IMU 驱动节点和用 visual 编写的可视化模型。（仅支持 ubuntu 16.04）
   * wit_imu.launch，打开用 IMU 驱动节点。
   * rviz_and_imu.launch，打开 IMU 驱动节点和 Rviz 可视化。

# 3. 串口助手测试通讯

以 ubuntu16.04 为例，波特率 9600

1. 安装 cutecom（你也可以安装其他的串口助手进行调试）。

   ```
   apt-get install cutecom -y
   ```

2. 插入串口模块，需要通过命令ls /dev/ttyU*查看新加入的串口模块的端口号，然后对其赋予读写权限，以/dev/ttyUSB0为例，输入指令：sudo chmod 777 /dev/ttyUSB0，然后根据提示输入管理员密码。注意每次新插入USB模块都需要重新赋读写权限。

3. 安装成功后在终端输入 cutecom，打开串口助手，然后进行一些设置，注意从下拉框中选择的串口号是不对的，需要根据上面第二步中获取到的串口号进行修改，比如/dev/ttyUSB0，如图中所示。  

![Description](https://witpic-1253369323.cos.ap-guangzhou.myqcloud.com/img-md/f827f353-7ad0-4c5f-83e2-0d117ad34d41)
   

1. 然后我们点击`open device` ，此时下面的空白面板会有 imu 的数据打印。

2. 我们可以等待 imu 的数据打印一会儿，然后点击 `close device`来查看。

3. 如果可以找到`55 51`、 `55 52`、 `55 53`开头的信息，那么模块发送的数据是没有问题的，如果有数据但没有找到正确包头的数据，则需要核对一下波特率设置，切换到正确的波特率就可以显示了。

   

