![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/54/521cb9e3-3d69-433c-aa03-2a5444b73ec7_image.png)

## 들어가며

나는 항상 EC2 내부에서 Redis를 사용해왔다. 그냥 항상 쓰던 명령어들이나, 이 이상의 필요성을 못느꼈기때문에 불편함을 잘 몰랐는데
요즘에 Redis를 자주 사용하다보니까 매번 Termius 들어가서 EC2 들어가서 redis-cli 치고 명령어 치는게 귀찮게 느껴졌다..
그래서 생각을 했는데 분명 GUI툴이 있을텐데.. 왜 내가 생각을 못하고 불편하게 사용을 했던걸까? 라는 생각이 들었다.

> 어떤걸 사용할지 고민이 많았는데, 내가 최종적으로 선택한것은 Another Redis Desktop Manager이었다.

### 왜 Another Redis Desktop Manager를 선택했는가?

사실 따지자면 설치가 제일 간단해보였고 사용법이 어렵지 않아보였다. 그래서 사용한 게 크다.
원래는 터미널로만 사용했으니 뭘 설치하든 괜찮다는 생각이었다.

## 설치 방법

설치 URL:https://github.com/qishibo/AnotherRedisDesktopManager
이 주소로 들어가면 된다.

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/54/404c8e56-b23f-49b5-a056-b03f0b2a3171_image.png)

들어가면 이런 화면이 나온다. 그럼 오른쪽에 Releases를 선택해서 본인에게 맞는 버전을 선택해주면 된다.

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/54/8784ea91-7a03-4061-9902-4eec3dfdca63_image.png)

나같은 경우는 Mac을 사용중이라, 3번째인 mac - 1.7.1-arm를 사용해주었다.
설치를 완료하고 실행시켜주면 된다.

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/54/a39bcf91-c08f-4661-b01f-f7f643f95871_image.png)

접속을 하면 이런 형태의 연결이 뜨게 되는데 자신에게 맞게 잘 조절해서 써주면 된다.
나는 EC2와 연결할거라 SSH를 선택하여 .pem이랑 정보들을 맞게 넣어준 후 확인을 눌렀더니 바로 실행이 됐다.

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/54/f0bbc7f3-943d-445f-aff2-813c6e94e105_image.png)

원래 같았으면 매번 이러면서 확인을 했을텐데

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/54/3dfdf17a-2ad5-4089-b7ca-60b4ed9214d1_image.png)

지금은 이렇게 바로 확인이 가능해서 너무 편해졌다.

### 마치며

지금은 나의 생각 이상으로 편한것들이 많이 존재하니까 불편하다싶으면 바로바로
찾아봐서 적용해야겠다 라고 생각이 많이들었다.
다른 개발들도 할 때 불편할것을 먼저 생각하고 들어가야겠다.