![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/ae029311-e27a-4e9a-8ed4-b649197eaeac_image.png)

## 들어가며

나는 예전부터 오픈소스 기여에 대해서 관심이 많았다. 스프링에 대한 오픈소스도 많이 찾아보고
다른 사람은 파이썬과 다른 오픈소스들을 많이 했다고 한다. 전에도 first 이슈를 찾아보라고 많이 해서 막 찾아봤었는데.
그런데 도저히 내가 손댈수 있는곳이 없었다. 그리고 이미 다른 사람들이 커멘트를 달아 이미 할당을 받고있었다.
그래서 잠시 접어두었는데 오늘 문뜩 뭔가 할 게 없을까 찾아보다가 아파치를 들어갔더니 오! 한번 해보면 좋겠다 싶은 첫 이슈가 있었다.

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/b1fb4c04-6593-4d7f-b66b-d38fdd1f0a07_image.png)

이런 이슈가 있는것이다.
그래도 사실 쉽지 않겠다 했는데 요즘 자꾸 머리에 맴도는 말이, 그냥 해! 였다. 뭐든지 그냥 해보자 라는 생각이 갑자기
엄청 들어서 일단 해보기로 하고 커멘트를 달았다.

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/6843c5f9-a674-46fc-b4b7-f16b267a3400_image.png)

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/e21851ba-c3b3-4eea-ad6b-e814849d74c8_image.png)

긍정적인 반응이었다. 할당을 받고, 작업을 시작했다. 문제는 이러했다.

### Apache Seata (incubator-seata) 2.x 브랜치

이슈: ConsulConfigurationTest.testInitSeataConfig가 CI에서 간헐적으로 실패

**에러 예시:**

AssertionFailedError: expected: <val1> but was: <null>
at org.apache.seata.config.consul.ConsulConfigurationTest.testInitSeataConfig(ConsulConfigurationTest.java:121)


현상: 로컬에선 안정적으로 통과(수십 회 반복), CI에서만 가끔 실패 → 전형적인 플래키 테스트

**원인 추정 (진단)**

테스트에서 Consul KV 값을 “쓰기 → 즉시 읽기” 하는 경로에 전파/타이밍 지연이 개입될 여지가 있음

CI 환경은 리소스 경합/스케줄링 등으로 미세한 지연이 더 커질 수 있어 null을 읽어오는 타이밍 레이스 발생

**해결(Deflake)**

프로덕션 코드 변경 없이 테스트만 수정

“쓰기 직후 읽기” 구간에 짧은 대기/재시도 루프 추가 → 최대 약 3초 동안 100ms 백오프로 재시도

기대값을 얻으면 즉시 탈출, 그 외엔 짧게 몇 번 더 확인했다.

**패치 코드**
  ```

@Test
void testInitSeataConfig() throws Exception {
    // Mock initial config load
    GetValue initValue = mock(GetValue.class);
    when(initValue.getDecodedValue()).thenReturn("val1");
    Response<GetValue> initResponse = new Response<>(initValue, 1L, false, 1L);
    when(mockConsulClient.getKVValue(eq("key1"), (String) isNull())).thenReturn(initResponse);

    ConsulConfiguration newInstance = ConsulConfiguration.getInstance();

    // Short retry loop to absorb potential propagation delay in CI environments
    String value = null;
    long deadline = System.nanoTime() + java.util.concurrent.TimeUnit.SECONDS.toNanos(3); // Max ~3 seconds
    do {
        value = newInstance.getLatestConfig("key1", null, 1000);
        if ("val1".equals(value)) break;
        Thread.sleep(100);
    } while (System.nanoTime() < deadline);

    // Verify that the value retrieved matches the expected one
    assertEquals("val1", value, "KV should be visible after a short await");
}
```
위와 같이 코드를 다시 재수정했다. 다시 읽도록 말이다. 

**로컬 재현 & 검증 방법**
먼저 내 로컬에서 테스트를 해보기로 했다. 내 로컬 먼저 설명하겠다. 
아래처럼 도커로 띄워줬다.

macOS 15.6 (Apple Silicon), JDK 17

Consul 1.15 (dev 모드, Docker)

docker run --rm -d --name consul -p 8500:8500 -p 8600:8600/udp \
  consul:1.15 agent -dev -client=0.0.0.0 -ui
export CONSUL_HTTP_ADDR=http://127.0.0.1:8500

**테스트 실행 커맨드**
플러그인이 하나가 자꾸 안먹어서 단일테스트로 그 모듈만 진행했다.

모듈 단위 :./mvnw -pl config/seata-config-consul -am test


**단일 테스트 메서드:**

./mvnw -q -pl config/seata-config-consul -am \
  -Dtest=org.apache.seata.config.consul.ConsulConfigurationTest#testInitSeataConfig \
  -Dsurefire.printSummary=true -Dsurefire.useFile=false \
  -DfailIfNoTests=false \
  test

**PR 작업 절차 (요약)**

포크 → 로컬에 클론

upstream/ origin 설정

git remote rename origin upstream
git remote add origin https://github.com/<내-깃헙아이디>/incubator-seata.git
  
  이렇게 해주었다. 


**브랜치 생성**

git checkout -b fix/consul-config-test-deflake


**수정 & 커밋**

git add .
git commit -m "test: deflake ConsulConfigurationTest#testInitSeataConfig (#7584)"


**푸시**

git push -u origin fix/consul-config-test-deflake


GitHub에서 Compare & pull request → Create pull request

  
  이렇게 올렸다. 
  >일단 내 로컬에서 테스트를 해봤을때 (30번 연속 돌림) 모두 통과를 했다. 종종 실패하였었는데 
  다행히 30번을 돌리면서 모두 통과가 되게 되었다. 그래서 Pr을 올렸다. 


## PR 을 어떻게 올렸지?
![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/d2f11196-f4aa-4f07-92e6-035157bad30f_image.png)

처음에 이런식으로 올렸다. 문제에 대해서,  무슨 문제가 있고, 어떤 이슈를 했는지, 어떻게 해결했는지를 적어주어야한다. 
 올리고 금방 답글이 왔다.
![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/dcba7340-9623-4f39-b863-61824a44e968_image.png)

저 깃허브 md 파일에 나의 깃과, 이슈 해결을 뭘 했는지 적으라는거였다! 와.. 이렇게 바로 된다고 ? 라고 생각했다.  신나게 적은 다음에 바로 푸시했다. 

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/5d0af793-0a87-482f-900c-283ce2df8991_image.png)

아 지금 생각해보니 밑에 내 깃을 적었어야했나 라는 생각이 든다. 그냥 내 커밋만 적었네..
  여튼 좋은게 좋은거다. 
  
  **다시 돌아와서** 이게 빌드에 문제가 있는거라 여러번 액션을 다시 해야한다고 몇번해보고 pr을 해준다고 하였다.
  총 5번정도를 돌렸는데 중간에 한번 실패한것이다. 아 이거 역시 안되나 라고 하고 원인을 알아봤는데 나의 Pr수정사항은 해결이 됐고 다른쪽에서 에러가 난것이었다. 네트워크 문제였던거같다.

![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/ed861301-dfb5-4a11-a586-a15106e76de3_image.png)

에러는 이랬고
![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/263c5885-048c-4376-9764-2ffff97ab699_image.png)

원인을 알아보고 나의 문제가 아니었다. 그것을 다시 커멘트로 적었다. 그러고 몇분후에 갑자기 커멘트가 달렸다! 
ding어플을 쓴다면 개발자 커뮤니티에 초대해준다고 커멘트를 받고 PR이 되었다. 
  
![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/92/5e52042d-db91-4b3e-848a-fa0fa308752f_image.png)

PR이 되고 난 후에 내가 메인으로 올라왔다 ㄷㄷ... 간단한걸 해결했지만 정말 자신감이 가득 찼다. 
  내가 스타 26k에 PR을 했다니! 

## 마치며 
너무 신기한 경험이었다. 공부를 하다가 문뜩 갑자기 생각이 나서 뒤져보다가 들어가서 해결한게 아파치라니! 
물론 첫 이슈에서 쉬운 이슈였지만 나는 **아파치**라는 레포에 나의 이름이 올라간게 감격스럽다. 
생각보다 첫 이슈가 많이 올라오지 않는 레포인데 딱 보고 한 게 정말 운이 좋았다고 생각한다. 
중간에 에러가 한번 났을때 아.. 이게 맞을까 라는 생각도 들었지만 그냥해.. 라는 생각으로 했더니 잘 됐다. 
나중에 기회가 된다면 Spring에 도전해보고싶다. 
  
이슈: https://github.com/apache/incubator-seata/issues/7583
PR: https://github.com/apache/incubator-seata/pull/7584