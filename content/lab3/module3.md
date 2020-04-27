---
title: IoT 보안 터널링 구성(자동)
weight: 3
pre: "<b>3.3 </b>"
---

## AWS IoT 보안 터널링 - 자동 구성  

모듈 3에서 진행했던 보안 터널 구성은 직관적이기는 하지만 매번 토큰을 각각의 디바이스에 업로드 해주어야한다는 불편함이 있습니다. 이번 모듈 4에서는 AWS CLI *open-tunnel* 명령어를 이용하면 remote device에 token을 미리 복사해 주지 않아도 되고, Remote device에 미리 local proxy를 설치만 미리 해놓고 ‘실행해 놓지 않아도’ 되는 자동 구성 방법에 대해서 알아봅니다. 

### 소스와 대상 디바이스에서 로컬 프록시 설치하기

1. 소스 디바이스 (admin 인스턴스)와 대상 디바이스 (GGAD 인스턴스)에 로컬 프록시를 설치하기 위해서는 다양한 dependency 소프트웨어를 설치해야하고, 그에 따른 시간이 오래 걸립니다. 따라서 워크샵 펀의상 이 부분은 이미 구성이 되어있는 AMI로 대체되었습니다. 

    - 세부적인 과정이 궁금한 사용자들을 위해 AMI를 구성했던 부트스트랩 스크립트를 **참고**해주시기 바랍니다. 
{{%expand " 부트스트랩 스크립트보기" %}}


```
#!/bin/bash

sudo apt-get update

# 파이썬 설치 
sudo apt-get -y install python3-pip
sudo python3 -m pip install AWSIoTPythonSDK
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 3
sudo update-alternatives --config python

# 폴더 생성
mkdir ~/dependencies
cd ~/dependencies

# local proxy 설치를 위한 dependency 설치 
sudo apt-get -y install build-essential
sudo apt-get -y install cmake
sudo apt-get -y install zlibc
sudo apt-get -y install libssl-dev
sudo apt-get -y install unzip

## Boost dependency 파일 설치 
wget https://dl.bintray.com/boostorg/release/1.69.0/source/boost_1_69_0.tar.gz -O /tmp/boost.tar.gz
sudo tar xzvf /tmp/boost.tar.gz
cd boost_1_69_0
./bootstrap.sh
sudo ./b2 install

## Protobuf dependency 파일 설치 
cd ~/dependencies
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protobuf-all-3.6.1.tar.gz -O /tmp/protobuf-all-3.6.1.tar.gz
sudo tar xzvf /tmp/protobuf-all-3.6.1.tar.gz
cd protobuf-3.6.1
sudo mkdir build
cd build
sudo cmake ../cmake
sudo make
sudo make install


## Catch2 test framework 설치 
cd ~/dependencies
sudo git clone https://github.com/catchorg/Catch2.git
cd Catch2
sudo mkdir build
cd build
sudo cmake ../
sudo make
sudo make install

# local proxy 설치
cd ~/dependencies
git clone https://github.com/aws-samples/aws-iot-securetunneling-localproxy
cd aws-iot-securetunneling-localproxy
mkdir build
cd build
cmake ../
make
```

{{% /expand %}} 

## AWS IoT 보안 터널링 테스트  

2. 대상 디바이스 (GGAD 인스턴스) 에서 로컬 프록시 설정을 확인하겠습니다. Cloud9의 GGAD 터미널로 이동합니다. 

{{% notice warning %}}
GGAD 인스턴스의 터미널이 맞는지 다시한번 확인합니다. 
{{% /notice %}}

3. 명령어를 실행했을 때 다음과 같이 나타나면 정상입니다.  
```
cd ~/dependencies
ls
```    
![securetunnel](/lab2/image/securetunnel_12.png)

4. 다음 명령어를 실행하여 local 프록시의 자동 실행을 위한 스크립트를 다운로드하고, proxy-demo.py 파일을 홈 디렉터리로 복사해옵니다. 
```
cd ~
git clone https://github.com/yijeong/smartfactoryiot.git
cp ~/smartfactoryiot/device/proxy-demo.py ~    
```

5. 파일 편집을 위해 편집기를 실행합니다. 
```
vi ~/proxy-demo.py
```

6. 수정해야 할 곳은 # **Parameter** 의 **iot_endpoint** 부분으로, 다음과 같이 값을 수정합니다. 

    1. AWS IoT 콘솔의 사물 메뉴로 [이동](https://console.aws.amazon.com/iot/home?region=us-east-1#/thinghub)하여 **SFworkshop_thing"을 선택합니다. 
    ![securetunnel](/lab2/image/securetunnel_13.png)

    2. 사물의 메뉴 중 상호 작용을 클릭하면 다음과 같이 REST API 엔드포인트가 기술되어있습니다. 이 값을 **iot_endpoint**에 넣어줍니다. 
    ![securetunnel](/lab2/image/securetunnel_14.png)

    3. *proxy-demo.py*는 대략 다음과 같은 모습이 됩니다. 
    ![securetunnel](/lab2/image/securetunnel_15.png)

7. 이제 스크립트를 실행합니다. 그림과 같이 에러가 나지 않고 대기 상태에 들어가면 정상입니다. 이 스크립트는 예약된 topic($aws/things/<thing-name>/tunnels/notify)의 메시지를 수신하면 destination용 token을 수신하고 local proxy를 실행하는 역할을 합니다. 

```
python ~/proxy-demo.py
```
![securetunnel](/lab2/image/securetunnel_16.png)


8. AWS CLI open-tunnel 명령을 실행하기 위해, 하나의 터미널을 더 열어줍니다. 
![securetunnel](/lab2/image/securetunnel_17.png)
![securetunnel](/lab2/image/securetunnel_18.png)


9. cloud9 인스턴스는 이미 aws cli 및 관련 권한을 갖추고 있습니다. open-tunnel 명령어를 통해 새로운 터널을 열겠습니다. 이 명령어를 통해 신규 ssh tunnel이 생성되고 token도 target device에 배포가 됩니다. ($aws/things/<thing-name>/tunnels/notify 토픽으로 메시지가 전송)
```
aws iotsecuretunneling open-tunnel --destination-config --region us-east-1 thingName=SFworkshop_thing,services=ssh
```
![securetunnel](/lab2/image/securetunnel_20.png)

{{% notice info %}}
추후 사용하기 위해 결과값 중 sourceAccessToken의 값을 복사해둡니다. 
{{% /notice %}}


10. 다시 GGAD 인스턴스 터미널로 돌아가보면 다음과 같이 새로운 tunnel에 대한 공지를 받고 연결을 수립하는 것을 볼 수 있습니다. 
![securetunnel](/lab2/image/securetunnel_19.png)

11. 이제 소스 디바이스 (admin 인스턴스) 에서 로컬 프록시를 실행해보겠습니다. 사용하시는 편한 터미널 도구를 이용해서 (예: putty) admin 인스턴스에 접속합니다. 
    - admin 인스턴스의 IP는 Cloudformation 의 출력탭에서 확인할 수 있습니다. 
    ![securetunnel](/lab2/image/securetunnel_10.png)

    - 사진의 예는 MAC OS 의 기본 터미널입니다. 
    ```
    chmod 400 ~/Downloads/SFworkshop-keypair.pem 
    ssh -i ~/Downloads/SFworkshop-keypair.pem ubuntu@<admin인스턴스의 퍼블릭IP>
    ```
    ![securetunnel](/lab2/image/securetunnel_11.png)

12.  admin 인스턴스 내에서 다음과 같이 명령어를 실행해 로컬 프록시를 시작합니다. sourceAccessToken값은 앞서 16번에서 복사해둔 값을 사용합니다. 정상적으로 실행이 되면 그림과 같이 연결 대기 상태가 됩니다. 

```
# 로컬 프록시 경로로 이동 
cd ~/dependencies/aws-iot-securetunneling-localproxy/build/bin/

# 로컬 프록시 시작
./localproxy -r us-east-1 -s 5000 -t <sourceAccessToken값>
```
![securetunnel](/lab2/image/securetunnel_21.png)

13. 이제 하나의 터미널창을 더 띄우고, 로컬 PC에 저장되어있는 keypair파일 (*SFworkshop-keypair.pem*)을 admin 인스턴스의 홈디렉터리에 복사해옵니다. 명령어의 예시는 MAC OS 기반입니다. 
```
# 로컬 PC에서 실행
scp -i ~/Downloads/SFworkshop-keypair.pem ~/Downloads/SFworkshop-keypair.pem ubuntu@<admin인스턴스의 퍼블릭IP>:~
```

14. admin 인스턴스에 접속하여 키파일이 잘 복사되었는지 확인합니다. 
```
# 로컬 PC에서 실행
ssh -i ~/Downloads/SFworkshop-keypair.pem ubuntu@<admin인스턴스의 퍼블릭IP>

# admin 인스턴스에서 확인 
ls
```

15. 복사해온 키와 로컬 프록시를 이용해서 GGAD 용 인스턴스에 접속해봅니다. 
```
ssh -i ~/SFworkshop-keypair.pem ubuntu@localhost -p 5000
```
![securetunnel](/lab2/image/securetunnel_22.png)

수고하셨습니다. 보안 터널링 구성을 모두 완료하였습니다. 

---
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
