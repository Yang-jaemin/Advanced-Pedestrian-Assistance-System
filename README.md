# Advanced Pedestrian Assistance system (APAS)

https://project-apas.streamlit.app/

## 🔎 프로젝트 소개

1. 프로젝트 개요
    
    차량을 위한 자율 주행 시스템은 이미 많이 개발되어 왔다. 이에 관점을 바꿔 차량을 위한 자율 주행 시스템이 아닌, 시각 장애인 등 보행에 어려움을 겪는 사람들을 위한 자율 주행시스템을 개발하고자 하였다. 
    
    AI 기술의 도움으로 보행에 위협 요소인 각종 장애물(자동차, 사람, 가로수, 가로등 등)을 피해 보다 안전하고 원할하게 이동할 수 있는 방안을 제시하였다.
    
    스마트폰을 목걸이에 걸고 거리를 걸어다니며, 카메라 기능과 연동된 웹을 통해 보행자 전방에 탐지되는 객체들의 위험 정도를 인식하여 사용자에게 알리고, 사전에 피해갈 수 있도록 돕는 매커니즘을 구현하는 것이 목적이다.
    
2. 기대 효과
    
    사회적 측면 : 눈이 잘 안보이는 보행자들에게 보다 안전한 보행 환경을 제공해줄 수 있다.
    
    기술적 측면 : 배송 로봇을 포함한 다양한 기기에서 활용 가능할 수 있다.
    
<br/>

## 🪛 기능 소개

1. 온라인
    - 실시간으로 카메라 데이터를 활용하여 전방에 장애물이 있는지 탐지하는 기능이다.
2. 오프라인
    - 이미지/영상을 넣어 모델의 결과가 어떻게 나오는지 확인해볼 수 있는 기능이다.
    
<br/>

## 🖼️ 서비스 아키텍쳐

<img width="1344" alt="APAS architecture" src="https://github.com/boostcampaitech5/level2_cv_semanticsegmentation-cv-10/assets/50127209/9d22a937-3435-46f4-b912-1df78b5f00a1">

<br/>

## 🛠️ 사용 기술 소개

1. **streamlit / streamlit cloud**
    - 별도의 어플을 깔지 않고 웹으로 기능을 체험해보기 위해 웹으로 구현하고자 하여 streamlit을 활용하였다.
    - 원격 호스트에서 로컬 디바이스에 접근하기 위해서 getUserMedia() API를 사용하는데, 이는 HTTPS 서빙을 필요로 한다. streamlit community cloud는 기본적으로 HTTPS로 서빙이 가능하여 사용하였다.

2. **streamlit webrtc**
    - realtime inference를 수행하기 위해 서버와 클라이언트 사이의 소켓 전송 방식을 사용해 프레임을 실시간 전송하는 기능을 사용 → 클라이언트에서 전송 코드를 실행해야하는 문제점이 발생
    - 빠른 통신과 유저의 편의성을 위해 webrtc 기능을 탑재한 streamlit-webrtc 라이브러리를 활용하였다.
    - webrtc의 video frame callback을 커스터마이징하여 활용하여 매 프레임 탐지된 위험물을 warning level에 맞춰 bbox를 출력하도록 하였다.
    - 추가로 50프레임마다 현재의 위험물의 개수, 거리에 따라 warning mode 여부를 탐지하고 사용자에게 음성 TTS파일로 알림을 제공하였다.

3. **warning algorithm**
    - 위험도와 방향에 우선순위를 두어 알림을 제공하고자 warning system을 개발하였다.
    - object 가 위험 영역내에 5 개 이상 있을 때 warning mode 를 설정하여 user 에게 주의하라는 신호를 주도록 시스템을 설계하였다.
    - 탐지된 객체들의 위험도를 지정하기 위해 거리에 따라 사다리꼴 모양으로 구획선을 나눈 후, 각도에 따라 진행 방향 상 충돌가능 영역을 설정하였다.
    - EDA 와 카메라 촬영 실험후 실제로 보행시 주의해야할 영역을 safe,yellow,orange,red 단계로 임의로 정의하였고,인류학자인 에드워드 홀의 근접학에 언급된 바에 따라 사람이 불편을 느끼는 개인적 거리인 1.2m 거리와 공적인 거리인 3.6m 거리를 고려하여 아래 그림과 같이 사다리꼴의 warning area 를 정의하였다. 카메라의 위치는 몸통 중앙, 높이는 지상으로부터 1 미터를 기준으로 하였다.
    - 카메라로 찍은 이미지는 소실점이 존재하므로 앞서 언급한 거리와 소실점을 고려하여 물체와 유저의 진행 방향에 대해 고려하였다. 각도를 구해 진행 방향 상의 충돌 가능 영역을 설정하여 아래와 같이 노란색 삼각형과 흰색 삼각형 영역으로 설정하였다.
    - 이 두가지 기능을 적용하여 데이터셋에 투영시켜보면, 아래 그림과 같이 filter 가 도출된다.
    - 이를 8개의 영역으로 나누어 각각의 위험 순위를 메겼고, 그중 가장 위험하다고 여겨지는 물체 한개에 대해서만 tts 로 유저에게 제공하였다.
    - 같은 레벨에 객체가 있을 때에는 우리나라가 우측통행임을 고려하여 중앙 > 왼쪽 > 오른쪽 순으로 우선순위를 두었다.
<img width="1300" alt="스크린샷 2023-08-17 오후 9 59 06" src="https://github.com/Yang-jaemin/Advanced-Pedestrian-Assistance-System/assets/108872973/a7397ce0-e071-4c68-be04-3063f6579fba">
<br/>

<img width="1300" alt="스크린샷 2023-08-17 오후 9 54 39" src="https://github.com/Yang-jaemin/Advanced-Pedestrian-Assistance-System/assets/108872973/ba75e596-4abb-4d7b-a77b-91706537e7d7">
<br/>
<img width="1300" alt="스크린샷 2023-08-17 오후 9 52 26" src="https://github.com/Yang-jaemin/Advanced-Pedestrian-Assistance-System/assets/108872973/e666bff5-8133-4504-bbba-05ad85c58c79">

<br/>


## 👨‍👨‍👧‍👦 Members


| 이름 | github | 맡은 역할 |
| --- | --- | --- |
| 김보경 &nbsp;| [github](https://github.com/bogeoung) | Streamlit web/app 구현, realtime inference 설계 및 구현|
| 양재민 | [github](https://github.com/Yang-jaemin) | Data 전처리, Warning system algorithm Logic 설계 |
| 임준표 | [github](https://github.com/anonlim) | Data 전처리, Warning system algorithm Logic 설계 |
| 정남교 | [github](https://github.com/jnamq97) | Streamlit web/app 구현, realtime inference 설계 및 구현|
<br/>


