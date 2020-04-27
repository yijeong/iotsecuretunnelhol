---
title: IoT device 기본 구성 (디바이스)
weight: 2
pre: "<b>2.2 </b>"
---

- 해당 모듈에서는 앞서 AWS IoT에 등록했던 사물 디바이스에 대한 인스턴스 구성을 합니다. 관련 SDK를 설치하고 sample 스크립트를 이용해서 정상적으로 데이터가 구독되는지까지 확인합니다. 

1. Cloud9 [콘솔](https://console.aws.amazon.com/cloud9/home?region=us-east-1)의 SFworkshop-GGC 환경으로 이동합니다. 

2. 프롬프트를 통해 해당 터미널이 GGC(cloud9, *IAM User*가 나타남) 인스턴스인지 GGAD(EC2 instance, *ubuntu*가 나타남)인스턴스인지 확인하고, 필요시 GGC 인스턴스로 이동합니다. 

3. 상단의 File > Upload Local Files… 메뉴를 이용하여, 다운로드 받은 ***connect_device_package.zip*** 파일을 Cloud9 IDE에 업로드합니다.
![ggadsetting](/lab2/image/ggad_setting_1.png)
![ggadsetting](/lab2/image/ggad_setting_2.png)

4. 다음 명령어를 통해 다운받은 연결 키트를 GGAD 인스턴스로 복사합니다. 
```
scp -i ~/environment/SFworkshop-keypair.pem ~/environment/connect_device_package.zip ubuntu@$GGAD_IP:~/
```

5. 복사가 완료되면 GGAD인스턴스로 접속하고, 다운 받은 파일의 압축을 해제합니다. 
```
# GGAD로 로그인
ssh -i ~/environment/SFworkshop-keypair.pem ubuntu@$GGAD_IP

# 다운받은 파일 확인
ls

# 파일 압축 해제 
unzip connect_device_package.zip 
```

{{% notice info %}}
연결키트에는 해당 사물 디바이스에 대한 인증서와 퍼블릭키,프라이빗 키 그리고 Python을 위한 AWS Device SDK를 다운로드 및 설치하고 간단한 구독 테스트를 하기 위한 시작 스크립트(**start.sh**)가 포함되어있습니다.  
{{% /notice %}}

6. 압축이 해제된 파일들 중 start.sh 에 실행권한을 추가하고, 시작 스크립트를 실행합니다. 
```
# 실행 권한 추가
chmod +x start.sh

# 스크립트 실행 
./start.sh
```

7. 다음과 같은 화면이 나오면서 메시지가 publish 되면 정상입니다. 
![ggadsetting](/lab2/image/ggad_setting_4.png)


8. publish된 메시지가 정상적으로 구독이 되는지 확인하기 위해 IoT 콘솔의 테스트 메뉴로 [이동](https://console.aws.amazon.com/iot/home?region=us-east-1#/test)합니다. 

9. 구독 주제로 start.sh에 미리 설정되어있던 ***sdk/test/Python*** 을 입력한 뒤 주제 구독을 클릭합니다. 
![ggadsetting](/lab2/image/ggad_setting_6.png)

10. 다음과 같이 테스트 메시지가 구독되고 있다면 정상입니다. 
![ggadsetting](/lab2/image/ggad_setting_7.png)

{{% notice info %}}
GGAD 인스턴스에서 스크립트의 실행을 멈추셨다면 더이상 구독 메시지가 보이지 않습니다. 메시지가 보이지 않는다면 스크립트가 실행되고 있는지 확인해주시기 바랍니다. 
{{% /notice %}}

9. 다시 cloud9 의 GGAD 터미널로 돌아와 *ls* 명령어를 실행하면 **aws-iot-device-sdk-python**과 **root-CA.crt** 파일이 생성된 것을 확인할 수 있습니다. 해당 파일들은 **start.sh** 파일의 실행결과로 생성이 되었습니다. 
![ggadsetting](/lab2/image/ggad_setting_5.png)


- 이제 디바이스의 기본 구성이 완료 되었습니다. 

---
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
