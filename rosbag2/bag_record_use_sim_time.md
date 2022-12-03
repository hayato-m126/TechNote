# ros2 bag record で sim_time を反映する

## 概要

[ROS2 humble の11月のリリース](https://discourse.ros.org/t/new-packages-and-patch-release-for-humble-hawksbill-2022-11-23/28495)でrosbag2にアップデートが入り、record時にsim_timeを反映するオプションが使用可能になりました。

この更新内容について紹介します。

### 想定環境

- Ubuntu 22.04
- ROS2 humble

### 対象読者

- ROS2で開発している方

### 関連リンク

1. https://github.com/ros2/rosbag2/issues/764
2. https://github.com/ros2/rosbag2/pull/994

## 導入方法

apt で配布されているので更新します。

```shell
sudo apt update
sudo apt upgrade
```

インストール後にバージョンが`0.15.3-1`になっていればOKです。

```shell
❯ apt-cache policy ros-humble-rosbag2
ros-humble-rosbag2:
  インストールされているバージョン: 0.15.3-1jammy.20221109.192253
```

## 使い方

sim_timeで記録するオプション`--use-sim-time`が追加されているので、これを指定するだけです。

```shell
# ネットワークに流ているすべてのトピックをシミュレーション時間を使って記録する
ros2 bag record -a --use-sim-time
```

## アップデートにより何が変わるか

概要の関連リンクの1でも述べられている通り、ROS2では従来レコードしたメッセージのタイムスタンプには、システムの時間が適用されていました。

実機で取得したbagを使ってシミュレーションを実行して、シミュレーション時に出力されるtopicをbagにレコードすると、タイムスタンプはシミュレーションを実行した時刻になってしまいます。

シミュレーションはbag playのrateを指定することで、倍速でシミュレーションを回したりすることができますが、その結果を取ったbagの時間まで伸び縮みしてしまいます。
なので、topicが想定通りの周期[Hz]で記録されたか調べるとき等に、どのrateで取ったかがわかってないと判断に困ったりします。

```shell
# 2倍速で再生
ros2 bag play input_bag --rate 2.0 --clock 200

# 別ターミナルで記録。input_bagのdurationの半分になる。
ros2 bag record -a
```

use-sim-timeが反映されると、rateを変更してもこういった問題が起こらなくなります。

ROS1だとuse-sim-timeがtrueならsim-timeで記録されていたので、このアップデートでROS1のときと同等の動作にすることができます。

## 公開されているbagを使って試してみる

[ここに置いてあるサンプルデータ](https://tier4.github.io/driving_log_replayer/quick_start/setup/)
を使って実際にコマンドを叩いて確かめてみる。

### 再生するbagの内容

再生するbagの内容を示す。
autowareの独自定義型を含むbagで、2022年4月5日取られた68.972sのbag

```shell
❯ ros2 bag info input_bag/

Files:             input_bag_0.db3
Bag size:          2.6 GiB
Storage id:        sqlite3
Duration:          68.972s
Start:             Apr  5 2022 15:07:31.594 (1649138851.594)
End:               Apr  5 2022 15:08:40.566 (1649138920.566)
Messages:          21590
Topic information: Topic: /localization/kinematic_state | Type: nav_msgs/msg/Odometry | Count: 3440 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/fix_velocity | Type: geometry_msgs/msg/TwistWithCovarianceStamped | Count: 345 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/nav_sat_fix | Type: sensor_msgs/msg/NavSatFix | Count: 343 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/navpvt | Type: ublox_msgs/msg/NavPVT | Count: 344 | Serialization Format: cdr
                   Topic: /sensing/imu/tamagawa/imu_raw | Type: sensor_msgs/msg/Imu | Count: 1936 | Serialization Format: cdr
                   Topic: /sensing/lidar/concatenated/pointcloud | Type: sensor_msgs/msg/PointCloud2 | Count: 685 | Serialization Format: cdr
                   Topic: /sensing/lidar/left/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 688 | Serialization Format: cdr
                   Topic: /sensing/lidar/right/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 687 | Serialization Format: cdr
                   Topic: /sensing/lidar/top/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 685 | Serialization Format: cdr
                   Topic: /tf | Type: tf2_msgs/msg/TFMessage | Count: 4484 | Serialization Format: cdr
                   Topic: /tf_static | Type: tf2_msgs/msg/TFMessage | Count: 2 | Serialization Format: cdr
                   Topic: /vehicle/status/control_mode | Type: autoware_auto_vehicle_msgs/msg/ControlModeReport | Count: 1985 | Serialization Format: cdr
                   Topic: /vehicle/status/steering_status | Type: autoware_auto_vehicle_msgs/msg/SteeringReport | Count: 1990 | Serialization Format: cdr
                   Topic: /vehicle/status/turn_indicators_status | Type: autoware_auto_vehicle_msgs/msg/TurnIndicatorsReport | Count: 1988 | Serialization Format: cdr
                   Topic: /vehicle/status/velocity_status | Type: autoware_auto_vehicle_msgs/msg/VelocityReport | Count: 1988 | Serialization Format: cdr
```

### sim_timeをつけずに倍速再生して記録する

#### 再生側のターミナル

```shell
# bagにautowareに独自定義型が含まれているのでautowareのworkspaceを事前にsourceしておく
# cd autoware
# source install/setup.bash
~/sample_dataset
❯ ros2 bag play input_bag --rate 2.0 --clock
```

#### 記録側

```shell
# bagにautowareに独自定義型が含まれているのでautowareのworkspaceを事前にsourceしておく
# cd autoware
# source install/setup.bash
ros2 bag record -a
```

#### 記録されたbagをinfoした結果

レコードしたbagの結果。手動でやっているので少し取りこぼしがありますが、topic数は大体同じでdurationが半分のbagができました。

```shell
❯ ros2 bag info rosbag2_2022_11_30-04_03_30/

Files:             rosbag2_2022_11_30-04_03_30_0.db3
Bag size:          2.5 GiB
Storage id:        sqlite3
Duration:          32.730s
Start:             Nov 30 2022 04:03:30.663 (1669748610.663)
End:               Nov 30 2022 04:04:03.393 (1669748643.393)
Messages:          21856
Topic information: Topic: /clock | Type: rosgraph_msgs/msg/Clock | Count: 1310 | Serialization Format: cdr
                   Topic: /localization/kinematic_state | Type: nav_msgs/msg/Odometry | Count: 3275 | Serialization Format: cdr
                   Topic: /rosout | Type: rcl_interfaces/msg/Log | Count: 10 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/fix_velocity | Type: geometry_msgs/msg/TwistWithCovarianceStamped | Count: 327 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/nav_sat_fix | Type: sensor_msgs/msg/NavSatFix | Count: 327 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/navpvt | Type: ublox_msgs/msg/NavPVT | Count: 327 | Serialization Format: cdr
                   Topic: /sensing/imu/tamagawa/imu_raw | Type: sensor_msgs/msg/Imu | Count: 1845 | Serialization Format: cdr
                   Topic: /sensing/lidar/concatenated/pointcloud | Type: sensor_msgs/msg/PointCloud2 | Count: 653 | Serialization Format: cdr
                   Topic: /sensing/lidar/left/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 654 | Serialization Format: cdr
                   Topic: /sensing/lidar/right/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 655 | Serialization Format: cdr
                   Topic: /sensing/lidar/top/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 654 | Serialization Format: cdr
                   Topic: /tf | Type: tf2_msgs/msg/TFMessage | Count: 4259 | Serialization Format: cdr
                   Topic: /tf_static | Type: tf2_msgs/msg/TFMessage | Count: 2 | Serialization Format: cdr
                   Topic: /vehicle/status/control_mode | Type: autoware_auto_vehicle_msgs/msg/ControlModeReport | Count: 1889 | Serialization Format: cdr
                   Topic: /vehicle/status/steering_status | Type: autoware_auto_vehicle_msgs/msg/SteeringReport | Count: 1889 | Serialization Format: cdr
                   Topic: /vehicle/status/turn_indicators_status | Type: autoware_auto_vehicle_msgs/msg/TurnIndicatorsReport | Count: 1890 | Serialization Format: cdr
                   Topic: /vehicle/status/velocity_status | Type: autoware_auto_vehicle_msgs/msg/VelocityReport | Count: 1890 | Serialization Format: cdr
```

### sim_timeをつけて倍速再生して記録する

#### 再生側のターミナル

```shell
# bagにautowareに独自定義型が含まれているのでautowareのworkspaceを事前にsourceしておく
# cd autoware
# source install/setup.bash
~/sample_dataset
❯ ros2 bag play input_bag --rate 2.0 --clock
```

#### 記録側

```shell
# bagにautowareに独自定義型が含まれているのでautowareのworkspaceを事前にsourceしておく
# cd autoware
# source install/setup.bash
ros2 bag record -a --use-sim-time
```

#### 記録されたbagをinfoした結果

レコードしたbagの結果。
bag playを実行してからrecordしているので、最初2秒くらいは取りこぼしていますが、入力と同じ時刻になっています。

きちんとsim-timeが反映されていますね。

```shell
❯ ros2 bag info rosbag2_2022_11_30-04_25_00/

Files:             rosbag2_2022_11_30-04_25_00_0.db3
Bag size:          2.5 GiB
Storage id:        sqlite3
Duration:          66.700s
Start:             Apr  5 2022 15:07:33.856 (1649138853.856)
End:               Apr  5 2022 15:08:40.557 (1649138920.557)
Messages:          22266
Topic information: Topic: /clock | Type: rosgraph_msgs/msg/Clock | Count: 1334 | Serialization Format: cdr
                   Topic: /localization/kinematic_state | Type: nav_msgs/msg/Odometry | Count: 3336 | Serialization Format: cdr
                   Topic: /rosout | Type: rcl_interfaces/msg/Log | Count: 10 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/fix_velocity | Type: geometry_msgs/msg/TwistWithCovarianceStamped | Count: 333 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/nav_sat_fix | Type: sensor_msgs/msg/NavSatFix | Count: 333 | Serialization Format: cdr
                   Topic: /sensing/gnss/ublox/navpvt | Type: ublox_msgs/msg/NavPVT | Count: 334 | Serialization Format: cdr
                   Topic: /sensing/imu/tamagawa/imu_raw | Type: sensor_msgs/msg/Imu | Count: 1880 | Serialization Format: cdr
                   Topic: /sensing/lidar/concatenated/pointcloud | Type: sensor_msgs/msg/PointCloud2 | Count: 666 | Serialization Format: cdr
                   Topic: /sensing/lidar/left/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 666 | Serialization Format: cdr
                   Topic: /sensing/lidar/right/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 667 | Serialization Format: cdr
                   Topic: /sensing/lidar/top/velodyne_packets | Type: velodyne_msgs/msg/VelodyneScan | Count: 666 | Serialization Format: cdr
                   Topic: /tf | Type: tf2_msgs/msg/TFMessage | Count: 4339 | Serialization Format: cdr
                   Topic: /tf_static | Type: tf2_msgs/msg/TFMessage | Count: 2 | Serialization Format: cdr
                   Topic: /vehicle/status/control_mode | Type: autoware_auto_vehicle_msgs/msg/ControlModeReport | Count: 1925 | Serialization Format: cdr
                   Topic: /vehicle/status/steering_status | Type: autoware_auto_vehicle_msgs/msg/SteeringReport | Count: 1925 | Serialization Format: cdr
                   Topic: /vehicle/status/turn_indicators_status | Type: autoware_auto_vehicle_msgs/msg/TurnIndicatorsReport | Count: 1925 | Serialization Format: cdr
                   Topic: /vehicle/status/velocity_status | Type: autoware_auto_vehicle_msgs/msg/VelocityReport | Count: 1925 | Serialization Format: cdr
```

## おまけ

ros2 bag reindexのコマンドの使い方が少し変わっていました。`-s`のあとにstorageを書くようになっていました。

```shell
❯ ros2 bag reindex -h
usage: ros2 bag reindex [-h] [-s {my_test_plugin,my_read_only_test_plugin,sqlite3,mcap}] bag_path

Reconstruct metadata file for a bag

positional arguments:
  bag_path              Bag to open

options:
  -h, --help            show this help message and exit
  -s {my_test_plugin,my_read_only_test_plugin,sqlite3,mcap}, --storage {my_test_plugin,my_read_only_test_plugin,sqlite3,mcap}
                        Storage implementation of bag. By default attempts to detect automatically - use this argument to override.
```
