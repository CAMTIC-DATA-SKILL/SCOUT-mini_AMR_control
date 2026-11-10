## 사용자 매뉴얼 파일
camtic_ST-mini_사용매뉴얼.md파일은 토큰 절약을 위해 pdf를 변환한 것으로 잘못된 부분이 있을 수 있습니다. 
**매뉴얼은 pdf를 기본으로 확인**해야합니다. 

## 간단한 연결 테스트 하는 법

### 접속·상태 확인

```bash
# 내 PC
ssh camtic@192.168.0.211        # 와이파이 접속 (랜선: 192.168.1.220)
```

```bash
# camtic_dev 안
ip -br link show can0           # UP이면 CAN 통로가 열린 것 (실제 통신은 아래 토픽으로 확인)
pgrep -x scout_base_node        # 숫자가 나오면 바퀴 프로그램이 이미 실행 중
pkill -x scout_base_node        # 위에서 숫자가 나온 경우에만 실행
```

```bash
# 로봇 PC  (camtic_dev에서 exit 한 번 후)
docker ps                       # camtic_nav, camtic_zenoh가 없어야 개발 모드 사용 가능
```

### 실행

```bash
# camtic_dev 안
ros2 launch camtic_bringup camtic_bringup.launch.py        # 바퀴 + 센서 + 자율주행 (개발 모드)
ros2 launch camtic_bringup slam_bringup.launch.py          # 바퀴 + 센서 + SLAM (위와 동시 실행 금지)
ros2 launch realsense2_camera rs_launch.py pointcloud.enable:=true   # 카메라만
ros2 run teleop_twist_keyboard teleop_twist_keyboard       # 키보드 조종 (임시 설치 패키지)
```

### 데이터 확인

```bash
# camtic_dev 안
ros2 topic hz /wheel_odom                  # 약 50Hz면 바퀴 정상
ros2 topic hz /scan                        # 약 10Hz면 라이다 정상
ros2 topic echo --once /scout_status       # error_code: 0, 배터리 전압
ros2 node list                             # 실행 중인 노드
ros2 topic list                            # 토픽 목록
ros2 node info /<노드 이름>                 # 노드가 올리고 받는 토픽
ros2 topic info /cmd_vel -v                # 토픽의 발행자·구독자
```

### 지도 저장 (매핑을 끝내고 로봇을 세운 뒤)

```bash
# camtic_dev 안
mkdir -p /home/camtic/my_ws/maps
ros2 run nav2_map_server map_saver_cli -f /home/camtic/my_ws/maps/test_map --ros-args -p save_map_timeout:=10.0
```

## 젯슨(로봇 PC) 끄는 법

**1) 지도 저장을 먼저 끝냅니다.** SLAM을 끄면 만든 지도가 사라집니다. 저장한 파일은 `/home/camtic/my_ws/maps/` 안에 `test_map.pgm`, `test_map.yaml`이 생겼는지 확인하세요.

```bash
# camtic_dev 안
ls -l /home/camtic/my_ws/maps/
```

**2) 실행 중인 프로그램을 모두 끕니다.** teleop은 `k`로 정지한 뒤 Ctrl + C, 그다음 `slam_bringup`이나 `camtic_bringup`을 실행한 터미널에서 Ctrl + C를 누릅니다.

**3) camtic_dev에서 나와 로봇 PC에서 전원을 끕니다.**

```bash
# camtic_dev 안
exit                      # 프롬프트가 camtic@camtic:~$ 로 바뀌면 로봇 PC
```

```bash
# 로봇 PC
sudo poweroff
```

명령을 입력하면 SSH 접속이 끊기는 것이 정상입니다. 전원 버튼을 길게 눌러 강제로 끄지 마세요. 저장 중인 파일이 손상될 수 있습니다. Jetson이 완전히 꺼질 때까지 잠시 기다린 뒤에 베이스 전원을 끄세요. 베이스 전원을 끄는 정확한 절차는 매뉴얼에 없으니, 로봇에 달린 전원 스위치 안내를 확인하세요.

다음에 켤 때는 전원을 켜면 `camtic_dev`와 CAN 자동 복구가 자동으로 켜집니다. 바퀴 프로그램은 자동으로 실행되지 않으니 위의 `camtic_bringup`부터 다시 실행하면 됩니다. 이번에 apt로 임시 설치한 `teleop_twist_keyboard`는 컨테이너를 새로 만들지 않는 한 남아 있습니다.
