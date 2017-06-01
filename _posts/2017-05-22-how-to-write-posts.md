---
layout: post
title: 'Jekyll 블로그 github를 통해 퍼블리싱하는 방법'
tags: [programming]
description: >
  Jekyll(지킬)을 활용해서 블로그 글을 작성하고 github를 통해 배포하는 방법
---
지난 포스트를 통해 Jekyll을 설치하고 시작하는 방법에 대해 알아봤는데요.

> [Jekyll로 시작하는 블로그](http://blog.hyeyoonjung.com/2017/05/04/how-to-start-jekyll/)

지난번에 만든 파일을 활용해서 Github에 연결하고 배포하는 방법을 소개해드릴게요.

```shell
.
├── Gemfile
├── Gemfile.lock
├── _config.yml
├── _posts
│   └── 2017-05-06-welcome-to-jekyll.markdown
├── _site
│   ├── about
│   │   └── index.html
│   ├── assets
│   │   └── main.css
│   ├── feed.xml
│   ├── index.html
│   └── jekyll
│       └── update
│           └── 2017
│               └── 05
│                   └── 06
│                       └── welcome-to-jekyll.html
├── about.md
└── index.md

9 directories, 11 files
```

Jekyll을 오류없이 설치했다면 생성한 블로그 폴더에 위와 같은 구조를 갖춘 폴더와 파일들이 설치됩니다.

### 블로그 기본 설정(setting)

Jekyll 설치 후 가장 먼저 해야할 일은 블로그의 타이틀을 비롯해서 필자의 정보(이메일, SNS 계정 등)을 설정하는 것인데요. 블로그 폴더 최상단에 위치한 `_config.yml` 파일에서 블로그 기본 세팅을 할 수 있습니다.

```
title: Your awesome title
email: your-email@domain.com
description: > # this means to ignore newlines until "baseurl:"
  Write an awesome description for your new site here. You can edit this line in _config.yml. It will appear in your document head meta (for Google search results) and in your feed.xml site description.
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
twitter_username: jekyllrb
github_username:  jekyll
```

### 블로그 글 작성

Jekyll은 마크다운(markdown) 형식으로 포스트를 작성해서 Github을 통해 퍼블리싱을 하는 방식으로 블로그를 운영하는데요. `_post` 폴더에 `.markdown`, `.md` 형식으로 블로그 글을 작성하고 저장하면 됩니다. 여기서 지켜야 할 규칙 몇 가지가 있는데요.

#### 포스트 파일 형식

블로그에 올릴 각각의 포스트는 일정한 이름 규칙을 따라야 합니다. Jekyll블로그를 설치했을 때 기본적으로 생성되는 예시 포스트를 보면

`2017-05-06-welcome-to-jekyll.markdown`

위와 같은 형식을 갖는데요. `년-월-일-포스트 제목` 순으로 파일 이름을 작성하면 됩니다.

#### 포스트 기본 형식

그리고 포스트의 내부를 보면 상단에 포스트에 관련된 정보를 작성하는 란이 있습니다.

```markdown
---
layout: post
title:  "Welcome to Jekyll!"
date:   2017-05-06 13:45:35 +0900
categories: jekyll update
---
```

글을 작성하기 전 상단에 위와 같은 형식으로 제목, 날짜, 카테고리 등을 설정해주면 퍼블리싱을 했을 때 글의 제목과 작성된 날짜 등이 포스트에 나타납니다.

![블로그 설정](http://blog.hyeyoonjung.com/public/img/2/jekyll_post.png)

### Github에 배포하기
이제 블로그를 온라인에 배포해야하는 단계가 남았는데요. Github을 통해 퍼블리싱을 할 경우 무료로 블로그를 배포할 수 있다는 장점이 있습니다. Github에 퍼블리싱을 하기 위해서는 일단 Github 계정이 있어야 합니다. 계정이 없을 경우 Github에서 계정을 만들어주세요.
[Github 사이트](https://github.com/)

#### Github 레포지토리 만들기
회원가입 혹은 로그인을 완료했다면, 우측 상단의 +아이콘을 누르고 New Repository를 클릭하거나, 우측 하단의 New repository라는 버튼을 클릭해서 레포지토리를 생성할 수 있습니다.

![Github repository](http://blog.hyeyoonjung.com/public/img/2/git_repository2.png)

#### Github에 블로그 폴더 업로드
레포지토리를 만드는 것까지 완료했다면, 이제 터미널 혹은 터미널이 익숙하지 않은 경우 소스트리(source tree)와 같은 프로그램을 이용해서 블로그 폴더를 Github 사이트에 업로드하면 됩니다.

아래의 명령어를 순서대로 터미널에 입력하면 됩니다.
```shell
# Jekyll 블로그를 설치한 터미널로 이동 후 git 활성화
git init
# 블로그 폴더를 git staging 상태로 올리기
git add .
# 커밋 메세지 작성
git commit -m "Add blog"
# github 레포지토리 주소 연결
git remote add origin 본인의 레포지토리 주소
# github에 파일 업로드
git push -u origin master
```
터미널 환경 혹은 git에 익숙하지 않다면 이 과정에서 어려움을 겪을 수 있는데요. 생활코딩과 같은 곳에서 git에 관련된 강의를 들으면 좀 더 쉽게 github를 활용할 수 있습니다.

[생활코딩: 지옥에서 온 Git](https://opentutorials.org/course/2708)

위의 명령어가 제대로 동작했다면 아래와 같이 파일들이 올라간 것을 확인할 수 있습니다.

![Github](http://blog.hyeyoonjung.com/public/img/2/github.png)

#### Github의 git pages 기능으로 블로그 배포하기
블로그 배포를 위해서는 git pages 기능을 활용해야하는데요. 정적인 웹사이트를 Github을 통해 무료로 퍼블리싱할 수 있는 기능입니다. github 레포지토리 페이지에서 상단에 Settings를 클릭합니다. 이 곳에서 레포지토리의 이름를 바꾸고 콜라보레이터를 등록하는 등의 작업을 할 수 있는데요. 아래로 내려서 Github pages라는 부분을 찾습니다.

![Github pages](http://blog.hyeyoonjung.com/public/img/2/github_pages.png)

그리고 source를 클릭해서 배포를 원하는 브랜치를 선택하는데요. 저는 일단 master 브랜치를 배포 대상으로 선택했습니다. 그리고 save를 클릭하면 퍼블리싱되고 있는 주소가 상단의 이미지처럼 나타납니다. 약 5~10분 정도 후에 주소를 들어가면 본인의 블로그가 퍼블리싱 된 것을 확인할 수 있습니다.

![Github pages](http://blog.hyeyoonjung.com/public/img/2/github_pages2.png)
