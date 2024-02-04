# 开始之前（若是 WSL2）
请确保 usb 相机连接到 WSL2，以下是 WSL2 连接 usb 流程 ：
1. Windows 下安装 [usbipd-win](https://github.com/dorssel/usbipd-win/releases)（选择 msi 文件，直接安装）
2. 在 Linux 中安装 USBIP 工具和硬件数据库
```shell
sudo apt install linux-tools-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*-generic/usbip 20
```
3. Attach usb
```shell
# 管理员运行powershell
usbipd wsl list
usbipd wsl attach -b <busid>
# ubuntu下查看usb设备
lsusb
```
# 步骤
1. 安装 `camera_calibration` 功能包，其中 `humble` 为 ROS 版本
```shell
sudo apt install ros-humble-camera-calibration
```
2. 启动标定程序。`size` 为横向内角点 x 纵向内角点，`square` 为方格边长（单位米），`remap` 为图像话题来源，就是视频流转换后的话题
```shell
ros2 run camera_calibration cameracalibrator --size 11x8 --square 0.025 --ros-args --remap image:=/stream_to_topic/stream_image
```
- 注意 `size` 参数中为字母 `x`
- 遇到报错可以尝试 `pip install -U numpy`
3. 将标定板左右上下前后倾斜移动，直到 **X**（上下）、**Y**（左右）、**Size**（大小）、**Skew**（倾斜）这几项都变成绿色，点击 **CALIBRATE**（校准）按钮可以看到矫正效果
4. 再将标定板放在相机视野内，右上角会显示标定结果的 `lin.`（线性误差），这个值越低越好，低于 0.1 最好
5. 也可以调节 **Camera type** 来选择相机类型，调节 **scale** 来控制显示多少画面
6. 最后点击 **SAVE** 按钮可以保存参数文件到 `/tmp/calibrationdata.tar.gz`，其中的 `ost.txt` 和 `ost.yaml` 就是参数文件
# camera_calibration 参数详细
```shell
Camera Name:

-c, --camera_name
        name of the camera to appear in the calibration file


Chessboard Options:

You must specify one or more chessboards as pairs of --size and--square options.

  -p PATTERN, --pattern=PATTERN
                    calibration pattern to detect - 'chessboard','circles', 'acircles','charuco'
  -s SIZE, --size=SIZE
                    chessboard size as NxM, counting interior corners (e.g. a standard chessboard is 7x7)
  -q SQUARE, --square=SQUARE
                    chessboard square size in meters

ROS Communication Options:

 --approximate=APPROXIMATE
                    allow specified slop (in seconds) when pairing images from unsynchronized stereo cameras
 --no-service-check
                    disable check for set_camera_info services at startup

Calibration Optimizer Options:

 --fix-principal-point
                    fix the principal point at the image center
 --fix-aspect-ratio
                    enforce focal lengths (fx, fy) are equal
 --zero-tangent-dist
                    set tangential distortion coefficients (p1, p2) to
                    zero
 -k NUM_COEFFS, --k-coefficients=NUM_COEFFS
                    number of radial distortion coefficients to use (up to
                    6, default 2)
 --disable_calib_cb_fast_check
                    uses the CALIB_CB_FAST_CHECK flag for findChessboardCorners

    This will open a calibration window which highlight the checkerboard.
```
# 参考资料
- [ROS 2 Humble 标定纠正畸变全景鱼眼展开网络摄像头](https://blog.csdn.net/duanluan/article/details/131091855)
- [WSL 连接 USB](https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb)
