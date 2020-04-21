---
title: 내용 정리
weight: 4
pre: "<b>3.4 </b>"
---

## 정리해보겠습니다

- 소스와 대상 디바이스에 로컬 프록시를 설치가 되어있습니다. 
- 대상 디바이스 (GGAD인스턴스)에서는 AWS에서 예약된 topic($aws/things/[사물이름]/tunnels/notify)의 메시지를 구독(subscribe)하는 스크립트가 실행중입니다. (*proxy-demo.py*)
- 여기에 aws cli 권한이 있는 사용자가 open-tunnel 명령어(```aws iotsecuretunneling open-tunnel --destination-config --region us-east-1 thingName=SFworkshop_thing,services=ssh```)로 새로운 보안 터널과 두개의 토큰을 생성합니다. 
- *proxy-demo.py*는 메시지가 수신되면 destination 용 토큰을 받아 로컬 프록시를 실행합니다. 
- 로컬 프록시를 실행하는 명령어는 다음과 같습니다. ```./localproxy -r us-east-1 -d localhost:22 -t <destination_client_access_token>```
- 소스 디바이스 (admin 인스턴스)에서도 ```./localproxy -r us-east-1 -s 5000 -t <source_client_access_token>``` 와 같이 로컬 프록시를 가동합니다. 
- 다시 소스 디바이스로 ```ssh -i kp-smart-factory.pem ubuntu@localhost -p 5000``` 로 접속을 시도하면, 소스 디바이스의 5000포트가 대상 디바이스의 22번 포트로 포워딩 되어 보안 연결이 정상적으로 접속됩니다.  


수고하셨습니다. 모듈 3 을 모두 완료하셨습니다.
 
---
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
