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

### 기존 오류

Teleoperate

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
