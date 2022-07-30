# AWS regional data transfer 요금 청구

[목록으로 돌아가기](/README.md)

## 문제 발견

오늘 AWS로부터 AWS 프리티어 사용량 경고 이메일을 받았다.

![AWS Freetier](/images/aws-freetier.png)

이메일 자체는 별 거 아니었다. 프리티어에서 무료로 주어지는 750시간 중 85%를 넘게 사용했다, 뭐 그런 내용이다.

나중에 검색하다 알게 된 사실이지만, ec2 인스턴스를 1개만 사용하는 이상 1달 내내 사용해도(24 x 30 = 720) 750시간을 넘길 수는 없으니 큰 의미 없는 내용이라고 한다.

그래도 혹시 몰라서 AWS 결제 대시보드에 들어가 청구서를 확인해 봤다.

그런데 놀랍게도 $0.01, 한화 13원의 사용료가 부과되어 있었다.

![AWS Invoice](/images/aws-invoice.png)

내가 한 거라고 해봐야 강의 내용을 따라서 간단한 텍스트 몇 줄 입출력 한 게 전부인데, 대체 어디서 요금이 청구된 것일까?

화면 아래를 보니 자세한 내역을 확인할 수 있었다.

![AWS Data Transfer Bandwidth](/images/aws-data-transfer-bandwdith.png)

Data Transfer 부분의 Bandwidth에서 요금이 청구되었다.

### *regional data transfer - in/out/between EC2 AZs or using elastic IPs or ELB*

이게 대체 뭘까? 13원이 큰 액수는 아니니 상관 없었지만, 나도 모르는 사이에 요금이 청구될 수 있다는 점이 찜찜해서 원인을 찾아보기로 했다.

## 원인 탐색

먼저 저 문구로 구글링을 했다. 바로 나와 비슷한 문제를 겪은 사람을 발견했고, 그 글에는 이런 답변이 달려있었다.

### *"Any inter-AZ traffic like EC2 in AZ-A to RDS in AZ-B will incur inter-AZ traffic charges."*

무슨 말인지 잘 모르겠지만, 대충 모든 AZ 간 트래픽에는 요금이 부과된다는 뜻인 것 같다. 즉, 내게 청구된 요금은 내가 AZ간 트래픽을 발생시켰기 때문에 청구되었다는 뜻이다.

EC2와 RDS는 대충 알겠는데 AZ는 뭘까?

찾아 보니 AZ는 Availability Zone의 약자로, 가용 영역이라는 뜻이었다. 어디서 본 단어라고 생각했는데, RDS에서 DB인스턴스를 생성할 때 본 적이 있는 녀석이었다.

![AWS AZ](/images/aws-az.png)

가용 영역이라는 물리적으로 분리되어 있는 별개의 데이터센터 같은 것이라 보면 될 거 같다. AWS의 각 리전에는 최소 2개의 AZ가 있고, 서울 리전에는 총 4개의 AZ가 있다.

## 탐색 결과

나는 DB 인스턴스를 생성할 때 4개의 AZ 중 ap-northeast-2a를 선택해서 생성했는데, 알고 보니 내가 생성한 EC2는 ap-northeast-2c에 생성되어 있었다.

(참고로 ap-northeast-2 `==` 아시아 태평양(서울)이다.)

EC2에 올린 스프링 프로젝트에서 어떤 데이터를 DB에 저장하는 순간, 위에서 말한 *inter-AZ traffic*(AZ간 데이터 전송) 이 발생한 것이었다.

그리고 이것은 AWS 프리티어와 관계 없이 요금이 청구되는 것 같다.

## 결론

나는 강의를 들으며 텍스트 몇 줄 저장한 게 전부였기에 13원밖에 청구되지 않았다. 만약 이걸 놓치고 지나갔다가, 추후 개인 프로젝트를 하면서 더 많은 트래픽이 발생했다면 액수는 더 커졌을 것이다.

원인을 찾자마자 DB 인스턴스의 AZ를 EC2와 같은 ap-northeast-2c로 변경했다. 아마 이제는 프리티어 제한을 초과하지 않는 이상 요금이 청구되지 않을 것이다.
