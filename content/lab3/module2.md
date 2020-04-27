---
title: IoT 보안 터널링 구성(수동)
weight: 2
pre: "<b>3.2 </b>"
---

## 소스와 대상 디바이스에서 로컬 프록시 설치하기

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

### AWS 콘솔에서 보안 터널 열기 

4. AWS IoT 콘솔에서 관리 > 터널로 [이동](https://console.aws.amazon.com/iot/home?region=us-east-1#/tunnelhub)합니다. 
![securetunnel](/lab2/image/securetunnel_2.png)

5. 디바이스에 대한 보안 터널 열기에서 **터널 열기**를 클릭합니다. 
![securetunnel](/lab2/image/securetunnel_3.png)

6. 새 터널 열기에서 간단한 설명을 입력합니다. (예 : *secure tunneling for SFworkshop*)
![securetunnel](/lab2/image/securetunnel_4.png)

7. 대상 구성 창을 활성화하고 **ThingName** 에서 이전 모듈에서 생성했던 **SFworkshop_thing**을 선택합니다. 서비스 이름을 ***ssh*** 라고 입력합니다. 
![securetunnel](/lab2/image/securetunnel_5.png)

8. 그 외 나머지 설정은 기본값을 그대로 두고 **터널 열기**를 클릭하여 다음 단계로 진행합니다. 
![securetunnel](/lab2/image/securetunnel_6.png)

{{% notice info %}}
보안 터널의 제한 시간은 12시간이 기본값이며, 1분에서 12시간까지 사용자 정의하여 구성할 수 있습니다. 
{{% /notice %}}

9. 소스 및 대상의 클라이언트 엑세스 토큰을 다운로드하고, 다운로드가 완료된 것을 확인하면 **완료**를 클릭하여 구성을 완료합니다. 
![securetunnel](/lab2/image/securetunnel_7.png)

{{% notice info %}}
엑세스 토큰은 추후 사용하므로 잘 보관합니다.
{{% /notice %}}

10. 다음과 같이 활성화된 터널이 열려 있는 것을 확인할 수 있습니다. 
![securetunnel](/lab2/image/securetunnel_8.png)


## AWS IoT 보안 터널링 테스트  

11. 소스 인스턴스 (admin 인스턴스)에 접속을 해야합니다. 좋아하는 터미널을 이용해 인스턴스에 접속합니다. 

- admin 인스턴스의 IP는 Cloudformation 의 출력탭에서 확인할 수 있습니다. 
    ![securetunnel](/lab2/image/securetunnel_28.png)

- 예를들어, 하기 명령어와 사진의 작업은 MAC OS 의 기본 터미널에서 수행했습니다. 

```
#키페어 권한 부여 
chmod 400 ~/Downloads/SFworkshop-keypair.pem 

# admin인스턴스에서 GGAD인스턴스 접속을 위한 키페어 복사 
scp -i ~/Downloads/SFworkshop-keypair.pem ~/Downloads/SFworkshop-keypair.pem ubuntu@<admin인스턴스의 퍼블릭IP>:~/

#로컬PC에서 admin인스턴스로 엑세스토큰 복사 
scp -i ~/Downloads/SFworkshop-keypair.pem ~/Downloads/xxxxxxxx-sourceAccessToken.txt ubuntu@<adminInstanceId>:~/

# admin 인스턴스에 접속
ssh -i ~/Downloads/SFworkshop-keypair.pem ubuntu@<admin인스턴스의 퍼블릭IP>

# 파일 확인
ls
```
{{% notice info %}}
키페어 파일이나 토큰을 저장한 위치가 다르다면 명령어를 수정하여 수행하여야 합니다. 
{{% /notice %}}

![securetunnel](/lab2/image/securetunnel_11.png)

12. 위의 9 번 단계에서 다운로드 받은 source와 destination 용 토큰을 각각의 device에 복사해야합니다. 우선 대상 인스턴스 (GGAD 인스턴스) 에 destionation 토큰 (예 : *xxxxxxx-destinationAccessToken.txt*) 을 복사합니다. 

- cloud9에서 상단의 File > Upload Local Files… 메뉴를 이용하여, 다운로드 받은 ***xxxxxxx-destinationAccessToken.txt*** 파일을 Cloud9 IDE에 업로드합니다.
![ggadsetting](/lab2/image/ggad_setting_1.png)
![ggadsetting](/lab2/image/ggad_setting_2.png)

- 프롬프트를 통해 해당 터미널이 GGC(cloud9, *IAM User*가 나타남) 인스턴스인지 GGAD(EC2 instance, *ubuntu*가 나타남)인스턴스인지 확인하고, 필요시 GGC 인스턴스로 이동합니다. 

```
exit
```

- 다음 명령어를 통해 다운받은 엑세스 토큰을 GGAD 인스턴스로 복사합니다. 
```
#Cloud9에서 GGAD로 엑세스토큰 복사 
scp -i ~/environment/SFworkshop-keypair.pem ~/environment/xxxxxxx-destinationAccessToken.txt ubuntu@$GGAD_IP:~/

# GGAD로 로그인
ssh -i ~/environment/SFworkshop-keypair.pem ubuntu@$GGAD_IP
```


13. Target device (GGAD 인스턴스)에서 앞서 다운로드 받은 token을 사용하여 local proxy를 실행합니다. 하기와 같이 connection이 성공적으로 맺어집니다. 

```
# 엑세스 토큰값 확인 (복사해놓을것)
cat ~/xxxxxxx-destinationAccessToken.txt

# local 프록시로 경로 이동
cd ~/dependencies/aws-iot-securetunneling-localproxy/build/bin

# local 프록시 실행
./localproxy -r us-east-1 -d localhost:22 -t <destination_client_access_token> 
```
![securetunnel](/lab2/image/securetunnel_23.png)


13. 이번에는 Source device (admin인스턴스) 에서 앞서 다운로드 받은 token을 사용하여 local proxy를 실행해 보겠습니다. 마찬가지로 하기와 같이 connection이 성공적으로 맺어집니다. 

```
# 엑세스 토큰값 확인 (복사해놓을 것)
cat ~/xxxxxxx-sourceAccessToken.txt

# local 프록시로 경로 이동
cd ~/dependencies/aws-iot-securetunneling-localproxy/build/bin

# local 프록시 실행
./localproxy -r us-east-1 -s 5000 -t <source_client_access_token> 
```

![securetunnel](/lab2/image/securetunnel_24.png)


14. Source device (admin인스턴스) 의 터미널을 하나 더 띄워서 local proxy를 실행할 때 지정한 Port (5000) 로 ssh 연결을 시도합니다. 

```
ssh -i ~/SFworkshop-keypair.pem ubuntu@localhost -p 5000

```

- 다음과같이 접속에 성공합니다.
![securetunnel](/lab2/image/securetunnel_25.png)

- 소스 디바이스(admin인스턴스) 측 로그 
![securetunnel](/lab2/image/securetunnel_26.png)

- 대상 디바이스(GGAD인스턴스) 측 로그 
![securetunnel](/lab2/image/securetunnel_27.png)


15. Troubleshooting 혹은 admin 작업 후 tunnel을 삭제합니다. 

 

---
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
