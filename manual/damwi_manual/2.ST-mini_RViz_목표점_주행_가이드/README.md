## rviz에서 waypoint 찍고 이동하는 법

### 1. PC에서 노머신으로 원격 접속하는 법

camtic,Ubuntu 20.04.6 LTS 접속

접속창이 보이지 않을 경우 젯슨의 노머신 재부팅 필요

```bash
# 내 PC (cmd 터미널)
ping 192.168.1.220            # ping이 오지 않을 경우 네트워크 문제
ssh camtic@192.168.1.220      # 젯슨 원격접속

#camtic_dev 안
exit                          #노머신이 설치되어있는 로봇 PC로 이동

# 로봇 PC
sudo /usr/NX/bin/nxserver --restart
```
노머신을 재부팅하고 ping이 오면 연결 가능

### 2. rviz 맵 확인

```bash
# camtic_dev 안 (터미널 1)
ros2 launch camtic_bringup camtic_bringup.launch.py    # 바퀴 + 센서 + 자율주행 + 위치 복원

```
```bash
# camtic_dev 안 (터미널 2)
rviz2
```
Rviz 설정항목
- Add - by topic - /map - map
  
  map - topic - Durability Policy - Transient Local 변경 (맵핑 파일 불러오기)
- Add - by topic - /scan - LaserScan (라이다센스 값 불러오기)
- Add - By display type - TF (AMR 위치 확인)
  
  TF - Frames - base_link, map, odom 만 체크

### 3. 현재위치 확인 및 이동
**현재위치 변경**

- 맵과 라이다센서 값(보라색 선)이 맞지 않을 경우 Rviz 위쪽 상단의 2D Pose Estimate로 변경

**이동**

- Rviz 위쪽 상단의 2D Goal Pose로 이동

**현재좌표 확인**
```bash
# camtic_dev 안
ros2 topic echo --once /amcl_pose
```
