![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/156/042c36dc-a6af-4505-a9ea-78a40aeab7ad_image.png)
## 들어가며
이번에 새로운 프로젝트를 들어가게됐다. 
`항상 프로젝트를 처음 들어갈때 느끼는거지만, 전의 코드가 많이 아쉽다이다.`
코드를 막 짤때는, 와 이번건 잘 짠 거 같은데? 라고 생각이 들고
1~2달만 지나도, 아쉽다는 소리가 나온다. 그리고 코드를 짤 때
이건 대충 구현만 시켜놓고 나중에 리팩토링 해야지라고 생각한 부분이
쌓이다보니 코드가 별로였던 거 같아서, 이번 프로젝트에는
좀 제대로 해보려고 생각하고있다. 아마 이번에 짜고도 몇달후에 보면
나의 코드가 별로일 것을 알지만 현 상태에서 최선의 선택을 하려고한다.

가장 먼저 할것은, `정적 분석 도구`이다. 
팀 프로젝트를 하면, 코드 리뷰시에 놓치는 부분이 많이 생긴다.
"이 메서드는 너무 복잡한데?" , "이 변수는 사용하지 않는거같은데"등과
같이 말이다. 사람이 다 잡기엔 한계가 있고, 시대도 많이 변해 도움을
받을 수 있을 거 같았다. 그래서 이번에 **정적 분석 도구**라는 것을
도입하게 되었다. 버그, 취약점, 코드 스멜등을 자동으로 잡아주는 도구다.

이 글에서는 이번 프로젝트에 도입한 **SonarCloud**에 대해 정리하려고한다.

### 정적 분석이 뭘까?
정적 분석은, 프로그램을 실행하지않고 **소스 코드**만 보고 분석하는것이다.
반대로 동적 분석도 있다. 프로그램을 실행하면서 분석하는것이다.

>**정적 분석**
- 소스코드
- 코드 전체
- 버그, 취약점, 코드 스멜 탐지

>**동적 분석**
- 실행 환경
- 실행 가능한 경로만
- 테스트, 모니터링

정도의 차이이다. 정적 분석은 개발 단계에서 코드의 구조적 문제를 찾는데 사용된다.

### SonarCloud란 무엇일까?
![](https://velog.velcdn.com/images/jihun4452/post/88f6b3ba-c37b-4c5f-a365-1e5b059e5919/image.png)

SonarQube는 대표적인 정적 분석 도구다. 20개 언어를 지원하고 
코드의 문제점을 잘 잡아준다.

**SonarCloud**는 SonarQube의 클라우드(SaaS) 버전이다.

`SonarQube: 직접 서버 구축해서 운영`
 `SonarCloud: 클라우드라서 서버 관리가 필요없다`

내가 이번에 SonarCloud를 선택한 이유는
- 서버 관리가 필요없어서
- PR 코멘트가 무료여서
- Github 연동이 쉬워서이다.

### SonarCloud가 잡아주는 것들
SonarCloud는 크게 `다섯 가지를 분석`해준다.

**Bugs**
런타임에 터질 수 있는 잠재적인 버그다.
null 체크 안 한 것, 무한 루프 가능성 같은 것들을 잡아준다.

**Vulnerabilities**
SQL 인젝션, XSS 같은 보안 취약점이다.
OWASP Top 10 기준으로 검사해준다.

**Code Smells**
당장 버그는 아닌데 유지보수를 힘들게 만드는 코드다.
메서드가 너무 길다거나, 중첩이 너무 깊다거나 하는 것들이다.

**Duplications**
중복 코드 비율을 알려준다.
같은 로직이 여러 군데 복붙되어 있으면 잡아준다.

**Coverage**
테스트 코드 커버리지다.
이건 SonarCloud 단독으로는 안 되고 JaCoCo 같은 도구랑 연동해야 한다.

>나는 먼저 테스트 커버리지때문에 정적 분석도구를 사용하려고 했다.
그 다음은 코드의 품질이었다. 클로드나 다른 AI들도 코드 리뷰를 해줄 수 있지만
커버리지 자체를 분석해주지는 않기때문에 다른것이 필요하다고 판단했던거 같다.
정적 분석도구도 생각보다 종류가 많아 고민이 많았다.

## SonarCloud 연동하기

### 1. SonarCloud 계정 연결
sonarcloud.io에 접속해서 GitHub 계정으로 로그인한다.
OAuth로 연결되니까 별다른 설정 없이 바로 된다.

로그인하고 나면 `Organization`과 `Repository`를 선택할 수 있다.
Analyze project에서 원하는 레포를 선택하면 끝이다.

이렇게만 해도 PR 올릴 때마다 자동으로 분석해서 코멘트를 달아준다.

### 2. Github Actions 연동
이렇게만 하면 근데 **테스트 커버리지**를 볼 수 없다. 
`No Coverage Information`이라고 뜬다. 

커버리지까지 보려면 CI에 SonarCloud 분석을 통합시켜야한다.
**GitHub Actions**로 연동하는 방법을 밑에 설명하려고 한다.

#### Sonar-project.properties설정
프로젝트 루트에 `sonar-project.properties` 파일을 만든다.

```java
sonar.projectKey=your-org_your-repo
sonar.organization=your-org
sonar.host.url=https://sonarcloud.io

sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=build/classes/java/main

sonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml

sonar.exclusions=**/test/**,**/Q*.java,**/*Doc*.java
```

각각 설명을 하자면,
- `sonar.projectKey`: SonarCloud 프로젝트 키. SonarCloud 대시보드에서 확인할 수 있다
- `sonar.organization`: SonarCloud Organization 이름
- `sonar.sources`: 분석할 소스 코드 경로
- `sonar.tests`: 테스트 코드 경로
- `sonar.java.binaries`: 컴파일된 클래스 파일 경로. 이거 설정하면 분석 정확도가 올라간다
- `sonar.coverage.jacoco.xmlReportPaths`: JaCoCo 커버리지 리포트 경로
- `sonar.exclusions`: 분석에서 제외할 파일 패턴. QueryDSL의 Q클래스 같은 건 제외한다.

이렇게 설명할 수 있다. 

#### build.gradle 설정
```java
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
    id 'jacoco'
    id 'org.sonarqube' version '4.4.1.3373'
}

jacoco {
    toolVersion = "0.8.11"
}

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        html.required = true
    }
}

sonar {
    properties {
        property "sonar.projectKey", "your-org_your-repo"
        property "sonar.organization", "your-org"
        property "sonar.host.url", "https://sonarcloud.io"
    }
}
```
- 

`jacocoTestReport`에서`xml.required = true`가 중요하다.
SonarCloud는 XML 형식의 커버리지 리포트를 읽기 때문이다.

#### GitHub Actions Workflow
`.github/workflows/sonar.yml` 파일을 만든다.
```java
name: SonarCloud Analysis

on:
  push:
    branches: [ main, develop ]
  pull_request:
    types: [ opened, synchronize, reopened ]

jobs:
  sonarcloud:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 전체 히스토리 가져오기. SonarCloud가 blame 정보 활용함

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cache Gradle packages
        uses: actions/cache@v3
        with:
          path: ~/.gradle/caches
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
          restore-keys: ${{ runner.os }}-gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build and Test with JaCoCo
        run: ./gradlew clean build jacocoTestReport

      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          
```

단계별로 뜯어보면:

**fetch-depth: 0**
```
name: Checkout
 uses: actions/checkout@v4
 with: fetch-depth: 0`
 ```

전체 Git 히스토리를 가져온다. SonarCloud가 blame 정보를 활용해서
"이 코드 누가 언제 작성했는지"를 분석하기 때문이다.
기본값인 1로 하면 최신 커밋만 가져와서 blame 정보가 없다.

**JaCoCo 리포트 생성**

```
name: Build and Test with JaCoCo
 run: ./gradlew clean build jacocoTestReport`
```
테스트를 실행하고 JaCoCo 커버리지 리포트를 생성한다.
`build/reports/jacoco/test/jacocoTestReport.xml` 경로에 XML 파일이 생긴다.

**SonarCloud Scan**

```
- name: SonarCloud Scan
 uses: SonarSource/sonarcloudgithubaction@master
 env: 
 	GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} 
 	SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
 ```
SonarCloud 공식 GitHub Action을 사용한다.
`GITHUB_TOKEN`은 자동으로 제공되고, 
`SONAR_TOKEN`은 직접 발급받아서 시크릿에 등록해야 한다.

#### SONAR_TOKEN 발급

1. [sonarcloud.io](https://sonarcloud.io) 접속
2. 우측 상단 프로필 > My Account > Security
3. Generate Tokens에서 토큰 생성
4. GitHub 레포 > Settings > Secrets and variables > Actions
5. New repository secret으로 `SONAR_TOKEN` 추가
토큰은 한 번 발급하면 다시 볼 수 없으니까 잘 저장해둬야 한다.

이제 설정이 다 끝났다.

### PR 분석 결과 확인
설정을 다 하고 PR을 올리면 이런식으로 코멘트가 달린다.

```
Quality Gate passed

Analysis results:
- 0 Bugs
- 0 Vulnerabilities  
- 2 Code Smells
- 0 Security Hotspots
- 85.3% Coverage on new code
- 0.0% Duplications on new code
```

Quality Gate는 SonarCloud에서 설정한 품질 기준이다.
기본 설정은 새로 추가된 코드에 대해:
- 버그 0개
- 취약점 0개
- 코드 스멜 A등급 이상
- 커버리지 80% 이상
- 중복 3% 이하

이 기준을 통과하면 passed, 아니면 failed가 뜬다.

### SonarLint로 로컬에서 미리 검사하기

PR 올리기 전에 로컬에서 미리 검사하고 싶으면 **SonarLint**를 사용하면 된다.

IntelliJ라면 Plugins에서 SonarLint를 설치하면 된다.
설치하면 코딩하면서 실시간으로 문제를 잡아준다.

SonarCloud랑 연동하면 SonarCloud에서 설정한 룰을 그대로 적용할 수도 있다.
Settings > Tools > SonarLint > Connection에서 SonarCloud를 추가하면 된다.

---
### 다른 정적 분석 도구들은 무엇이 있을까?
위에서 말한대로 SonarCloud말고도 여러 도구들이 정말 많다.
각 도구를 정리해보려고 한다.

#### Codacy

Codacy는 설정이 쉽고 UI가 깔끔하다.
GitHub에 연결하면 바로 분석이 시작된다.
```
장점:
- 설정이 간단함
- UI가 직관적
- 여러 언어 지원

단점:
- JavaScript 분석이 ESLint 기반이라 이미 ESLint 쓰고 있으면 중복
- 커스텀 룰 설정이 제한적
```

#### CodeClimate

CodeClimate은 코드 품질뿐만 아니라 **팀 생산성 분석**도 해준다.
PR 사이클 타임, 코드 리뷰 시간 같은 메트릭을 볼 수 있다.
```
장점:
- 유지보수성 점수(Maintainability) 제공
- 테스트 커버리지 추적
- 팀 생산성 메트릭

단점:
- Bitbucket 연동이 약함
- 가격이 비쌈
```

#### DeepSource

DeepSource는 AI 기반이라 **자동 수정(Autofix)**을 제안해준다.
문제를 찾으면 "이렇게 고치면 됩니다" 하고 코드까지 알려준다.
```
장점:
- 자동 수정 제안
- 노이즈가 적음 (진짜 문제만 잡음)
- 설정 없이 바로 시작 가능

단점:
- 지원 언어가 적음 (Python, Go, Ruby, Java, JS 정도)
- 자체 서버 설치 불가
```
#### Snyk Code

Snyk은 **보안에 특화**된 도구다.
코드 품질보다는 보안 취약점을 집중적으로 잡는다.
```
장점:
- 보안 취약점 탐지 능력이 뛰어남
- IDE에서 실시간 스캔
- 의존성 취약점도 같이 검사

단점:
- 코드 스타일이나 복잡도는 안 잡음
- 보안 외 기능은 약함

```

#### ESLint + Prettier
JavaScript/TypeScript 프로젝트라면 ESLint + Prettier 조합도 좋다.
무료고 커스터마이징이 자유롭다.
```
javascript// .eslintrc.json
{
  "extends": ["next", "prettier"],
  "plugins": ["@typescript-eslint"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "warn",
    "no-console": "warn"
  }
}
```

```
장점:
- 완전 무료
- 커스텀 룰 자유로움
- 로컬에서 바로 검사

단점:
- 보안 취약점은 못 잡음
- 언어별로 다른 도구 써야 함
```


### 왜 SonarCloud를 선택했는가

결국 내가 원했던 건 **테스트 커버리지 분석**과 **코드 품질 체크**였다.

Codacy는 설정이 쉽긴 한데, 커스텀 룰 설정이 제한적이었다.
CodeClimate은 팀 생산성 메트릭까지 제공해서 좋아보였는데 가격이 부담됐다.
DeepSource는 AI 자동 수정이 매력적이었지만, 아직 지원 언어가 적었다.
Snyk은 보안에 특화되어 있어서 내가 원하는 커버리지나 코드 스멜 분석은 약했다.
ESLint + Prettier는 프론트엔드에서는 좋지만, Java 백엔드 프로젝트라 맞지 않았다.

SonarCloud는 **커버리지 + 코드 품질 + 보안 취약점**을 다 잡아주고,
public 레포면 무료고, GitHub 연동도 쉽고, 레퍼런스도 많았다.
그래서 이번 프로젝트에는 SonarCloud를 선택했다.

물론 SonarCloud도 완벽하진 않다.
JaCoCo 연동 설정이 처음엔 좀 귀찮고,
private 레포면 유료라는 점도 있다.
그리고 AI 기반 자동 수정 같은 건 DeepSource가 더 낫다.

그래도 **종합적인 코드 품질 관리**라는 측면에서는
현재로서는 SonarCloud가 가장 균형 잡힌 선택이라고 생각한다.

## 마치며
처음엔 설정이 좀 귀찮긴 한데, 한 번 해두면 PR 올릴 때마다 알아서 돌아가니까
장기적으로는 시간이 절약된다.
아마 몇 달 후에 이 프로젝트 코드를 다시 보면 또 아쉬운 점이 보이겠지만,
적어도 지금 할 수 있는 최선의 선택은 한 거 같다.