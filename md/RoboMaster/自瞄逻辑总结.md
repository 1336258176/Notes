# 坐标系及单位定义

- `gimbal_frame`：云台系（随机器人时刻变化）
- `camera_frame`：相机系（随机器人时刻变化）
- `odom_frame`：惯性系（与机器人相对静止），初始化为上电时 C 板位姿
- [坐标系定义](https://heurm.yuque.com/pwxb48/yy4glk/bslnoq1kqgctcn8u)
# serial_bridge Pkg

- 主要功能：向电控发送字节流数据、接收电控传来的字节流数据
- Topic name：`/driver/serial/receive`、`/driver/serial/send`
- 校验方式：`CRC16`
- 第三方库： https://github.com/wjwwood/serial/tree/main

1. 应该考虑两个通信延迟，`transmit_cost` 和 `INSTimeCost`，其中 `transmit_cost` 需要实测，`transmit_cost` 为物理通信延迟， `INSTimeCost` 为电控解算位姿所花费的时间，因此电控实际发送的时间戳等于接收到的时间戳 - `transmit_cost` - `INSTimeCost`；这两项主要用于时间同步；
3. 发数的同时也需要接收，使用多线程，另一个线程名为 `receive_thread_`，在该变量所属类的析构函数中 `join()`

与电控通信协议如下：
```cpp
  struct UploadCMD_1
  {
    float INSTimeCost{};
    float q_w{};
    float q_x{};
    float q_y{};
    float q_z{};
    uint8_t autoaim_status{};
    uint8_t my_team{};
  } __attribute__((__packed__));
  
  struct UploadCMD_2
  {
    float INSTimeCost{};
    float q_w{};
    float q_x{};
    float q_y{};
    float q_z{};
    float accel[3]{};
    float gyro[3]{};
    float cov_meas[9]{};
  } __attribute__((__packed__));

  
  struct DownloadCMD_1
  {
    uint8_t allow_shoot = 0;
    uint8_t shoot_frequency{};
    float   yaw{};
    float   pitch{};
    uint8_t health_check{};
  } __attribute__((__packed__));
  
  struct DownloadCMD_2
  {
    uint8_t TgtStatus{};
    uint16_t PH1{};
    float Yaw{};
    float Pitch{};
    uint32_t posPredictTrackingCount{};
    uint16_t PH2{};
    uint8_t PH3{};
  } __attribute__((__packed__));
```

目前向下位机通信在用的只有 `DownloadCMD_1`，向电控传的是惯性系下云台的 `yaw` 和 `pitch`，而电控传上来的是 C 板中 IMU 目前的位姿信息（四元数）

# information Pkg

- 主要功能：向自瞄系统提供一些基础信息，如相机与 C 板的相对位姿、队伍颜色、自瞄模式，对 `serial_bridge pkg` 接收到的字节流数据进一步处理

1. 发送 `camera_frame` 到 `gimbal_frame` 的 tf 静态广播
2. 发送 `gimbal_frame` 到 `odom_frame` 的 tf 动态广播，这一步实际是将电控发送上来的云台四元数以 tf 树的形式广播出来
3. 队伍颜色以及自瞄模式是以 ros2异步参数服务器的方式设置的，函数如下

```cpp
void InformationNode::setRecoParamAsync(const rclcpp::Parameter & param)
{
  if (!detector_param_client_->service_is_ready())
  {
    RCLCPP_WARN(get_logger(), "Service not ready, skipping parameter set");
    return;
  }
  
  if (
    !set_param_future_.valid() ||
    set_param_future_.wait_for(std::chrono::seconds(0)) == std::future_status::ready)
    {
    RCLCPP_INFO(get_logger(), "Setting detect_color to %s...", param.as_string().c_str());
    set_param_future_ = detector_param_client_->set_parameters(
      {param},
      [this, param](const ResultFuturePtr & results)
        {
          for (const auto& result : results.get())
          {
            if (!result.successful)
            {
              RCLCPP_ERROR(get_logger(), "Failed to set parameter: %s", result.reason.c_str());
              return;
            }
          }
          RCLCPP_INFO(get_logger(), "Successfully set detect_color to %s!", param.as_string().c_str());
          initial_set_param_ = true;
        });
  }
}
  

void InformationNode::setFCParamAsync(const rclcpp::Parameter & param)
{
  if (!fc_param_client_->service_is_ready())
  {
    RCLCPP_WARN(get_logger(), "Service not ready, skipping parameter set");
    return;
  }

  if (
    !set_fc_param_future_.valid() ||
    set_fc_param_future_.wait_for(std::chrono::seconds(0)) == std::future_status::ready)
    {
    RCLCPP_INFO(get_logger(), "Setting autoaim_mode to %s...", param.as_string().c_str());
    set_fc_param_future_ = fc_param_client_->set_parameters(
      {param},
      [this, param](const ResultFuturePtr & results)
        {
          for (const auto& result : results.get())
          {
            if (!result.successful)
            {
              RCLCPP_ERROR(get_logger(), "Failed to set parameter: %s", result.reason.c_str());
              return;
            }
          }
          RCLCPP_INFO(get_logger(), "Successfully set detect_color to %s!", param.as_string().c_str());
        });
  }
}
```

# camera Pkg

- 主要功能：作为相机驱动，根据相机序列号读相机和配置文件、采集图像，传给其他包
- Topic name：`/camera/front/capture`、`/camera/front/camera_info`
# recognition Pkg

- 主要功能：识别装甲板，对其进行 pnp 解算，发布其在相机系下的位姿
- topic name：`/autoaim/armors`

1. Opencv pnp 解算出来的的旋转矩阵和平移向量是在 opencv 相机系定义下的，与 ros2 相机系定义不同，具体差别见[坐标系定义](https://heurm.yuque.com/pwxb48/yy4glk/bslnoq1kqgctcn8u)，解算后要注意转换；

```cpp
  for (const auto & armor : *armors_)
  {
    const cv::Mat OpenCV_2_REP103 = (cv::Mat_<double>(3,3)
        << 0,0,1,
           -1,0,0,
           0,-1,0);
    const cv::Mat t_vec_REP103 = OpenCV_2_REP103 * armor.t_vec;
    const cv::Mat r_vec_REP103 = OpenCV_2_REP103 * armor.r_vec;
    autoaim_sys_interfaces::msg::Armor armor_msg;
    armor_msg.label            = armor.armor_label;
    armor_msg.size             = ArmorSize::SMALL == armor.armor_size ? "Small" : "Big";
    armor_msg.pose.position.x  = t_vec_REP103.at<double>(0) / 1.0e3;
    armor_msg.pose.position.y  = t_vec_REP103.at<double>(1) / 1.0e3;
    armor_msg.pose.position.z  = t_vec_REP103.at<double>(2) / 1.0e3;
  
    cv::Mat r_mat;
    cv::Rodrigues(r_vec_REP103, r_mat);
    tf2::Matrix3x3 tf2_rot_mat(
      r_mat.at<double>(0, 0), r_mat.at<double>(0, 1), r_mat.at<double>(0, 2),
      r_mat.at<double>(1, 0), r_mat.at<double>(1, 1), r_mat.at<double>(1, 2),
      r_mat.at<double>(2, 0), r_mat.at<double>(2, 1), r_mat.at<double>(2, 2));
  
    tf2::Quaternion tf2_quaternion;
    tf2_rot_mat.getRotation(tf2_quaternion);
    armor_msg.pose.orientation = tf2::toMsg(tf2_quaternion);
    armors_msg.armors.emplace_back(armor_msg);
  }
  pub_armors_->publish(armors_msg);
```

2. 判断灯条颜色颜色逻辑：在给定 ROI 内使用灯条矩形内的红蓝通道像素均值差判断颜色，若红色均值大于蓝色则为红色灯条；反之为蓝色灯条

```cpp
RecognitionNode::LightColor
RecognitionNode::judgeColor(
  const ImageChannels   & channel_imgs,
  const cv::RotatedRect & lightbar)
{
  // 取灯条旋转矩形周围的最小包围矩形作为ROI, 减少计算量
  cv::Rect2f roi = lightbar.boundingRect2f();
  // 限制ROI在图像范围内
  makeRectSafe(roi, cv::Size(channel_imgs[0].cols, channel_imgs[0].rows));
  // 截取灯条
  const cv::Mat mark = cv::Mat::zeros(channel_imgs[0](roi).size(), CV_8UC1);
  cv::Point2f rRectPoint[4];
  lightbar.points(rRectPoint);
  cv::line(mark, rRectPoint[0] - roi.tl(), rRectPoint[1] - roi.tl(), cv::Scalar(255));
  cv::line(mark, rRectPoint[1] - roi.tl(), rRectPoint[2] - roi.tl(), cv::Scalar(255));
  cv::line(mark, rRectPoint[2] - roi.tl(), rRectPoint[3] - roi.tl(), cv::Scalar(255));
  cv::line(mark, rRectPoint[3] - roi.tl(), rRectPoint[0] - roi.tl(), cv::Scalar(255));
  cv::floodFill(mark, lightbar.center - roi.tl(), 255, nullptr);  // 255像素填充旋转矩形
  
  // 使用灯条矩形内的红蓝通道像素均值差判断颜色
  const cv::Scalar scalar_red  = cv::mean(channel_imgs[2](roi), mark);
  const cv::Scalar scalar_blue = cv::mean(channel_imgs[0](roi), mark);

  return (scalar_red[0] > scalar_blue[0]) ? LightColor::RED : LightColor::BLUE;
}
```

# tracker Pkg

## tracker

- 主要功能：根据机器人 ID 构建跟踪器，构建卡尔曼滤波器进行滤波
- Topic name：`/tracker/units`

1. 利用 tf message filter 过滤目标坐标系为 `odom_frame` 的 tf 转换，若找不到相应的 tf 变换可适当延长 tf message filter 的队列长度；
2. 目前所使用的模型为 CV model（即匀速模型）

## kalman filter

# fire-control Pkg

- 主要功能：射击评估、弹道解算
- topic name：`/tracker/units`

1. 根据机器人 id 锁定目标
2. 目前使用 whx 的老弹道模型，需要二次预测，二次预测的本质是对 `pitch` 二次迭代，并非对预测量二次迭代，同时我们需要根据实测结果手动给予 `yaw` `pitch` 补偿，详细参考[弹道模型](https://heurm.yuque.com/pwxb48/yy4glk/ii7qms4ngzoegphz)
```cpp
  const double dt = (this->now() - aim_unit.header.stamp).seconds();
  auto s          = aim_unit.armor.target_state;
  s.x += s.dx * dt;
  s.y += s.dy * dt;
  s.z += s.dz * dt;
  
  double bullet_fly_time{};
  double bullet_speed = this->get_parameter("bullet_speed").as_double();
  double pitch        = bullet_solve_->bulletCompensation(
    std::sqrt(s.x * s.x + s.y * s.y), s.z, 0, bullet_speed, bullet_fly_time);
  
  /* 二次预测 */
  const auto pre_x = s.x + s.dx * bullet_fly_time;
  const auto pre_y = s.y + s.dy * bullet_fly_time;
  const auto pre_z = s.z + s.dz * bullet_fly_time;
  pitch            = bullet_solve_->bulletCompensation(
    std::sqrt(pre_x * pre_x + pre_y * pre_y), pre_z, 0, bullet_speed, bullet_fly_time);
  pitch += deg2rad(0.7);
  double yaw = std::atan2(pre_y, pre_x) + deg2rad(0.2);
```

3. 要给解算出的 yaw pitch 进行限位，防止云台猛烈甩头

```cpp
  if (M_PI < yaw) yaw -= 2 * M_PI;
  if (-M_PI > yaw) yaw += 2 * M_PI;
  
  if (M_PI < pitch) pitch -= 2 * M_PI;
  if (-M_PI > pitch) pitch += 2 * M_PI;
```

4. 射击评估逻辑：选取距离枪管轴线距离最近的目标，再判断云台与其的直线距离，同时还需要判断目前云台 `yaw` `pitch` 和预期转向 `yaw` `pitch` 的相对位置，评估是否可以射击

```cpp
double FireControlNode::getGimbalDist(const autoaim_sys_interfaces::msg::Unit & unit)
{
  geometry_msgs::msg::PoseStamped odom_frame_pose;
  geometry_msgs::msg::PoseStamped gimbal_frame_pose;
  gimbal_frame_pose.header.stamp    = unit.header.stamp;
  gimbal_frame_pose.header.frame_id = "";
  odom_frame_pose.pose.position.x   = unit.armor.target_state.x;
  odom_frame_pose.pose.position.y   = unit.armor.target_state.y;
  odom_frame_pose.pose.position.z   = unit.armor.target_state.z;
  odom_frame_pose.header.stamp      = unit.header.stamp;
  odom_frame_pose.header.frame_id   = "odom_frame";
  
  Eigen::Matrix3d camera2odom_rotation;
  try
  {
    tf2_buffer_->transform(odom_frame_pose, gimbal_frame_pose, "gimbal_frame");
  } catch (const tf2::TransformException & ex)
  {
    RCLCPP_WARN(this->get_logger(), "Failure: %s.", ex.what());
  } catch (...)
  {
    RCLCPP_ERROR(this->get_logger(), "TF2 Transform Error.");
  }

  const auto & p    = gimbal_frame_pose.pose.position;
  const double dist = std::sqrt(p.y * p.y + p.z * p.z);
  RCLCPP_INFO(this->get_logger(), "Armor: %s, Dist: %lf", unit.armor.label.c_str(), dist);
  return dist;
}
```

```cpp
bool FireControlNode::isAllowShoot(double armor_distance,
                                   const EulerAngle & shoot_euler_angle,
                                   ArmorSize armor_size)
{
  const double pitch_length = (125.0 / 2.0) / 1e3;
  const double yaw_length   = (armor_size == ArmorSize::SMALL ? 135.0 / 2.0 : 225.0 / 2.0) / 1e3;
  const double pitch_angle  = std::atan2(pitch_length, armor_distance);
  const double yaw_angle    = std::atan2(yaw_length, armor_distance);
  
  auto [gimbal_roll, gimbal_pitch, gimbal_yaw] = getGimbalEulerAngle();
  
  if (std::abs(shoot_euler_angle.pitch - gimbal_pitch) < pitch_angle &&
      std::abs(shoot_euler_angle.yaw - gimbal_yaw) < yaw_angle)
  {
    return true;
  }
  return false;
}
```