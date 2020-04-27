---
title: 환경 구성  
weight: 20
pre: "<b>1.1 </b>"
---


## 사전 준비 사항

- AWS 계정 및 IAM User : IAM, AWS IoT, Amazon EC2 에 대한 권한이 있는 IAM User를 사용해야 합니다. 
- 브라우저: 최신 버전의 크롬, 파이어폭스


## 리전 선택

- AWS 리전: 이번 실습은 ***버지니아 북부*** (us-east-1) 리전에서 실행합니다.

## 리소스 생성 

### EC2 키페어 생성 **(이미 생성된 경우 이 단계 생략 가능)**

1. EC2 키페어는 EC2인스턴스 생성을 위해 필요한 항목이므로, CloudFormation 템플릿을 실행하기에 앞서 EC2 키페어를 생성합니다. 
	
2. 이미 EC2 키페어가 생성된 경우라면, 이 단계를 생략합니다. EC2 키페어가 없다면 EC2 콘솔로 [접속](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#KeyPairs:)하여 키페어를 생성합니다.
	
3. EC2 콘솔 화면의 좌측 메뉴에서 “키 페어”를 클릭하고 “키 페어 생성” 버튼을 클릭합니다.
	
4. “키 페어 이름”에 “SFworkshop-keypair”를 입력한 후, “생성”을 클릭합니다.
	
5. 생성이 완료되면 “SFworkshop-keypair.pem” 파일이 임의의 폴더에 자동으로 다운로드됩니다.
	
![Keypair](/lab1/image/keypair_1.png)

{{% notice warning %}}
다운받은 프라이빗 키는 분실하면 다시 복구할 수 없습니다. 워크샵동안 잘 관리해주시기 바랍니다. 
{{% /notice %}}



### cloudformation 으로 초기 환경 구성 

직관적인 IoT 환경을 구현하기 위해서는 raspberry pi 혹은 intel miniPC와 같은 장비로 진행을 하는 것이 가장 좋지만, 해당 자원을 확보하지 않은 사용자를 위해 EC2로 device를 시뮬레이션 합니다. 
워크샵 환경을 구성하기 위해 CloudFormation을 실행합니다. CloudFormation을 통해 다음과 같은 AWS 리소스가 생성됩니다. 


| AWS 서비스 유형| 리소스명	| 기타 |
|-------------|--------|-----|
|VPC	 |SFworkshop-VPC	| 10.0.0.0/16|
|VPC Subnet|	SFworkshop-public-Sub	|10.0.0.0/24|
|VPC Subnet|	SFworkshop-private-Sub	|10.10.2.0/24|
|Cloud9|	SFworkshop-GGC | 개발환경, c5.large|
|EC2|	SFworkshop-thing-instance | GGAD용 인스턴스, t3.large|
|EC2|	SFworkshop-admin-instance | admin용 인스턴스, t2.micro|
|기타 서비스|	Elastic IP, Route table, Security Groups, NAT Gateway, IAM Role 등||


1. [AWS 관리 콘솔](https://console.aws.amazon.com/)로 접속해서, 아래의 화면과 같이 AWS 버지니아 북부 리전이 선택되었는지 확인합니다.{{% button href="https://console.aws.amazon.com/" icon="fas fa-terminal" %}}Open AWS Console{{% /button %}} 본 실습 의 모든 과정은 버지니아 북부(N.Virginia) 리전에서 이루어지므로, 원활한 실습 진행을 위해서 반드시 올바른 리전이 선택되었는지를 확인하시기 바랍니다.
![Region Select](/lab1/image/region_select.png)

2. CloudFormation 템플릿을 실행하기 위해 [링크](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://do-not-delete-smartfactory-cloudformation-template.s3.amazonaws.com/template/iotsecuretunneling.template&stackName=SmartFactoryworkshop)를 클릭합니다. [링크](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://do-not-delete-smartfactory-cloudformation-template.s3.amazonaws.com/template/iotsecuretunneling.template&stackName=SmartFactoryworkshop)를 클릭하면 아래와 같은 화면이 표시됩니다. 파라미터는 수정하지 않으셔도 됩니다. 
	
![cloudformation](/lab1/image/cloudformation_1.png)

3. 정상적으로 진행이 되는 경우 아래와 같이 2개의 스택의 상태가 "CREATE_COMPLETE)으로 되어 있는 것을 확인하실 수 있습니다. 
![cloudformation](/lab1/image/cloudformation_3.png)

{{% notice info %}}
 해당 템플릿은 별도의 VPC 와 EIP 등을 추가로 생성하므로  기존에 사용하던 계정에 VPC 나 EIP 의 수가 한계에 도달한 경우 스택 생성이 실패할 수 있습니다. 그런 경우에는 추가 자원을 확보한 뒤에 다시 스택 생성을 해주시기 바랍니다. 
{{% /notice %}}


### GGAD 용 EC2 인스턴스 접속확인  


4. **SmartFactoryworkshop** 스택의 출력 탭에서 **Cloud9**의 값을 확인하고 클릭하여 접속합니다. 
![cloudformation](/lab1/image/cloudformation_output_1.png)
![cloud9](/lab1/image/cloud9_1.png)

{{% notice info %}}
이 워크샵에서 cloud9 인스턴스는 private subnet에 있는 GGAD용 EC2에 접근하기 위한 bastion host 역할을 맡게 됩니다. 통일성을 위해 앞으로 이 cloud9 인스턴스는 **GGC**(Greengrass Core)라고 부르도록 하겠습니다. 
{{% /notice %}}

5. cloud9에서 GGAD(EC2 instance)인스턴스로 접근할 수 있도록 keypair를 가져와야합니다. 상단의 File > Upload Local Files… 메뉴를 이용하여, 처음에 다운 받은 ***SFworkshop-keypair.pem*** 파일을 Cloud9 IDE에 업로드합니다.
![cloud9](/lab1/image/key_copy_1.png)
![cloud9](/lab1/image/key_copy_2.png)

6. import한 키 파일의 권한을 변경합니다. 
```
chmod 400 ~/environment/SFworkshop-keypair.pem
```

7. 원활한 워크샵 진행을 위해 cloudformation 으로 생성한 EC2 인스턴스의 private IP를 찾아 환경 변수로 설정합니다. 
![cloudformation](/lab1/image/cloudformation_output_2.png)
```
export GGAD_IP=<사설IP>
```

{{% notice info %}}
이 워크샵에서 EC2 인스턴스는 Greengrass Awared Device 역할을 맡게 됩니다. 통일성을 위해 앞으로 이 cloud9 인스턴스는 **GGAD**(Greengrass Awared Device)라고 부르도록 하겠습니다. 
{{% /notice %}}


8.  정상적으로 접속이 되는지 확인합니다. *Are you sure you want to continue connecting (yes/no)* 문구가 나오면 *yes*라고 입력하여 응답합니다. 
```
ssh -i ~/environment/SFworkshop-keypair.pem ubuntu@$GGAD_IP
```

9. 정상적으로 로그인이 되는 것이 확인되면 다시 GGAD 인스턴스틑 빠져나옵니다. 
```
exit
```

* * *

- 위와 같은 작업을 통해 아래와 환경이 준비됩니다. 
![Lab Diagram](/lab1/image/architecture_1.png)



---
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
