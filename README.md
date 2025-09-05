# ROS-with-Docker-Swarm
# 1. Introduction

준비: 노트북과 라즈베리파이에 도커 엔진과 도커 컴포즈 플러그인을 설치하고, 필요한 파일을 디렉터리에 저장한다.
https://docs.docker.com/manuals/

목적: Docker Swarm을 이용하여 overlay 네트워크를 구성한 후 overlay 네트워크를 통해 토픽을 주고 받아 터틀봇을 조종하는 예제.
- 노트북: Manager Node
- 라즈베리파이: Worker Node

  필요한 파일(터틀봇 메뉴얼 참고)
  https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/
  - 라즈베리파이:
    - ~/docker_ws/docker_file/Dockerfile: 멘더에서 제공하는 이미지를 Dockerfile로 만들어놓은 것으로 지금은 Dockerfile로 이미지를 만들어서 사용한다. 터틀봇을 동작시키기 위한 패키지와 opencr 패키지가 포함된다.
  - 노트북:
    - ~/docker_ws/comp_file/compose.yaml: Docker Swarm에서 서비스를 배포해 overlay 네트워크를 만드는데 필요하다. ROS master, talker, listener 서비스가 포함된다.
  - ~/docker_ws/docker_file/Dockerfile: 노트북에서 터틀봇을 조종하기 위한 teleop 패키지와 기타 터틀봇 패키지가 들어있는 이미지의 Dockerfile이다

<시스템 구조>
<img width="915" height="514" alt="image" src="https://github.com/user-attachments/assets/d24393be-8e00-4553-a3f6-dda7be05ad51" />
Swarm을 통해 노트북과 라즈베리파이가 연결되고, 서비스가 배포되면 노트북과 라즈베리파이에 걸쳐서 overlay 네트워크가 형성된다. 
이 overlay 네트워크는 노트북과 라즈베리파이의 컨테이너들이 서로 통신을 할 수 있도록 한다.
 overlay 네트워크 아래에 있는 컨테이너에서 ROS가 실행되므로 ROS 네트워크는 overlay 네트워크에 구현된다.
 overlay 네트워크 아래에 있는 컨테이너에서 실행된 ROS 노드들은 이 overlay 네트워크를 통해 토픽을 주고 받을 수 있다.

 # 2. Docker Swarm 연결
#노트북
$ docker swarm init ⇒ 노트북을 Swarm Manager 노드로 형성한다. 해당 명령어를 실행하면 아래의 사진처럼 만들어진 Swarm에 Worker 노드를 추가하기 위한 명령어가 나오는데 해당 명령어를 복사하여 라즈베리파이에서 실행해야 한다.
<img width="952" height="154" alt="image" src="https://github.com/user-attachments/assets/fb6bd23c-0cb6-42de-a6f5-27221f7d8f60" />
#라즈베리파이
$ docker swarm join —token <TOKEN> <노트북-IP>:2377 ⇒ 라즈베리파이를 만들어진 Swarm의 Worker 노드로 형성한다. 성공하면 “This node joined a swarm as a worker”라는 출력이 나타난다.
#노트북
$ docker node ls ⇒ Swarm과 내부 노드들을 확인하는 명령어이다. 아래와 같은 사진처럼 출력된다.
$ docker node update —label-add role=notebook <노트북 호스트 이름>
$ docker node update —label-add role=raspberry <라즈베리파이 호스트 이름>
⇒ Swarm 내의 각 노드에 라벨을 부여하는 명령어이다. compose.yaml 파일에서 서비스가 배포되는 노드의 라벨과 일치해야 한다.
<img width="1032" height="77" alt="image" src="https://github.com/user-attachments/assets/0898c3c5-ede5-40b0-91d7-15ba37628244" />

# 3. Overlay Network 형성

#노트북
**$ cd ~/docker_ws/comp_file/**
**$ docker stack deploy —compose-file compose.yaml rosstack** ⇒ 도커 스택을 형성하여 컴포즈 파일에 정의된 노드로 서비스가 배포된다.
**$ docker stack ls** ⇒ 스택 형성을 확인하는 명령어이다.
**$ docker service ls** ⇒ 서비스 형성을 확인하는 명령어이다. REPLICAS가 모두 1/1인지 확인한다.
**$ docker network ls** ⇒ overlay 네트워크가 만들어졌는지 확인한다. 네트워크의 이름은 rosstack_rosnet이다.
**$ docker network inspect rosstack_rosnet** ⇒ overlay 네트워크 내부 컨테이너들이 잘 연결되었는지 확인한다.
<img width="728" height="285" alt="image" src="https://github.com/user-attachments/assets/2de4b0ef-5a25-4207-bb7d-1a7adb7b919b" />
#라즈베리파이
**$ docker network ls** ⇒ overlay 네트워크가 만들어졌는지 확인한다.
**$ docker network inspect rosstack_rosnet** ⇒ overlay 네트워크 내부 컨테이너들이 잘 연결되었는지 확인한다. 

#노트북(다른 터미널에서)
**$ docker container ps** ⇒ master와 listener 컨테이너가 작동하고 있는지 확인한다.
**$ docker attach <listener 컨테이너 이름>** ⇒ listener 컨테이너에 접속해서 토픽을 잘 받고 있는지 확인한다.
<img width="1222" height="96" alt="image" src="https://github.com/user-attachments/assets/bd30d5dc-5924-4fa0-ba94-fd45ec02b0ee" />

# 4. 이미지 생성

### 라즈베리파이에 터틀봇 이미지 생성

#라즈베리파이
**$ cd ~/docker_ws/docker_file/
$ docker build -t turtlebot3:noetic .**

### 노트북에 터틀봇 패키지 이미지 생성

#노트북
**$ cd ~/docker_ws/docker_file/
$ docker build -t tb3_pc:noetic .**

# 5. 라즈베리파이 터틀봇 컨테이너

### 라즈베리파이에서 터틀봇 컨테이너를 overlay 네트워크에 연결하여 생성

#라즈베리파이
**$ docker run —name tb3 —hostname tb3 —device=/dev/ttyACM0:/dev/ttyACM0 —device=/dev/ttyUSB0:/dev/ttyUSB0 —network rosstack_rosnet -d -it turtlebot3:noetic bash**
⇒ 컨테이너를 만들기 전에 라즈베리파이에서 USB 장치 연결을 확인한다. ($ sudo dmesg | grep tty)
**$ docker network inspect rosstack_rosnet** ⇒ 터틀봇 컨테이너가 overlay 네트워크에 연결되었는지 확인한다.
**$ docker attach tb3** ⇒ 컨테이너에 접속

### 라즈베리파이의 터틀봇 컨테이너에서 터틀봇 세팅

#라즈베리파이의 터틀봇 컨테이너
**$ cd ~/opencr_update/
$ ./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr** ⇒ opencr 세팅
<img width="2048" height="1112" alt="image" src="https://github.com/user-attachments/assets/a9109b70-2a71-4079-9b7e-1f95048fa786" />
$ cd ~/catkin_ws/
$ catkin_make
$ export LDS_MODEL=LDS-02
$ export ROS_MASTER_URI=http://<master-IP>:11311 ⇒ overlay 네트워크에 연결된 마스터 컨테이너의 IP를 사용해야 한다.
$ export ROS_HOSTNAME=<tb3-IP> ⇒ overlay 네트워크에 연결된 터틀봇 컨테이너의 IP를 사용해야 한다.
$ source /opt/ros/noetic/setup.bash
$ export TURTLEBOT3_MODEL=burger
$ source ~/catkin_ws/devel/setup.bash
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch ⇒ 터틀봇을 작동하기 위한 ROS launch 파일을 실행시킨다.
<img width="2048" height="1990" alt="image" src="https://github.com/user-attachments/assets/ada6cd4b-e56f-4941-80b7-408cdd773ebd" />

### 노트북에서 teleop 컨테이너를 overlay 네트워크에 연결해 생성

#노트북
**$ docker run —name teleop —hostname teleop —network rosstack_rosnet -d -it tb3_pc:noetic bash
$ docker network inspect rosstack_rosnet** ⇒ teleop 컨테이너가 overlay 네트워크에 연결되었는지 확인한다.
**$ docker attach teleop** ⇒ 컨테이너에 접속한다.

### 노트북의 teleop 컨테이너에서 teleoperation 세팅

#노트북의 teleop 컨테이너
**$ cd ~/catkin_ws/
$ catkin_make
$ export ROS_MASTER_URI=http://<master-IP>:11311** ⇒ overlay 네트워크에 연결된 마스터 컨테이너의 IP를 사용해야 한다.
**$ export ROS_HOSTNAME=<teleop-IP>** ⇒ overlay 네트워크에 연결된 teleop 컨테이너의 IP를 사용해야 한다.
**$ source /opt/ros/noetic/setup.bash
$ export TURTLEBOT3_MODEL=burger
$ source ~/catkin_ws/devel/setup.bash
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch** ⇒ 터틀봇을 조종하기 위한 teleoperation launch 파일을 실행시킨다.





