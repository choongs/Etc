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
- github이 repository를 분석해서 적절한 템플릿을 제공.
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
### Event

workflows를 시작하는 방법
- github event 발생 (Ex: push)
- 스케쥴러 이벤트
- 외부 이벤트 (REST API를 통한 webhook 호출)

이벤트를 설정하는 방법은 `on` 이라는 문법을 사용합니다.

```yaml
# 어느 브랜치 든지 push 이벤트 발생하면 workflows 실행 
name : workflows name
on : push
```

```yaml
# POSIX cron 문법으로 스케쥴러 작성, every 5분마다 실행
on:
  schedule:
    - cron:  '0 * * * *'
```

또한 이벤트를 특정브랜치로 한정 지을수 있습니다.

```yaml
# 마스터 브랜치에 /test라는 폴더에서 push가 발생했을 실행
on:
  push:
    branches:
      - master
    #  paths는 옵션, 없을 경우 전체 디렉토리.
    paths:
      - 'test/*'
```

### Runner

workflows는 GitHub-hosted runners 와 self-hosted runners 에서 실행 가능합니다.

##### GitHub-hosted runners
github에서 가상머신 환경을 제공 합니다, 제공하는 OS는 windows, linux , mac os를 제공하며 실행할때마 새로운 가상머신을 제공합니다.
runner를 지정 할 때는 `runs-on` 이라는 문법을 사용합니다.

###### 하드웨어 리소스 
- 2-core CPU
- 7 GB of RAM memory
- 14 GB of SSD disk spac

``` yaml
# ubuntu 머신에서 실행.
runs-on: ubuntu-latest
```

*** 특정 IP 리스트만 허용하는 github를 사용하면  Github-hosted runners를 사용할 수 없다.


##### self-hosted runners
로컬에서 workflows 실행 환경을 구성이 가능합니다. 모든 self-hosted runner는 `self-hosted` 를 사용합니다.
그외에 추가적으로 OS type과 architecture type을 설정 할 수 있습니다.

```yaml
# any self-hosted runner
runs-on: self-hosted

# max os
runs-on: [self-hosted, macos]

#64비트 리눅스
runs-on: [self-hosted, linux, x64]

```

### checkout action
표준으로 제공하는  action들을 사용할 수 있지만 특히나 checkout action은 빌드와 테스트 또는 지속적인 통합을 위해서 반드시 포함되어야하는 action입니다.

```
- uses: actions/checkout@v2

```


### About YAML syntax for workflows

#### `name`
name은 workflow의 이름입니다, 생략이 가능하면 생략시 디폴트 값으로 workflow file 이름이 설정됩니다.

#### `on`

필수값입니다. workflow를 실행하기위하 조건이됩니다. 위에서도 설명했듯이 workflows를 실행하는 방법은 3가지가 존재합니다. 
push, PR  같은  github 이벤트로 실행하는 방법과 스케쥴링 걸어두어서 사용하는 방법, 마지막으로 외부 이벤트( webhook)로 실행하는 방법이 존재합니다.

``` yaml
# only push
on: push

# push or pull request
on: [push, pull_request]

```

##### `on.<push|pull_request>.<branches|tags>`
push, pull request 이벤트가 특정 브랜치 혹은  tags에서 발생할 때 실행하는 방법 .

```yaml
on:
  push:
    branches:    
      - master         # Push events on master branch
    # Sequence of patterns matched against refs/tags
    tags:        
      - v1             # Push events to v1 tag
      - v1.*           # Push events to v1.0, v1.1, and v1.9 tags
```

##### `on.<push|pull_request>.paths`
push, pull request가 특정 폴더를 포함해서 발생할 때 실행하는 방법.

```yaml
on:
  push:
  #java 파일을 push 할때마다 실행 , 와일드 카드 사용가능 . paths 또한 생략 가능 생략시 모든 파일및 디렉토리가 대상.
    paths:
    - '**.java
```

##### `on.schedule`

POSIX cron 문법을 사용하여 UTC 타임의 스케쥴링을 통한 workflows 실행도 가능합니다.

```yaml
on:
  schedule:
    # every 15분
    - cron:  '*/15 * * * *'
```

#### `env`
workflows내 모든 job 과 step에서 사용가능한 환경변수를 설정 할 수 있습니다.
Ex) `jobs.<job_id>.env `, `jobs.<job_id>.steps.env`

만약 동일한 이름의 환경변수가 존재하면, 모든 환경변수는 오버라이딩이 된다고 생각하시면 됩니다. Step이 실행하는동안은 workflows 및 job에 대한 동일한 이름의 환경변수값을 Step에서 사용되는 값으로 대체합니다.Job이 실행되는 동안은 workflows에 대한 동일한 이름의 환경변수값을 대체합니다.

``` yaml
env:
  SERVER: production
```

#### `Jobs` 
Jobs는 기본적으로 하나 이상의 Job으로 실행되고, 기본적으로 job은 병렬로 처리되어진다. 순차적으로 job을 처리하기 위해서는 
`jobs.<job_id>.needs` 키워드를 사용해야한다.

각각의 Job은 `runs-on`키워드에 정의된 환경에서 실행되어집니다.

##### `jobs.<job_id>`
각각 Job은 반드시 id를 가져야하며 Jobs내에서 유일한 값이 되어야합니다. `job_id`는 문자 혹은 _ 값으로 시작해야하며, 영문자,숫자 -,__만 포함되어야 합니다.

```yaml
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

####`jobs.<job_id>.name`
github에서 보여지는  job 이름 

##### `jobs.<job_id>.needs`
기본적으로 job은 병렬로 처리되어지지만 `needs` 키워드를 사용하여서 순차적으로 처리 할 수가 있습니다.

```
jobs:
# job2가 수행하기 위해서는 job1이 성공적으로 수행되어야 한다.
  job1:
  job2:
    needs: job1
# job3이 실행되기 위해서는 job1, job2가 성공적으로 수행되어야 한다.    
  job3:
    needs: [job1, job2]

```

* 작업이 실패를 기본적으로 건너 뛰지만, 조건문을 사용해서 작업을 진행 하게도 가능합니다.

##### `jobs.<job_id>.runs-on`
`runs-on` 키워드는 job이 실행되어지는 환경을 설정하는 부분입니다 (필수값)
실행되는 환경은 GitHub-hosted runner 와 self-hosted runner가 있습니다.

##### `jobs.<job_id>.env`
job 아래 모든 Step에서 사용가능한 환경변수를 설정가능합니다.
 ```yaml
 jobs:
  job1:
    env:
      FIRST_NAME: Mona
 ```
 
##### `jobs.<job_id>.if`
조건이 충족되는 경우에만 job을 실행 할 수 있게 합니다.

##### `jobs.<job_id>.steps`
하나의 Job안에 task들을 순차적으로 실행 가능합니다. Steps은 커맨드, 설정, repository 또는 Docker에 있는 github action을 수행 할 수 있습니다. Step은 Job과 다르게 단계별로 순차적으로 수행됩니다. 각각의 Step은 개별 프로세스에서 수행되어지고, 수행되어지는 환경의 워크스페이스 및 파일시스템에 접근 가능합니다. 환경변수에 대한 변경사항은 Step간 지속되지 않습니다. workflow 한계내에서 무한으로 수행이가능합ㄴ디.

```yaml
# Example
name: Greeting from Mona

on: push

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
    - name: Print a greeting
      env:
        MY_VAR: Hi there! My name is
        FIRST_NAME: Mona
        MIDDLE_NAME: The
        LAST_NAME: Octocat
      run: |
        echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

##### `jobs.<job_id>.steps.id`
유니크한 식별자이며, github에서 제공하는 context를 사용 할 수 있습니다.


##### `jobs.<job_id>.steps.name`
Stops 이름 , Stpes실행시 로그에서 노출되어지는 이름.


##### `jobs.<job_id>.steps.uses`
동일한 저장소나 공개된 저장소, 혹은 Docker에 이미 정의된 Action들을 사용할 수 있습니다.
Action을 사용할 때는 version이나 tag number를 같이 사용하길 강력하게 권장합니다, version을 정의하지 않은 경우에는 workflows 가 정상적으로 실행이 되지 않을수가 있습니다.

```yaml
steps:    
  - uses: actions/setup-node@74bc508 # Reference a specific commit
  - uses: actions/setup-node@v1      # Reference the major version of a release   
  - uses: actions/setup-node@v1.2    # Reference a minor version of a release  
  - uses: actions/setup-node@master  # Reference a branch
```

##### `jobs.<job_id>.steps.run`
가상환경내에서 쉘을 사용하여 명령어를 사용합니다. 기본적으로 비로그인상태에서 명령어를 실행합니다. 또한 다른 쉘을 선택 할 수 있고 사용자가 정으해서 사용할 수 있습니다. 모든 플랫폼에서 모든 쉘을 사용할 수 없습니다. 쉘의 종류에 따라 지원되는 플랫폼에 제한이 있습니다.

개별 run은 가상환경내에서 새로운 프로세스에서 실행되어지고, 멀티라인으로 커맨드를 실행 할 수 있습니다. 멀티라인의 실행되는 커맨드는 동일한 프로세스에서 실행되어집니다.

``` yaml
# single-line command
- name: Install Dependencies
  run: npm install
```


```yaml
# multi-line command
- name: Clean install dependencies and build
  run: |
    npm ci
    npm run build
```


```yaml
# 삭제 수행시 디렉토리 지정 (working-directory)
- name: Clean temp directory
  run: rm -rf *
  working-directory: ./temp
```


```yaml

# 쉘을 bash에서 사용하기 
steps:
  - name: Display the path
    run: echo $PATH
    shell: bash
```


