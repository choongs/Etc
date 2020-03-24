# Github Action

## Automate your workflow from idea to production
> GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want.

## Workflows란?

Workflow는 Github 저장소에서 발생하는 build, test, package, release, deploy 등 다양한 이벤트를 기반으로 직접 원하는 Workflow를 만들 수 있다 Workflow는 Runners라고 불리는 Github에서 호스팅 하는 머신의 Linux, macOS, Windows 환경 그리고 컨테이너에서 실행할 수 있다. 물론 Runners를 사용자 호스팅 환경에서 직접 구동시킬 수도 있다.

Github 마켓 플레이스에는 공유된 Workflow는 찾을 수 있으며, 자신이 직접 만든 Workflow를 공유할 수 있는 커뮤니티가 존재한다.

저장소별 wokrflow는 하나이상 만들수 있다.
workflows는 반드시 하나이상의 Job을 가져야하고 Job안에는 여러개의 Step이 존재 할 수 있습니다.

workflows는 github 이벤트가 발생했을때나 스케쥴러로 통해서 실행 할 수 있고 외부 이벤트를 통해서도 실행이 가능합니다.

workflows는 YAML문법을 이용해서 작성을 합니다.

## workflow 만드는 방법
- github이 repository를 분석해서 적절한 템플릿음 제공.
- 마켓플레이스에 공유된 workflow를 가져오는 방법.
- workflows 파일을 직접 생성하는 방법.

### workflows 파일 직접 생성하기.

1. repository 최상단 폴더(root path)에 github/workflows 폴더를 생성.
2. 생성된 폴더안에 yml or yaml 파일을 생성한다.(Ex. D.Collector-deploy.yml)
3. workflows 작업을 작성.
4. 변경된 파일을 github에 올려서 실행한다.

#### workflow example
```yaml
name: Greet Everyone
# push 이벤트가 발생하면 workflows를 실행 .
on: [push]

jobs:
  build:
    # Job name
    name: Greeting
    # 리눅스 환경에서 실행.
    runs-on: ubuntu-latest
    steps:
      # 이 Step은 Github이 제공하는 `hello-world-javascript-action` Action을 사용한다
      - name: Hello world
        uses: actions/hello-world-javascript-action@v1
        with:
          who-to-greet: 'Mona the Octocat'
        id: hello
      # 이 Step은 이전 Step에서 나온 출력을 사용한다
      - name: Echo the greeting's time
        run: echo 'The time was ${{ steps.hello.outputs.time }}.
```
