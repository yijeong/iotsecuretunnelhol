---
title: 자원 삭제
weight: 1
pre: "<b>4.1 </b>"
---

## CloudFormation 스택 삭제 

1. CloudFormation 콘솔로 [이동](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false)합니다. 

2. 다음 스택을 삭제합니다. 
    - SmartFactoryworkshop
    - aws-cloud9-SFworkshop-GGC-xxxxx

3. 약 5분 뒤 정상적으로 삭제가 완료되었는지 확인합니다. 


## IoT 자원 삭제 

4. AWS IoT 콘솔로 [이동](https://console.aws.amazon.com/iot/home?region=us-east-1#/thinghub) 합니다. 

5. 다음 자원들을 삭제합니다. 

    1. 사물 : SFworkshop_thing
    2. 터널 : 워크샵에서 사용했던 터널들 
    3. 보안 : 인증서 / 정책


## key pair 삭제 

6. EC2 키페어 콘솔로 [이동](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#KeyPairs:) 합니다.  

7. 다음 자원을 삭제합니다. 

    1. SFworkshop-keypair

{{% notice warning %}}
자원들의 다른 사용처가 있다면 삭제하면 안됩니다. 특히 키페어는 삭제 후 복구할 수 없습니다. 
{{% /notice %}}
---
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
