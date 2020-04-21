---
title: IoT device 기본 구성 (콘솔)
weight: 1
pre: "<b>2.1 </b>"
---


- AWS IoT에 연결된 디바이스는 AWS IoT 레지스트리 내에 사물(thing)로 표현됩니다. 사물은 특정 디바이스 또는 논리적 엔터티를 나타냅니다. 사물은 물리적 디바이스 또는 센서일 수 있습니다(예: 전구 또는 벽면 스위치). 또한 사물은 애플리케이션의 인스턴스와 같은 논리적 엔터티(**워크샵에서는 GGAD 인스턴스**) 또는 직접 AWS IoT에 연결하지는 않지만 여기에 연결하는 다른 디바이스와 관련이 있는 물리적 엔터티(예: 엔진 센서 또는 제어 패널이 장착된 차량)일 수도 있습니다.

1. 우선 AWS IoT [콘솔](https://console.aws.amazon.com/iot/home?region=us-east-1)로 이동합니다. 

2. AWS IoT 콘솔의 **온보딩** 메뉴로 이동합니다. 
![iotconsole](/lab2/image/iotconsole_1.png)

- 온보딩은 AWS IoT 디바이스 SDK용 연결 마법사를 사용하여 디바이스 또는 컴퓨터를 AWS IoT에 연결할 수 있는 방법입니다. 


3. 디바이스 온보딩을 클릭합니다. 
![iotconsole](/lab2/image/iotconsole_2.png)

4. 다음 화면에서 시작하기를 클릭하여 진행합니다.
![iotconsole](/lab2/image/iotconsole_3.png)

5. 현재 이 워크샵에서는 ubuntu 18.04 TLS 기반의 EC2 인스턴스를 GGAD용으로 사용합니다. 따라서 플랫폼은 **Linux/OSX**를 선택하고, SDK언어는 **Python**을 선택합니다. 
![iotconsole](/lab2/image/iotconsole_4.png)

6. 사물 이름은 "*SFworkshop_thing*"으로 지정하고 다음 단계를 클릭하여 진행합니다. 
![iotconsole](/lab2/image/iotconsole_5.png)

{{% notice warning %}}
사물의 이름이 달라지면 추후 코드 동작에 문제가 있을 수 있습니다. 
{{% /notice %}}

7. 과정이 완료되면 레지스트리에 사물과 그에 연결된 정책이 생성될 예정입니다. 사물 디바이스에 관련 설정을 하기 위한 연결 키트를 다운로드하고 잘 완료가되었는지 확인합니다. 
![iotconsole](/lab2/image/iotconsole_6.png)
![iotconsole](/lab2/image/iotconsole_7.png)

8. 이 단계는 모듈 2.2에서 수행할 예정이므로 우선 완료를 클릭하여 구성을 완료합니다. 
![iotconsole](/lab2/image/iotconsole_8.png)
![iotconsole](/lab2/image/iotconsole_9.png)

9. 다음과 같이 AWS IoT 콘솔 > 관리 > 사물에서 "**SFworkshop_thing**"이 등록된 것을 확인하실 수 있습니다. 
![iotconsole](/lab2/image/iotconsole_10.png)

10. 원활한 워크샵 진행을 위해 사물의 정책을 수정하겠습니다. "**SFworkshop_thing**"을 클릭하여 **보안** 메뉴로 이동하여 앞서 자동으로 생성되었던 인증서를 선택합니다. 
![iotconsole](/lab2/image/iotconsole_11.png)

11. 메뉴 탭에서 **정책** 를 클릭하고 마찬가지로 앞서 자동으로 생성되었던 정책을 선택합니다.  
![iotconsole](/lab2/image/iotconsole_12.png)

12. 정책 문서 편집을 클릭합니다. 
![iotconsole](/lab2/image/iotconsole_13.png)

13. 기존의 정책을 삭제하고 다음과 같이 수정하고 "**새 버전으로 저장하기**"를 클릭하여 저장합니다. 
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iot:*"
      ],
      "Resource": "*"
    }
  ]
}
```

14. 최종적인 모습은 다음과 같습니다. 
![iotconsole](/lab2/image/iotconsole_14.png)

{{% notice warning %}}
실제 운영환경에서는 최소권한 원칙을 준수하여 필요한 권한만을 부여하는 정책을 적용하는 것이 좋습니다. 
{{% /notice %}}


---
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
