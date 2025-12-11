# Drawing-a-picture-with-ROBOTIS-OMX-
Operate ROBOTIS-OMX and perform a drawing task


```bash
https://ai.robotis.com/omx/setup_guide_physical_ai_tools.html
```

### container.sh 내부에 들어가기 

```bash
#(open_manipulater/docker 에서 실행)
./container.sh enter 
```

### 실행 파일 포트 설정

```bash
# 원하는 port 확인 (leader의 port number를 확인하고 싶다면 leader만 꽂고 실행)
ls -al /dev/serial/by-id/ 
```
<img width="1188" height="117" alt="image" src="https://github.com/user-attachments/assets/f39268eb-19c3-43e3-966d-484119c04de0" />

<br/>

### 파일 수정 (리더 ,팔로워) 

#leader
```bash
sudo nano ~/ros2_ws/src/open_manipulator/open_manipulator_bringup/launch/omx_l_leader_ai.launch.py
```
#follower
```bash
sudo nano ~/ros2_ws/src/open_manipulator/open_manipulator_bringup/launch/omx_f_follower_ai.launch.py
```

<img width="1188" height="628" alt="image" src="https://github.com/user-attachments/assets/adf5ce26-8095-4f16-b8f5-1fd37042bdce" />


### Phyiscal AI Tools Docker 컨테이너 설정

```bash
git clone --recurse-submodules https://github.com/ROBOTIS-GIT/physical_ai_tools.git

cd physical_ai_tools/docker && ./container.sh start
 
./container.sh enter
```
### 실행

```bash
ros2 launch open_manipulator_bringup omx_ai.launch.py
```

---

### 기존 오류 (Teleoperate)

ros2 launch ~ 실행시 ID:11 motor에서 error 발생
11번 motor는 omx_f_follower에 위치

<img width="1340" height="285" alt="image" src="https://github.com/user-attachments/assets/7aaac09a-201d-40a1-a079-fa42d870c0e4" />

try
- port number 재확인 -> 이상 x
- launch/omx_l_leader_ai.launch.py , launch/omx_f_follower_ai.launch.py 에 정확히 기입하였는지 확인 -> 이상 x

따라서 hardware 쪽 문제라고 판단하여 omx_f_follower의 조립 재확인 및 인가 전원 확인
- follower Operating Voltage : 12V  but 현재 5V로 leader와 동일 전원 인가중이었음 -> 5V 전원 해제
- 다른 전원 인가시 불꽃이 튀고 operate 전에 바로 끊기는 현상 발생 -> 과전압으로 판단되어 정확히 12V 공급 어댑터 연결 -> 정상적으로 작동 ✅ 

<img width="1879" height="751" alt="image" src="https://github.com/user-attachments/assets/872da6de-f733-4c54-a07c-2aebb91aa296" />

---

toward...

```bash
- follower gripper 부분 재설정. 조립 잘못됨
- teleoperate 진행해보고 MoveIt 2 들어가기
```


---


11/29

bringup
```bash
ros2 launch open_manipulator_bringup omx_f.launch.py
```

이대로만 돌리면 에러 발생([ID:011] 끊김). omx_f.launch.py의 Port number 맞춰줘야함 (omx_f_follower_ai.launch.py 번호와 동일하게)
<img width="1233" height="140" alt="image" src="https://github.com/user-attachments/assets/2aa924eb-725c-4ee1-9ba3-34e1974048c1" />
<img width="1432" height="851" alt="image" src="https://github.com/user-attachments/assets/ba7d2adf-2eab-42a9-9acd-706b85c60b1d" />
정상적으로 작동. omx_f_follower_ai.launch.py과 다른 변수들도 맞춰야 하는지는 모르겠음.  + 그냥 omx_f_follower_ai.launch.py로 돌려도 되긴함! 

<br>

MoveIT 

```bash
 ros2 launch open_manipulator_moveit_config omx_f_moveit.launch.py
```
바로 실행하면 다음과 같은 에러 발생
<img width="1701" height="363" alt="image" src="https://github.com/user-attachments/assets/6be33008-7d46-4df6-b00b-906b9cb6eda3" />


오류 원인 : omx_f의 urdf.py 경로가 moveit에 없음 -> 로봇 구현이 안됨

<br>

해결 방안 (1) : omx_f.urdf.xacro 파일 moveit_config로 복제
```bash
cd ~/Project/omx/open_manipulator/open_manipulator_moveit_config

mkdir -p config/omx_f

cp ../open_manipulator_description/urdf/omx_f/omx_f.urdf.xacro \
   config/omx_f/omx_f.urdf.xacro
```
해결 방안 (2)  moveit -> launch -> omx_f_moveit.launch.py 수정
```bash
moveit_config = (
        MoveItConfigsBuilder(
            robot_name='omx_f', package_name='open_manipulator_moveit_config')
        .robot_description(   # ⬅ 여기 변수 생성 (urdf.xarcro 경로 추가해주는 과정)
            str(Path('config') / 'omx_f' / 'omx_f.urdf.xacro'))
        .robot_description_semantic(
            str(Path('config') / 'omx_f' / 'omx_f.srdf'))
        .joint_limits(str(Path('config') / 'omx_f' / 'joint_limits.yaml'))
        .trajectory_execution(
            str(Path('config') / 'omx_f' / 'moveit_controllers.yaml'))
        .robot_description_kinematics(
            str(Path('config') / 'omx_f' / 'kinematics.yaml'))
        .to_moveit_configs()
    )
```
마지막으로 재빌드 해주면!
```bash
cd /root/ros2_ws
colcon build --packages-select open_manipulator_moveit_config
source install/setup.bash

ros2 launch open_manipulator_moveit_config omx_f_moveit.launch.py
```
<img width="417" height="359" alt="image" src="https://github.com/user-attachments/assets/f64c029f-0d81-4477-8133-6058a98751bd" />
<img width="1395" height="422" alt="image" src="https://github.com/user-attachments/assets/0005fe84-7e5e-455b-b2fa-ccd9413dbd2e" />


<br>

OpenMANIPULATOR GUI (bringup + MoveIt 실행상태에서)

```bash
ros2 launch open_manipulator_gui omx_f_gui.launch.py
```

현재 Joint space에서는 작동하는데, Task 적용이 안되는 에러 발생
<img width="1552" height="865" alt="image" src="https://github.com/user-attachments/assets/e82bed7c-544c-41f8-8687-da6608307220" />

<br>

키보드 텔레오퍼레이션

```bash
ros2 run open_manipulator_teleop omx_f_teleop
```

Joint Control
```bash
1 / q - Joint 1 
2 / w - Joint 2 
3 / e - Joint 3
4 / r - Joint 4
5 / t - Joint 5
```
Gripper Control
```bash
o - Open gripper
p - Close gripper
```

---

Toward...
- GUI 내에서 Task 수행 및 학습
- Gazebo나 다른 시뮬 같이 수행
=> 일단 학습 가능 단계까지 구현하기 !!!


12/12

다 실패. 전 오늘 쉽니다