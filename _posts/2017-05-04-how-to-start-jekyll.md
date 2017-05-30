---
layout: post
title: 'Jekyll로 시작하는 블로그'
tags: [programming]
description: >
  Jekyll(지킬)을 활용해서 블로그를 설치하고 시작하는 방법
---
Jekyll(지킬)은 Markdown(마크다운) 등으로 작성된 파일을 변환시켜서 실제 웹 상에서 읽을 수 있게 해주는 정적 사이트 생성기입니다. Github의 정적인 페이지를 호스팅할 수 있는 기능인 git pages를 통해 jekyll을 통해서 만든 블로그를 퍼블리싱할 수 있습니다.
- [Jekyll 공식 사이트](https://jekyllrb.com/)
- [Jekyll 한글 문서](http://jekyllrb-ko.github.io/)

## Jekyll 설치 방법
> mac Os기준으로 설명했습니다.

### Requirements(Jekyll 3)

Jekyll을 설치하기 위해서는 아래의 환경이 사용자의 os에 설치되어 있어야 합니다.

- Linux, Unix, macOs 환경
- 2.0버전 이상의 [Ruby](https://www.ruby-lang.org/en/downloads/)
- [RubyGems](https://rubygems.org/pages/download)
- [GCC](https://gcc.gnu.org/install/), [Make](https://www.gnu.org/software/make/) (사용하고 있는 os에 GCC와 Make가 설치되어 있지 않은 경우에 설치 필요. 설치되었는지 확인하는 방법은 터미널에 `gcc -v`, `make -v` 를 입력해서 확인 가능)

Jekyll 2 혹은 그 이하의 버전의 경우

- Node JS 또는 다른 JavaScrupt실행환경(CoffeeScript 지원을 위해서 필요)
- Python 2.7

### Jekyll 설치

Jekyll을 설치하기 위한 환경세팅이 끝났다면, 아래의 명령어를 터미널에 입력해서 Jekyll을 설치할 수 있습니다.

```shell
gem install jekyll
```

Jekyll을 설치할 때 에러가 발생할 경우 Jekyll의 [troubleshooting](http://jekyllrb.com/docs/troubleshooting/#configuration-problems) 페이지를 참고해서 문제를 해결하거나, [Jekyll 커뮤니티](https://github.com/jekyll/jekyll/issues/new)에 이슈 사항에 대한 리포트를 보내서 해결할 수 있습니다.  

필자의 경우 기존에 설치되어 있던 Ruby와 RubyGems의 버전 때문에 에러가 발생했는데 아래와 같은 방법으로 문제를 해결했습니다.

```shell
sudo gem update --system	// RubyGems 업데이트
sudo gem install jekyll 	// sudo 권한으로 설치하려 했으나 오류 발생
sudo gem install -n /usr/local/bin jekyll	// 이 방법으로 설치
```

### Jekyll 시작하기

os에 Jekyll을 설치하는 것까지 완료했다면 이제 아래의 명령어를 터미널에 입력해서 블로그를 만들면 됩니다.

```shell
jekyll new blog		// blog 대신 원하는 디렉토리명(폴더명)을 설정해도 됩니다.
```

> 이 과정에서 bundler 관련한 에러가 발생했을 경우, 필자는 다음과 같은 방법으로 bundler를 설치해서 에러를 해결했습니다.
>
>  `sudo gem install -n /usr/local/bin bundler`

설치가 끝나면 명령어를 입력한 경로에 아래의 구조를 갖는 blog(디렉토리명)라는 폴더가 생깁니다.

```shell
.
├── Gemfile
├── Gemfile.lock
├── _config.yml
├── _posts
│   └── 2017-05-06-welcome-to-jekyll.markdown
├── _site
│   ├── about
│   │   └── index.html
│   ├── assets
│   │   └── main.css
│   ├── feed.xml
│   ├── index.html
│   └── jekyll
│       └── update
│           └── 2017
│               └── 05
│                   └── 06
│                       └── welcome-to-jekyll.html
├── about.md
└── index.md

9 directories, 11 files
```

### Jekyll 블로그 실행하기

blog 폴더로 이동한 후 아래의 명령어를 입력하면 로컬 서버(기본 주소: http://localhost:4000)에서 Jekyll로 구축한 블로그를 확인할 수 있습니다.

```shell
cd blog 	// blog 폴더로 이동
jekyll serve --watch	// 로컬 서버로 블로그 실행
```

![블로그](http://blog.hyeyoonjung.com/public/img/1/jekyll-blog.png)

위와 같은 Jekyll 기본 테마를 갖춘 블로그를 확인할 수 있습니다. 다음 포스팅을 통해 Jekyll로 만든 블로그를 Github를 통해 퍼블리싱하는 방법을 소개해드릴게요.

[Jekyll 블로그 github를 통해 퍼블리싱하는 방법](http://blog.hyeyoonjung.com/2017-05-22-how-to-write-posts/)
