---
layout: post
title: 'Intersection Observer API의 사용법과 활용방법'
tags: [javascript]
description: >
  Web API 중 하나인 Intersection Observer API를 알아보고 어떻게 활용할 수 있는지에 대해 정리한 글입니다.
---

[Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)(교차 관찰자 API)를 들어본 적이 있나요? 크롬 51버전부터 사용할 수 있는 이 Web API는 2016년 4월 [구글 개발자 페이지](https://developers.google.com/web/updates/2016/04/intersectionobserver) 통해 소개되었습니다. MDN을 비롯해서 `Intersection Observer`를 어떻게 활용할 수 있을지 살펴보니 생각보다 다양한 곳에 적용할 수 있고, 앞으로 유용하게 쓸 수 있을 것 같습니다. MDN에서는 `Intersection Observer`의 필요성을 아래와 같은 예를 들어 설명하고 있습니다.

- 페이지 스크롤 시 이미지를 Lazy-loading(지연 로딩)할 때
- Infinite scrolling(무한 스크롤)을 통해 스크롤할 때 새로운 콘텐츠를 불러올 때
- 광고의 수익을 계산하기 위해 광고의 가시성을 참고할 때
- 사용자가 결과를 볼 것인지에 따라 애니메이션 동작 여부를 결정할 때

> 현재(2019년 1월 기준)는 웹, 모바일 크롬, 안드로이드, 파이어폭스 등에서 지원하고 있으며 아직 사파리, 모바일 사파리에서는 지원하지 않고 있습니다. 따라서 사파리, 아이폰의 경우 예시가 제대로 동작하지 않을 수 있습니다.

## 기존 scroll 이벤트의 문제

웹사이트를 개발할 때 특정 위치에 도달했을 때 어떤 액션을 취해야 한다면 어떻게 구현할 수 있을까요? 보통 `addEventListener()`의 `scroll` 이벤트가 먼저 떠오릅니다. document에 스크롤 이벤트를 등록하고, 특정 지점을 관찰하며 엘리먼트가 위치에 도달했을 때 실행할 콜백함수를 등록하는 것이죠.

하지만 `scroll` 이벤트는 단시간에 수백번, 수천번 호출될 수 있고 동기적으로 실행되기 때문에 메인 스레드(Main Thread) 영향을 줍니다. 또한 한 페이지 내에 여러 `scroll` 이벤트(무한 스크롤, 광고 배너, 애니메이션 등)가 등록되어 있을 경우, 각 엘리먼트마다 이벤트가 등록되어 있기 때문에 사용자가 스크롤할 때마다 이를 감지하는 이벤트가 끊임없이 호출됩니다. (`디바운싱(Debouncing)`과 `쓰로틀링(Throttling)`을 통해 이러한 문제를 개선시킬 수도 있습니다.) 그리고 특정 지점을 관찰하기 위해서는 [getBoundingClientRect()](https://developer.mozilla.org/ko/docs/Web/API/Element/getBoundingClientRect) 함수를 사용해야 하는데, 이 함수는 `리플로우(reflow)` 현상이 발생한다는 단점이 있습니다.

> 리플로우(reflow): 리플로우는 브라우저가 웹 페이지의 일부 또는 전체를 다시 그려야하는 경우 발생한다.

예시를 통해 살펴보겠습니다. 간단한 `addEventListener()`의 `scroll` 이벤트를 구현했습니다. 특정 위치에 도달하면 `box`엘리먼트에 애니메이션을 동작시키는 코드입니다.

<iframe width="100%" height="400" src="//jsfiddle.net/hyeyoon/g1Lrfw76/22/embedded/result,js,html,css" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

```javascript
// 해당 요소가 viewport 내에 있는지 확인하는 함수
// 대표적인 예시로 사용되고 있는 stackoverflow의 예시를 가져왔습니다.
// https://stackoverflow.com/questions/123999/how-to-tell-if-a-dom-element-is-visible-in-the-current-viewport/7557433#7557433
function isElementInViewport (el) {
  var rect = el.getBoundingClientRect();
  return (
    rect.top >= 0 &&
    rect.left >= 0 &&
    rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
    rect.right <= (window.innerWidth || document.documentElement.clientWidth)
  );
}
// scroll 이벤트를 추가하고, 해당 element에 callback 함수를 등록하는 함수
const addEventToEl = (elList) => {
  document.addEventListener('scroll', () => {
    elList.forEach(el => {
    	if (isElementInViewport(el)) {
        el.classList.add('tada');
      } else {
        el.classList.remove('tada');
      }
    })
  })
}
// 동작시킬 elements리스트에 이벤트를 등록
const boxElList = document.querySelectorAll('.box');
addEventToEl(boxElList);
```

해당 코드를 크롬 개발자 도구의 Performance 탭을 통해 확인해보면 `getBoundingClientRect()`호출하는 과정에서 Recalculate Style, 리플로우 현상이 발생하는 것을 확인할 수 있습니다.

![addEventListener scroll 예시](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/4/scroll-event-example.png)

## Intersection Observer API의 등장

[Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)(교차 관찰자 API)를 사용하면 위와 같은 문제를 해결할 수 있습니다. 비동기적으로 실행되기 때문에 메인 스레드에 영향을 주지 않으면서 변경 사항을 관찰할 수 있습니다. 또한 `IntersectionObserverEntry` 내부의 `rootBounds`를 통해 `getBoundingClientRect()`를 호출한 것과 같은 결과를 알 수 있기 때문에 따로 `getBoundingClientRect()` 함수를 호출할 필요가 없어 리플로우 현상을 방지할 수 있습니다.

아래는 위에 예시와 같은 동작을 하지만 `Intersection Observer API`를 사용해서 구현한 예제입니다.

<iframe width="100%" height="400" src="//jsfiddle.net/hyeyoon/og319zw6/6/embedded/result,js,html,css" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

```javascript
// IntersectionObserver 를 등록한다.
const io = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    // 관찰 대상이 viewport 안에 들어온 경우 'tada' 클래스를 추가
    if (entry.intersectionRatio > 0) {
      entry.target.classList.add('tada');
    }
    // 그 외의 경우 'tada' 클래스 제거
    else {
      entry.target.classList.remove('tada');
    }
  })
})

// 관찰할 대상을 선언하고, 해당 속성을 관찰시킨다.
const boxElList = document.querySelectorAll('.box');
boxElList.forEach((el) => {
  io.observe(el);
})
```

이 코드 또한 개발자 도구의 Performance 탭을 통해 확인해보면, 이 코드는 위의 예제와 달리 리플로우 현상이 발생하지 않는 것을 확인할 수 있습니다.

![intersection observer 예시](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/4/intersectionobserver-example.png)

## Intersection Observer 사용 방법
Intersection Observer의 사용법에 대해 알아보겠습니다. [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#)에서는 IntersectionObserver를 아래와 같이 정의하고 있습니다.
> The Intersection Observer API provides a way to asynchronously observe changes in the intersection of a target element with an ancestor element or with a top-level document's viewport.

> Intersection Observer API는 타겟 엘리먼트가 조상 엘리먼트, 또는 최상위 문서의 뷰포트(브라우저에서는 보통 브라우저의 viewport)의 교차영역에서 발생하는 변화를 비동기로 관찰하는 빙법을 제공합니다.

기본 사용 방법은 아래와 같습니다. 여러 엘리먼트에 이벤트를 한 번에 등록하고 싶다면 콜백함수에 forEach를 사용해서 선언할 수 있습니다. [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/IntersectionObserver)를 생성하기 위해서는 교차되었을 때 실행할 callback함수를 등록해야 하고, 선택적으로 options 값을 넘겨줄 수 있습니다.
```javascript
// 기본구조는 콜백함수와 옵션을 받는다.
const io = new IntersectionObserver(callback[, options])
```
### Parameters
#### `callback`
- `callback`: 타겟 엘리먼트가 교차되었을 때 실행할 함수
  - `entries`: [IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) 객체의 리스트. 배열 형식으로 반환하기 때문에 forEach를 사용해서 처리를 하거나, 단일 타겟의 경우 배열인 점을 고려해서 코드를 작성해야 합니다.
  - `observer`: 콜백함수가 호출되는 IntersectionObserver

#### `options`
- `root`
  - default: `null`, 브라우저의 viewport
  - 교차 영역의 기준이 될 root 엘리먼트. observe의 대상으로 등록할 엘리먼트는 반드시 root의 하위 엘리먼트여야 합니다.

  ![root 예시](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/4/root.png)

- `rootMargin`
  - default: `'0px 0px 0px 0px'`
  - root 엘리먼트의 마진값. css에서 margin을 사용하는 방법으로 선언할 수 있고, 축약도 가능하다. px과 %로 표현할 수 있습니다. rootMargin 값에 따라 교차 영역이 확장 또는 축소된다.

  ![rootMargin 예시](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/4/rootmargin.png)

- `threshold`
  - default: `0`
  - 0.0부터 1.0 사이의 숫자 혹은 이 숫자들로 이루어진 배열로, 타겟 엘리먼트에 대한 교차 영역 비율을 의미합니다. 0.0의 경우 타겟 엘리먼트가 교차영역에 진입했을 시점에 observer를 실행하는 것을 의미하고, 1.0의 경우 타켓 엘리먼트 전체가 교차영역에 들어왔을 때 observer를 실행하는 것을 의미합니다.

  ![threshold 예시](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/4/threshold.png)

위의 설명을 바탕으로 실제 어떻게 사용하는지 예시를 한 번 보겠습니다.

```javascript
// options 설정
const options = {
  root: document.querySelector('.container'), // .container class를 가진 엘리먼트를 root로 설정. null일 경우 브라우저 viewport
  rootMargin: '10px', // rootMargin을 '10px 10px 10px 10px'로 설정
  threshold: [0, 0.5, 1] // 타겟 엘리먼트가 교차영역에 진입했을 때, 교차영역에 타켓 엘리먼트의 50%가 있을 때, 교차 영역에 타켓 엘리먼트의 100%가 있을 때 observe가 반응한다.
}

// IntersectionObserver 생성
const io = new IntersectionObserver((entries, observer) => {
  // IntersectionObserverEntry 객체 리스트와 observer 본인(self)를 받음
  // 동작을 원하는 것 작성
  entries.forEach(entry => {
    // entry와 observer 출력
    console.log('entry:', entry);
    console.log('observer:', observer);
  })
}, options)
```

### Methods
#### [`IntersectionObserver.observe(targetElement)`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/observe)
- 타겟 엘리먼트에 대한 IntersectionObserver를 등록할 때(관찰을 시작할 때) 사용합니다.

#### [`IntersectionObserver.unobserve(targetElement)`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/unobserve)
- 타겟 엘리먼트에 대한 관찰을 멈추고 싶을 때 사용하면 됩니다. 예를 들어 Lazy-loading(지연 로딩)을 할 때는 한 번 처리를 한 후에는 관찰을 멈춰도 됩니다. 이 경우에는 처리를 한 후 해당 엘리먼트에 대해 `unobserve(targetElement)`을 실행하면 이 엘리먼트에 대한 관찰만 멈출 수 있습니다.

#### [`IntersectionObserver.disconnect()`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/disconnect)
- 다수의 엘리먼트를 관찰하고 있을 때, 이에 대한 모든 관찰을 멈추고 싶을 때 사용하면 됩니다.

#### [`IntersectionObserver.takerecords()`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/takeRecords)
- `IntersectionObserverEntry` 객체의 배열을 리턴합니다.


## IntersectionObserverEntry 객체
위에서 IntersectionObserver에 대해 설명했을 때 IntersectionObserver에서 반환하는 callback은 IntersectionObserverEntry 객체의 배열을 반환한다고 했는데요. IntersectionObserver를 사용할 때 반환되는 이 객체의 정보는 어떤 동작을 등록하거나 할 때 유용하게 사용할 수 있습니다.

### Properties
#### 사각형의 크기를 반환하는 속성  
아래 세 가지 속성은 addEventListener를 설명할 때 언급했던 `Element.getBoundingClientRect()`를 실행한 것과 같은 결과를 반환합니다. `Element.getBoundingClientRect()` 함수의 경우 호출 시 리플로우(reflow) 현상이 나타나지만, 아래의 속성을 사용하면 리플로우 없이 정보를 알 수 있습니다.

- [`IntersectionObserverEntry.boundingClientRect`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/boundingClientRect): 타겟 엘리먼트의 정보를 반환합니다.
- [`IntersectionObserverEntry.rootBounds`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/rootBounds): root 엘리먼트의 정보를 반환합니다. root를 선언하지 않았을 경우 null을 반환합니다.
- [`IntersectionObserverEntry.intersectionRect`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/intersectionRect): 교차된 영역의 정보를 반환합니다.

![`intersectionobserverentry 예시`](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/4/intersectionobserverentry.png)

#### 유용한 정보를 반환하는 속성
- [`IntersectionObserverEntry.intersectionRatio`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/intersectionRatio): IntersectionObserver 생성자의 options의 threshold와 비슷합니다. 교차 영역에 타겟 엘리먼트가 얼마나 교차되어 있는지(비율)에 대한 정보를 반환합니다. threshold와 같이 0.0부터 1.0 사이의 값을 반환합니다.
- [`IntersectionObserverEntry.isIntersecting`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/isIntersecting): 타겟 엘리먼트가 교차 영역에 있는 동안 true를 반환하고, 그 외의 경우 false를 반환합니다.  
- [`IntersectionObserverEntry.target`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/target): 타겟 엘리먼트를 반환합니다.
- [`IntersectionObserverEntry.time`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/time): 교차가 기록된 시간을 반환합니다.

## Lazy-Loading 예제

이제 IntersectionObserver API에 대해 알아본 것을 바탕으로 이를 활용한 Lazy-Loading 예제를 만들어보겠습니다.

```html
<div class="example">
  <h1 class="title">Scroll Down <span class="arrow">👇</span></h1>
  <img src="https://picsum.photos/600/400/?random?0" alt="random image" class="image-default">
  <img data-src="https://picsum.photos/600/400/?random?1" alt="random image" class="image">
  <img data-src="https://picsum.photos/600/400/?random?2" alt="random image" class="image">
  <img data-src="https://picsum.photos/600/400/?random?3" alt="random image" class="image">
  <img data-src="https://picsum.photos/600/400/?random?4" alt="random image" class="image">
  <img data-src="https://picsum.photos/600/400/?random?5" alt="random image" class="image">
  <img data-src="https://picsum.photos/600/400/?random?6" alt="random image" class="image">
  <img data-src="https://picsum.photos/600/400/?random?7" alt="random image" class="image">
</div>
```

html의 경우 기본으로 불러올 `img` 하나를 제외하고 나머지 속성은 `src` 대신 `data-src` 속성에 이미지 주소를 선언했습니다.

> 일반적으로 `<img>` 태그는 `src` 속성을 만나면 이미지 소스를 내려받지만, 예시에서는 `src` 속성 대신에 `data-src`에 이미지 주소를 넣고 타겟 이미지가 교차 영역에 진입했을 때 타겟의 `src`에 `data-src`를 설정해주도록 했습니다.

```javascript
// IntersectionObserver의 options를 설정합니다.
const options = {
  root: null,
  // 타겟 이미지 접근 전 이미지를 불러오기 위해 rootMargin을 설정했습니다.
  rootMargin: '0px 0px 30px 0px',
  threshold: 0
}

// IntersectionObserver 를 등록한다.
const io = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    // 관찰 대상이 viewport 안에 들어온 경우 image 로드
    if (entry.isIntersecting) {
      // data-src 정보를 타켓의 src 속성에 설정
      entry.target.src = entry.target.dataset.src;
      // 이미지를 불러왔다면 타켓 엘리먼트에 대한 관찰을 멈춘다.
      observer.unobserve(entry.target);
    }
  })
}, options)

// 관찰할 대상을 선언하고, 해당 속성을 관찰시킨다.
const images = document.querySelectorAll('.image');
images.forEach((el) => {
  io.observe(el);
})
```
자바스크립트의 경우 위와 같이 구현합니다. css의 경우 아래 jsfiddle 예제의 css 탭을 참고하시면 될 것 같습니다. 아래는 위의 코드를 구현해놓은 예제입니다. 스크롤을 내려보면, 스크롤이 해당 이미지의 위치에 도달했을 때 이미지를 로딩하고 있습니다.

<iframe width="100%" height="400" src="//jsfiddle.net/hyeyoon/3yzoufem/32/embedded/result,js,html,css" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

크롬 개발자 도구의 Network 탭을 보면 순차적으로 이미지를 불러오는 것을 확인할 수 있습니다.

![lazyloading 예시](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/4/lazy-loading.png)

## 마무리
이상으로 IntersectionObserver API에 대해 알아봤습니다. IntersectionObserver를 활용하면 기존에 사용하고 있던 scroll 이벤트 중 IntersectionObserver로 구현할 수 있는 경우에는 기존 코드 대비 성능을 개선할 수 있고, 그 외에도 웹에 게시된 광고의 수익을 활용하는 등 다양한 영역에 활용할 수 있을 것 같습니다. 아직 Safari에서 지원을 하지 않는다는 아쉬움이 있지만, [polyfill](https://github.com/w3c/IntersectionObserver/tree/master/polyfill)을 제공하고 있기 때문에 한 번 사용해보는 것도 좋을 것 같습니다.

## 참고문헌
* [IntersectionObserver API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#)
* [IntersectionObserver’s Coming into View](https://developers.google.com/web/updates/2016/04/intersectionobserver)
* [Now You See Me: How To Defer, Lazy-Load And Act With IntersectionObserver](https://www.smashingmagazine.com/2018/01/deferring-lazy-loading-intersection-observer-api/?utm_source=frontendfocus&utm_medium=email)
* [너는 나를 본다: 지연 방법, 레이지 로드와 IntersectionObserver의 동작](https://github.com/codepink/codepink.github.com/wiki/%EB%84%88%EB%8A%94-%EB%82%98%EB%A5%BC-%EB%B3%B8%EB%8B%A4:-%EC%A7%80%EC%97%B0-%EB%B0%A9%EB%B2%95,-%EB%A0%88%EC%9D%B4%EC%A7%80-%EB%A1%9C%EB%93%9C%EC%99%80-IntersectionObserver%EC%9D%98-%EB%8F%99%EC%9E%91)
* [IntersectionObserver를 이용한 이미지 동적 로딩 기능 개선](https://tech.lezhin.com/2017/07/13/intersectionobserver-overview)
