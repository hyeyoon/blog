---
layout: post
title: '초보자를 위한 깃(Git) 사용법'
tags: [programming]
description: >
  버전 관리 시스템인 깃(Git)의 사용법에 대해 정리한 글입니다.
---
프로그래밍 프로젝트를 진행할 때 이전 버전의 코드를 보고 싶을 때, 또는 팀원들과 함께 프로젝트를 진행할 때가 있습니다. 또 직접 작성한 코드를 회사, 타인에게 보여주고 싶을 때도 있죠. 그럴 때 유용하게 사용할 수 있는 것이 깃(Git)입니다.

* [Git 공식 홈페이지](https://git-scm.com/)

![깃](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e0/Git-logo.svg/1200px-Git-logo.svg.png)
<center>source: wikipedia</center>

> 깃(Git)은 프로그램 등의 소스 코드 관리를 위한 분산 버전 관리 시스템이다. 기하학적 불변 이론을 바탕으로 설계됐고, 빠른 수행 속도에 중점을 두고 있는 것이 특징이다. 최초에는 리누스 토르발스가 리눅스 커널 개발에 이용하려고 개발하였으며, 현재는 다른 곳에도 널리 사용되고 있다. - Wikipedia

흔히 깃(Git)과 깃허브(Github)를 동일한 것으로 생각할 때가 있는데요. 깃허브는 깃 레포지토리를 웹에 호스팅할 수 있도록 지원하는 서비스입니다. 이러한 호스팅 서비스는 깃허브를 비롯해서 깃랩(Gitlab) 등 다양한 서비스가 있습니다.

* [Github](https://github.com/) : 오픈 소스 프로젝트에 좋음. 비공개 저장소의 경우 유료로 서비스 제공
* [Gitlab](https://about.gitlab.com/) : 비공개 저장소를 무료로 사용할 수 있음.

## 깃(Git) 설치 & 설정
### 깃 설치하기
Mac os에서는 Xcode가 설치되어 있다면 기본적으로 git이 설치되어 있습니다. 깃이 설치되었는지 확인하고 싶다면 터미널 창에 `git --version`라는 깃의 버전을 확인하는 명령어를 통해서 확인할 수 있습니다.

* [깃(git) 설치 가이드라인](https://git-scm.com/book/ko/v1/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EC%84%A4%EC%B9%98)

### 깃 config 설정
깃 설치를 완료했다면 git config 설정을 통해 사용 환경을 세팅할 수 있습니다.

* 사용자 설정
깃을 커밋할 때 입력되는 사용자의 이메일과 이름 정보를 등록하는 방법입니다.
```shell
git config --global user.name "Gil Dong"
git config --global user.email example@example.com
```

* 편집기 설정
깃에서 주로 사용할 텍스트 편집기를 입력합니다. 기본 설정된 편집기는 vi, vim editor인데요. 필자는 vim 에디터가 익숙하지 않아서 기본 편집기를 주로 사용하는 atom으로 설정했습니다.
```shell
git config --global core.editor atom
```

* git config 설정 확인
git config 설정이 제대로 되었는지 확인하고 싶다면 아래와 같은 명령어를 입력하면 됩니다.
```shell
git config --list
```

> 더 자세한 설정을 알고싶다면 공식 문서를 참고하세요.
>>[공식 가이드](https://git-scm.com/book/ko/v1/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EC%B5%9C%EC%B4%88-%EC%84%A4%EC%A0%95)

## 깃 시작하기
깃 설치와 세팅을 완료했다면, 이제 깃을 시작해볼까요? 깃은 레포지토리(폴더)에 깃 저장소를 만들거나 다른 서버에 있는 저장소를 클론할 수 있습니다.

### 레포지토리(폴더)를 깃 저장소로 만들기
터미널 창에서 깃 저장소 설치를 원하는 폴더로 이동한 후 아래의 명령어를 입력하면 `.git`이라는 파일이 생성됩니다.
```shell
git init
```
`.git`은 기본적으로 숨김 파일이기 때문에 숨김 파일을 보이게 설정하거나 터미널 명령어에 `ls -a`를 입력하면 숨김 파일까지 포함한 폴더와 파일을 볼 수 있습니다.

### 레포지토리의 상태 확인하기
깃 저장소를 설치한 레포지토리는 내부에 파일이 수정거나 추가, 삭제와 같은 변경사항이 있을 때를 추적할 수 있습니다.
```shell
git status
```
위의 명령어를 입력하면 변경이 있는 파일에 대해서는 빨간색(기본)으로 `Untracked`상태라고 알려줍니다. `Tracked` 된 파일의 경우 보통 초록색으로 표시됩니다. 만약 변경된 파일이 없을 경우 `nothing to commit, working directory clean`이렇게 커밋할 파일이 없다고 나옵니다.  

### 파일을 staging area로 올리기
![git add](https://git-scm.com/images/about/index1@2x.png)
<center>source: git-scm</center>
위의 그림을 보면 이해가 쉬운데요. 깃 저장소는 그림과 같이 구성되는데요. `untracked`되었다는 것은 아직 working directory에 있다는 것이고, 그 파일을 `tracked`되게 하려면 아래의 명령어를 통해 파일을 staging area로 옮겨줘야 합니다.
```shell
git add 파일명     # 파일 단위로 올리는 방법
git add .        # 변경된 전체 파일을 올리는 방법
```
이렇게 파일을 staging area로 옮겼다면, 이제 커밋을 통해 레포지토리에 변경사항을 적용할 수 있습니다. 그리고 혹시 staging area 올린 파일을 `untracked`로 변경하고 싶다면 아래의 명령어를 통해 해결할 수 있습니다.
```shell
git reset HEAD 파일명
```

### commit을 통해 레포지토리에 변경사항 적용하기
변경사항을 레포지토리에 적용시키기 위해서는 아래의 명령어를 입력해서 커밋하면 됩니다.
```shell
git commit                        # 깃 에디어를 통해 커밋 메세지 입력
git commit -m "commit message"    # 인라인으로 커밋 메세지 입력하는 법
```
만약 staging area에 파일을 추가하고 커밋 메세지를 입력하는 것이 번거롭다면 `-a`옵션을 추가해서 한 번에 해결할 수 있습니다.
```shell
git commit -a -m "commit message"
```

이렇게 깃 기본 사용법에 대해 알아봤습니다. 깃을 잘 활용하면 프로젝트 관리에 유용하게 사용할 수 있는데요. 만약 터미널 환경이 익숙하지 않다면 소스트리와 같은 프로그램을 활용하는 방법도 있습니다.

* [source tree](https://www.sourcetreeapp.com/)

또한 깃에 대해 더 자세히 알고 싶다면 아래의 강의를 추천합니다. 필자는 Udacity 강의를 통해 깃을 공부했는데요. 생활코딩 또한 Git 강의를 제공하고 있습니다.
* [생활코딩 - 지옥에서 온 Git](https://opentutorials.org/course/2708)
* [Udacity - How to Use Git and GitHub](https://www.udacity.com/course/how-to-use-git-and-github--ud775)

## 참고문헌
* [Git 공식 페이지](https://git-scm.com/)
