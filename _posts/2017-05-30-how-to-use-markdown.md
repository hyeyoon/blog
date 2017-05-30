---
layout: post
title: '마크다운(Markdown) 사용법'
tags: [programming]
description: >
  마크다운(Markdown)의 문법에 대해 정리한 포스트입니다.
---
개발을 공부하다보면, 특히 깃허브(github)를 사용하면 마크다운(Markdown)의 필요성을 느끼게 됩니다. 깃허브에 자신이 올린 레포지토리에 관한 설명을 적을 때나, 도큐멘테이션 작업을 할 때 마크다운을 사용하게 되는데요. 현재 제가 작성중인 지킬(jekyll) 또한 마크다운으로 작성합니다. 오늘은 마크다운으로 문서를 정리할 때 꼭 필요한 문법들을 정리해보겠습니다.


## 제목(Heading)
문서를 작성할 때 가장 기본이 되는 제목은 HTML의 `<h1>`~`<h6>` 태그와 유사합니다.

```
# Heading
## Heading
### Heading
#### Heading
##### Heading
###### Heading
```

# Heading
## Heading
### Heading
#### Heading
##### Heading
###### Heading


## 본문(paragraph)
HTML의 `<p>`와 같은 본문은 텍스트를 그대로 작성하면 됩니다.

```markdown
Lorem ipsum dolor sit amet, consectetur adipisicing elit
```
Lorem ipsum dolor sit amet, consectetur adipisicing elit


## 인용(Blockquotes)
인용은 `>`를 넣어서 작성합니다.
```markdown
> Lorem ipsum dolor sit amet, consectetur adipisicing elit
>> Lorem ipsum dolor sit amet, consectetur adipisicing elit

```
> Lorem ipsum dolor sit amet, consectetur adipisicing elit
>> Lorem ipsum dolor sit amet, consectetur adipisicing elit


## 리스트
### 순서가 없는 리스트(Unordered List)
`*` 또는 `-`를 사용해서 순서가 없는 리스트를 작성할 수 있습니다. tab 또는 2칸 띄어쓰기를 통해 중첩된 항목을 작성할 수 있습니다.

```markdown
* Frontend
  * HTML
  * CSS
  * JavaScript
    * Vue.js

- Frondend
  - HTML
  - CSS
  - JavaScript
    - Vue.js
```

* Frontend
  * HTML
  * CSS
  * JavaScript
    * Vue.js

- Frondend
  - HTML
  - CSS
  - JavaScript
    - Vue.js

### 순서가 있는 리스트(Ordered List)

```markdown
1. HTML
2. CSS
3. JavaScript
```

1. HTML
2. CSS
3. JavaScript

```markdown
1. HTML
1. CSS
1. JavaScript
```

1. HTML
1. CSS
1. JavaScript

```markdown
1. Frontend
    1. HTML
    2. CSS
    3. JavaScript
        1. Vue.js
2. Backend
```

1. Frontend
    1. HTML
    2. CSS
    3. JavaScript
        1. Vue.js
2. Backend


## 코드블럭(Code blocks)
코드블럭은 일반 문장 사이에 단어, 짧은 문장 단위로 표현할 수 있는 방법과 여러줄의 코드를 삽입하는 방법이 있습니다.

단어, 한 줄의 코드를 감싸는 경우 `를 앞뒤로 감쌉니다.

```Markdown
마크다운은 코드블럭을 `<pre>`와 `<code>`로 감쌉니다.
```

마크다운은 코드블럭을 `<pre>`와 `<code>`로 감쌉니다.

여러줄의 코드를 나타내는 코드블럭의 경우 코드블럭의 시작과 끝을 ```으로 감싸고 내부에 코드를 작성하면 됩니다.

```javascript
function square(n) {
  return n * n;
}
```

## 수평선(Horizontal Rules)
문단과 문단 사이를 나눌 때 등 사용되는 수평선은 HTML의 <hr />과 같이 동작합니다.

```markdown
* * *
***
*****
- - -
---------------------------------------
```
* * *
***
*****
- - -
---------------------------------------


## 링크(Links)
HTML의 하이퍼링크와 같은 링크는 다음과 같이 작성합니다. title은 생략이 가능합니다.
```markdown
[example](http://example.com "title")

검색엔진은 [구글](https://www.google.com "구글")을 사용합니다.
```
[example](http://example.com "title")

검색엔진은 [구글](https://www.google.com "구글")을 사용합니다.


## 강조(Emphasis)
HTML의 `<em>`과 같은 동작을 하는 강조는 `*`, `_`가 있고 `<strong>`은 `**`와 `__`를 사용합니다. 취소선은 `~~`을 사용합니다.

```markdown
*강조*한 텍스트
_강조_한 텍스트

```
*강조*한 텍스트

```markdown
**강조**한 텍스트
__강조__한 텍스트
```

**강조**한 텍스트

```markdown
~~취소~~한 텍스트
```

~~취소~~한 텍스트

## 이미지 삽입(Images)
이미지는 역시 HTML의 `<img>`태그와 동일하게 작동합니다. 대체 택스트를 삽입할 수 있으며, 링크 또는 로컬의 이미지파일을 연결할 수 있습니다.

```markdown
![대체 텍스트](/경로/example.jpg)
![대체 텍스트](링크)
```

```markdown
![Github](./public/img/3/github.png)
```
![Github](http://blog.hyeyoonjung.com/public/img/3/github.png)

```markdown
![Github](https://assets-cdn.github.com/images/modules/open_graph/github-octocat.png)
```
![Github](https://assets-cdn.github.com/images/modules/open_graph/github-octocat.png)

이상으로 마크다운의 기본 문법에 대해 알아봤습니다.

## 참고문헌
* [존 그루버의 Markdown: Syntax](https://daringfireball.net/projects/markdown/syntax)
