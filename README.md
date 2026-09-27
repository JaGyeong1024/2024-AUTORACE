# 2024 AutoRace

2024 스케일카 자율주행 경진대회 출전 코드.  
카메라와 2D LiDAR로 미션 구간을 인식하고, 각 미션 노드가 우선순위가 다른 `ackermann_cmd_mux` 입력으로 주행 명령을 보냅니다.

## Team

<div align="center">
<table align="center">
<tr>
<td align="center" width="150">
  <a href="https://github.com/a1038562"><img src="https://github.com/a1038562.png?size=200" width="100px;" alt=""/><br /><sub><b>이지원</b></sub></a>
</td>
<td align="center" width="150">
  <a href="https://github.com/tjsdn3065"><img src="https://github.com/tjsdn3065.png?size=200" width="100px;" alt=""/><br /><sub><b>김선우</b></sub></a>
</td>
<td align="center" width="150">
  <a href="https://github.com/YSH-research"><img src="https://github.com/YSH-research.png?size=200" width="100px;" alt=""/><br /><sub><b>윤수한</b></sub></a>
</td>
<td align="center" width="150">
  <a href="https://github.com/ttaehyun"><img src="https://github.com/ttaehyun.png?size=200" width="100px;" alt=""/><br /><sub><b>유태현</b></sub></a>
</td>
<td align="center" width="150">
  <a href="https://github.com/JaGyeong1024"><img src="https://avatars.githubusercontent.com/u/92356313?s=400&u=9df94c6f0e773e86773cb4fcc379f1204a7dcff7&v=4" width="100px;" alt=""/><br /><sub><b>구자경</b></sub></a>
</td>
</tr>
</table>
</div>

## Missions

| Mission | Sensor | Behavior | Node |
|---|---|---|---|
| 차선 주행 | Camera | 버드아이뷰 변환 후 슬라이딩 윈도우로 차선 추종 | `lane_main.py` |
| 차단기 | LiDAR | 차단기를 감지하면 정지, 올라가면 출발. 벽 등 다른 장애물에는 반응하지 않음 | `barricade_detector.py` |
| 라바콘 | LiDAR | `obstacle_detector` 장애물로 좌우 콘 사이 중앙 경로 추종 | `rubber_cone.py` |
| 터널 | LiDAR | 좌우 벽 거리 PID 주행 | `tunnel.py` |
| 빨간 노면 | Camera | 빨간 노면 구간에서 감속 | `red_road_node` |
| 횡단보도 | Camera | 정지선 검출 시 5초 정지 후 출발 | `crosswalk.py` |
| A/B 갈림길 | Camera (AR 마커) | AR 마커 ID로 좌/우 진행 방향 결정 | `choice_AB.py` |
| 회전교차로 | LiDAR | 회전 차량 클러스터의 이동 방향으로 주행 방향 플래그(`/direction_flag`) 결정 | `round_about_node`, `arrow_cluster_node` |
| 주차 | Camera | 주차 구역 검출 후 전진·후진 시퀀스 | `parking_rect.py` |

## Command Priority

모든 미션 노드는 `racecar`의 `high_level/ackermann_cmd_mux`에 입력을 보냅니다. 여러 입력이 동시에 들어오면 우선순위가 높은 명령이 선택됩니다 (`catkin_ws/src/racecar/racecar/config/racecar-v2/high_level_mux.yaml`).

| Input | Priority | Node |
|---|---|---|
| `input/nav_0` | 20 | `parking_rect.py` |
| `input/nav_1` | 18 | `rubber_cone.py` |
| `input/nav_2` | 15 | `red_road_node` |
| `input/nav_3` | 12 | `barricade_detector.py` |
| `input/nav_4` | 10 | `crosswalk.py` |
| `input/nav_5` | 8 | `tunnel.py` |
| `input/nav_6` | 5 | `lane_main.py` |

## Layout

| Path | Contents |
|---|---|
| `catkin_ws/` | 차량 기본 스택: 센서 드라이버, VESC, ackermann mux, 텔레옵 |
| `webot_ws/` | 대회 미션 패키지 |

## Packages

`webot_ws/src`:

| Package | Role |
|---|---|
| `lane_detection` | 차선 주행 및 카메라·LiDAR 미션 노드, 전체 실행 launch (`start.launch`) |
| `barricade_detector` | 차단기 감지 |
| `red_road` | 빨간 노면 감지 |
| `roundAboutNav` | 회전교차로 LiDAR 클러스터링 및 진입 판단 |
| `image_undistort_pkg` | 카메라 왜곡 보정 (`/usb_cam/image_raw` → `/usb_cam/image_raw/calib`) |
| `ar_track_alvar` | AR 마커 인식 |

`catkin_ws/src`:

| Package | Role |
|---|---|
| `racecar` | 차량 bringup, `ackermann_cmd_mux` |
| `vesc` | VESC 모터 드라이버 |
| `rplidar_ros` | RPLiDAR 드라이버 |
| `usb_cam` | USB 카메라 드라이버 |
| `razor_imu_9dof` | IMU 드라이버 |
| `obstacle_detector` | LiDAR 장애물 추출 (`/raw_obstacles`) |
| `fiducials`, `image_pipeline` | 마커 인식, 카메라 캘리브레이션 |
| `wego` | 텔레옵, 센서 확인용 rviz launch |

## Build

Ubuntu 20.04 + ROS Noetic.

```bash
cd catkin_ws && catkin_make && source devel/setup.bash
cd ../webot_ws && catkin_make && source devel/setup.bash
```

## Run

```bash
roslaunch wego teleop.launch              # 차량 bringup (VESC, RPLiDAR, USB 카메라, mux, 조이스틱)
roslaunch lane_detection start.launch     # 전체 미션 노드
```
