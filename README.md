# Github Actions 을 활용한 CI/CD 파이프라인

## repository의 spring boot app을 Github Actions로 배포하기

> 내가 만든 앱을 인터넷에 배포한다. (cloudtype 사용)

### 개념도

![개념도](https://github.com/user-attachments/assets/d5b762a0-da8e-4ccf-8130-9feed927e107)

### 전체 흐름

- 개발자는 `feature/`  로 시작하는 브랜치를 만들어서 test코드를 포함한 수정 작업을 완료한 뒤 Pull Request 생성한다.
- **(자동화) Pull Request를 만들면 해당 브랜치에 대해 `gradle test`를 수행한다.**
- Pull Request 코드의 test가 실패한 경우, Pull Request 를 생성한 개발자는 test 코드를 수정하여 Pull Request를 변경한다.
- Pull Request 코드의 test가 성공한 경우, 다른 개발자들의 승인을 기다힌다.
- 다른 개발자들은 Pull Request의 코드를 승인하거나 댓글로 소통한다.
- **(자동화) main 브랜치에 merge 되면 해당 브랜치를 cloudtype 서버에 배포한다.**

### GitHub Actions 설정

#### Pull Request가 만들어지면 테스트를 수행하는 GitHub Action

```yaml
name: Test Every PR
on:
  workflow_dispatch:
  pull_request:
permissions:
  contents: read
  pull-requests: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup JDK
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: gradle
      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew
      - name: Run gradlew test
        run: ./gradlew test
```

#### Cloudtype에 main 브랜치를 배포하는 GitHub Action

```yaml
name: Deploy to Cloudtype
on:
  workflow_dispatch:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Connect deploy key
        uses: cloudtype-github-actions/connect@v1
        with:
          token: ${{ secrets.CLOUDTYPE_TOKEN }}
          ghtoken: ${{ secrets.GHP_TOKEN }}
      - name: Deploy
        uses: cloudtype-github-actions/deploy@v1
        with:
          token: ${{ secrets.CLOUDTYPE_TOKEN }}
          project: nbc.docker/cicd
          stage: main
          yaml: |
            name: cicd
            app: java@17
            options:
              ports: 8080
            context:
              git:
                url: git@github.com:${{ github.repository }}.git
                ref: ${{ github.ref }}
              preset: java-springboot
```

## 서버 배포

HomeController 만 있는 스프링 부트 앱을 인터넷 서버에 배포한다.

![image](https://github.com/user-attachments/assets/25b46764-0f41-4317-84e2-fd7e584472ac)

## 참고자료
- [클라우드타입](https://cloudtype.io)
- [클라우드타입 이용가이드](https://help.cloudtype.io/guide/get-started-git)
