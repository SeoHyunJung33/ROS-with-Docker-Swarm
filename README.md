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
 
<img width="915" height="514" alt="image" src="https://github.com/user-attachments/assets/d24393be-8e00-4553-a3f6-dda7be05ad51" />
Swarm을 통해 노트북과 라즈베리파이가 연결되고, 서비스가 배포되면 노트북과 라즈베리파이에 걸쳐서 overlay 네트워크가 형성된다. 
이 overlay 네트워크는 노트북과 라즈베리파이의 컨테이너들이 서로 통신을 할 수 있도록 한다.
 overlay 네트워크 아래에 있는 컨테이너에서 ROS가 실행되므로 ROS 네트워크는 overlay 네트워크에 구현된다.
 overlay 네트워크 아래에 있는 컨테이너에서 실행된 ROS 노드들은 이 overlay 네트워크를 통해 토픽을 주고 받을 수 있다.

