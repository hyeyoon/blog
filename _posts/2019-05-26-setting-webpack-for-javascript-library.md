---
layout: post
title: '자바스크립트 라이브러리 배포를 위한 웹팩(webpack) 설정 경험기'
tags: [javascript]
description: >
  [dopyo.js](https://github.com/hyeyoon/dopyo.js)를 개발하면서, 배포를 위한 웹팩(webpack)을 설정 경험을 공유하는 글입니다.
---

평소 데이터 시각화에 관심과 svg를 공부해보고 싶어 사이드 프로젝트로 자바스크립트 차트 라이브러리인 [dopyo.js](https://github.com/hyeyoon/dopyo.js)를 개발하고 있습니다. 직접 자바스크립트 라이브러리를 만들고 배포까지 경험하는 것은 처음이라, 개발하는 과정을 블로그에 기록하려고 합니다. 포스트에 잘못된 부분이나 개선할 부분 등 피드백은 언제나 환영입니다.😄

## 웹팩(webpack)을 직접 설정하게 된 이유
최근에 자바스크립트 프레임워크, 라이브러리로 사용하고 있는 ```Vue```, ```React```, ```Angular```의 경우 boilerplate를 제공해주기 때문에 사용자가 직접 개발환경을 설정할 필요가 없습니다. 필자는 기존에 ```Vue```를 주로 사용해서 프로젝트를 진행했습니다. 그래서 ```Vue CLI```를 통해 ```Vue``` 개발환경을 설정하고 개발해왔기 때문에 개발할 때와 빌드할 때 생성되는 웹팩(webpack)설정을 사용하고, 따로 웹팩 설정을 직접 할 일은 드물었습니다. 그래서 사이드 프로젝트를 진행할 때는 직접 웹팩 설정해서 개발환경을 구축하겠다는 목표를 세웠습니다.

그리고\.\.\. 드디어 이번에 ```dopyo.js``` 프로젝트를 진행하며, 직접 웹팩 설정을 하게 되었습니다. 특히 순수 자바스크립트를 위한 웹 개발환경 설정의 경우에는 웹에서 튜토리얼을 찾기가 어렵지 않았지만, 자바스크립트 라이브러리 배포를 위한 튜토리얼은 찾기가 어려웠는데요. (좋은 튜토리얼이 있다면 피드백 남겨주시면 감사하겠습니다! 🙏) 이번 포스트를 통해 직접 경험한 라이브러리를 위하 웹팩 설정기(라고 쓰고 삽질기\.\.\.)를 소개합니다.

## 프로젝트 기본 구조 (Input)
일단 본격적으로 시작하기 전 ```dopyo.js```의 기본 폴더 구조는 아래와 같습니다. ```src``` 폴더 하위에 ```assets```와 ```js```로 css와 javascript 폴더가 구분되고, ```js```폴더 하위에 프로젝트의 메인 파일인 ```chart.js```가 있고 차트 관련 파일들이 있는 ```charts``` 폴더와 utility 함수들이 있는 ```utils``` 폴더로 구분되어 있습니다.
```
└── src
    ├── assets
    │   └── sass
    │       ├── base
    │       │   ├── _reset.scss
    │       ├── components
    │       │   └── chart.scss
    │       └── main.sass
    └── js
        ├── chart.js
        ├── charts
        │   ├── chartBasic.js
        │   ├── lineChart.js
        │   └── ...
        └── utils
            ├── calculate.js
            ├── helper.js
            └── variables.js
```

## 목표로 하는 웹팩 build 파일 구조 (Output)
웹팩을 통해 제가 빌드하고 싶은 폴더의 구조는 아래와 같습니다. 
- ```dist``` 폴더 하위에 ```css```와 ```javascript``` 빌드한 결과물을 저장할 것
- ```babel``` 컴파일을 통해 모던 브라우저에서 동작이 가능하게 할 것 
- cdn배포를 위해 minify한 css와 uglify한 javascript 파일을 제공할 것 

```
── dist
   ├── dopyo.css
   ├── dopyo.js
   ├── dopyo.min.css
   └── dopyo.min.js
```

## 웹팩(webpack) 설치
이제 본격적으로 웹팩을 시작해보겠습니다. 아래는 웹팩 공식 페이지에서 웹팩을 소개하는 이미지인데요. 웹팩은 자바스크립트를 위한 모듈 번들러로 아래 이미지의 왼쪽 부분과 같이 개발 시 작성된 다양한 javascript 파일들과 scss, css, image 등의 파일들을 이미지의 우측과 같은 형태로 번들링 해주는 자바스크립트 라이브러리입니다.

![webpack](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/5/webpack.png)

일단 웹팩을 설치 및 실행하기 위해서는 node 환경이 필요한데요. 이번 포스트에서는 npm으로 진행해보겠습니다. 

### 노드 프로젝트 생성
```package.json``` 파일이 이미 프로젝트 내부에 있는 경우 이 부분은 생략해도 됩니다.

```
npm init
```

### 웹팩 설치
웹팩4 부터는 webpack core와 webpack-cli 패키지가 분리되었습니다. 따라서 두 패키지를 각각 설치해야 합니다. webpack-cli webpack을 터미널에서 실행하기 위한 툴입니다.

```
npm install --save-dev webpack webpack-cli
```

### 웹팩 설정에 필요한 패키지 설치
아래 명령어를 터미널에 입력해서 패키지들을 설치합니다.
```
npm install --save-dev mini-css-extract-plugin webpack-merge clean-webpack-plugin terser-webpack-plugin optimize-css-assets-webpack-plugin
```

각 패키지가 ```dopyo.js```의 웹팩 설정 내에서 하는 역할은 아래와 같습니다.
- ```mini-css-extract-plugin```: css 파일을 따로 분리하기 위한 패키지
- ```webpack-merge```: 여러 개의 webpack 설정을 merge할 수 있는 패키지
- ```clean-webpack-plugin```: 폴더를 초기화할 때 사용하는 패키지
- ```terser-webpack-plugin```: javascript uglify할 때 사용하는 패키지
- ```optimize-css-assets-webpack-plugin```: css를 minify 할 때 사용하는 패키지

### 웹팩 config 파일 생성
프로젝트에서 사용할 ```webpack.prod.js```파일을 프로젝트 폴더 최상위에 생성합니다. 일반적인 웹팩 설정 파일은 ```webpack.config.js```파일 하나를 두고 사용할 수도 있는데, ```dopyo.js```에서는 개발 시 사용한 ```webpack.dev.js```가 따로 있기 때문에 배포를 위한 설정은 ```webpack.prod.js```에 작성했습니다.

## mode 설정
이제 단계별로 웹팩 설정을 직접 해보겠습니다. ```webpack.prod.js```파일에 아래와 같이 mode를 먼저 설정합니다. 배포를 위한 웹팩 설정을 진행 중이기 때문에 'production' 모드를 사용하겠습니다.

> 웹팩4에서는 기본적으로 none, development, production 세 가지를 제공하는데, development와 production에는 개발 중과 빌드 시 사용하는 옵션이 기본적으로 세팅되어 있어서 특별한 세부 설정 없이 사용할 수 있습니다. 자세한 내용은 [웹팩 공식 사이트 - mode]('https://webpack.js.org/configuration/mode')를 참고해주세요.

- webpack.prod.js

```javascript
module.exports = {
  mode: 'production'
}
```

## entry와 outputs 설정
모듈의 시작점을 설정하는 ```entry```와 번들할 파일을 어디에 저장하는지 설정하는 ```output```을 설정합니다. ```dopyo.js```의 최상위 모듈 파일은 ```src/js/chart.js```로 해당 파일 내부에 사용하는 ```scss``` 파일과 ```javascript``` 파일이 작성되어 있습니다.

### libraryTarget 설정
이 부분에서 가장 어려움을 겪었던 것은 ```libraryTarget``` 설정인데요. 일반적인 웹프로젝트 시 또는 npm 패키지만을 사용해서 개발할 때는 설정하지 않아도 됩니다. 하지만 이번에 ```dopyo.js```는 cdn 배포를 고려하고 있어서, script src를 통해서 링크로도 불러올 수 있어야 했는데요. 이 옵션 없이 웹팩 설정을 할 경우 다른 자바스크립트 파일에서 모듈로 빌드된 ```dopyo.js```를 사용할 수가 없었습니다. 그래서 모듈을 사용하지 않는 경우 글로벌 변수로 등록이 필요했습니다. (문제해결에 도움을 주신 Vahn 감사합니다. 🙏)

```libraryTarget``` 설정은 라이브러리를 내보내는 형식을 설정하는 옵션으로 여기서 사용한 ```umd```의 경우 AMD, CommonJS2로 내보내는 것입니다. ```libraryTarget```의 다른 옵션에 대해서는 [webpack output](https://webpack.js.org/configuration/output#outputlibrarytarget),  [webpack 설정 option에 대해서](https://trustyoo86.github.io/webpack/2018/01/10/webpack-configuration.html)를 참고해주세요.

- webpack.prod.js

```javascript
// path 사용 설정
const path = require('path')
module.exports = {
  mode: 'production',
  // 프로젝트의 최상위 모듈 파일을 시작점으로 설정
  entry: {
    // "dopyo"의 경우 빌드한 파일의 파일명으로 "app"과 같이 원하는 파일로 설정이 가능합니다.
    // 프로젝트 최상위 파일 경로를 작성합니다. "dopyo": './src/js/chart.js'와 같이 직접 경로를 넣을 수도 있습니다.
    "dopyo": path.resolve(__dirname, 'src/js/chart.js'),
  },
  // 빌드한 파일이 저장되는 곳
  output: {
    // 프로젝트 최상위 dist폴더로 경로 설정
    path: path.resolve(__dirname, 'dist'),
    // 번들 결과물의 파일 이름 설정
    // [name]은 위에 entry에서 설정한 파일의 이름(dopyo)입니다.
    filename: '[name].js',
    // assets이 있는 경우 해당 경로 설정해주는 옵션
    publicPath: '/',
    //
    libraryTarget: "umd",
  },
}
```

## module과 plugins 설정
다음으로 설정할 것은 module과 plugins 설정입니다. module은 번들링 하는 과정에서 해당 파일을 어떻게 로드할지를 설정합니다. 기본적으로는 ```rules``` 하위에 javascritp, css, image 등 각 파일 특성에 따른 loader를 설정해주는데요. 이 프로젝트에서는 javascript와 scss만 사용하고 있어 해당 부분에 대한 모듈을 설정했습니다. 그리고 plugins는 번들 된 결과물에 대해 어떻게 처리하는지 설정하는데, 여기서는 css파일 추출을 위해 사용을 설정했습니다.

### 모듈과 플러그인을 위한 패키지 설치

```
npm install --save-dev @babel/core @babel/plugin-syntax-dynamic-import @babel/preset-env babel-loader css-loader sass-loader  mini-css-extract-plugin
```

### 모듈, 플러그인 설정
javascript 파일에 대해서는 ```babel-loader```를 통해 자바스크립트 파일을 컴파일하게 설정했습니다. babel 옵션은 loader하위에 옵션으로 설정할 수도 있지만 ```dopyo.js```에서는 테스트 코드를 위한 바벨 설정이 따로 필요한 부분이 있어서 바벨 설정을 ```.babelrc```에 따로 분리해서 작성했습니다.

scss파일에 대해서는 번들링한 결과물에서 css파일을 따로 추출할 것이기 때문에 이때 사용하는 ```mini-css-extract-plugin``` 플러그인을 사용해서 옵션을 추가했습니다. 플러그인 설정에 대해서는 plugins 하위에 추가로 설정이 필요합니다.

- webpack.prod.js

```javascript
...
const MiniCssExtractPlugin = require("mini-css-extract-plugin")
module.exports = {
  ...
  module: {
    rules: [
      {
        // .js 확장자를 가진 파일에 대해 적용
        test: /\.js$/,
        // node_modules 폴더 제외
        exclude: /(node_modules)/,
        use: {
          // babel-loader 사용
          loader: 'babel-loader',
        }
      },
      {
        // .sass .scss 확장자를 가진 파일에 대해 적용
        test: /\.(sass|scss)$/,
        use: [
          // loader 사용 설정
          MiniCssExtractPlugin.loader,
          "css-loader",
          "sass-loader"
        ]
      }
    ]
  },
  plugins: [
    // MiniCssExtractPlugin 플러그인 설정
    // [name]은 entry에서 설정한 "dopyo"를 의미
    new MiniCssExtractPlugin({
      filename: "[name].css",
    })
  ]
}
```

## 자바스크립트 난독화(Uglify)와 css압축(minify) 파일 추출

```
webpack --config webpack.prod.js
```

여기까지 마친 상태에서 위의 명령어를 터미널에 입력해서 빌드를 진행해보면 ```dist``` 폴더 하위에 ```dopyo.js```, ```dopyo.css```이 생성됩니다.

![dopyo_dist](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/5/dopyo_dist.png)

하지만 js파일은 압축이 되어있고, css파일은 압축이 되어있지 않은 상태로 빌드가 됩니다.

- dopyo.js

```javascript
!function(t,n){if("object"==typeof exports&&"object"==typeof module)module.exports=n();else if("function"==typeof define&&define.amd)define([],n);else{var e=n();for(var r in e)("object"==typeof exports?exports:t)[r]=e[r]}}(window,function(){retu
...
```

- dopyo.css

```css
.container {
  box-sizing: border-box;
  width: 100%;
  padding: 20px; }

#chart {
  width: 100%; }
...
```

위에서 이야기했듯이 제가 원하는 결과물은 아래와 같이 압축을 하지 않은 것과 압축한 결과물 두 가지입니다.

```
── dist
   ├── dopyo.css
   ├── dopyo.js
   ├── dopyo.min.css
   └── dopyo.min.js
```

그래서 먼저 시도한 방법은 자바스크립트를 난독화한 버전과 난독화하지 않은 버전 두 가지로 만드는 것입니다.

### 웹팩으로 자바스크립트 난독화 파일 생성하기
일반적으로 웹팩에서 기본으로 제공하는 ```mode```의 ```production``` 옵션은 기본적으로 자바스크립트 결과물에 대해 난독화를 해줍니다. 하지만 이 프로젝트에서는 난독화와 난독화 하지 않고 babel 컴파일만 한 자바스크립트 두 파일이 필요합니다.

먼저 자바스크립트 난독화를 위한 플러그인을 설치합니다.
```
npm install --save-dev terser-webpack-plugin
```

그리고 기존 웹팩 설정을 이렇게 바꿔줍니다.

```javascript
...
// terser-webpack-plugin 플러그인 추가
const TerserJSPlugin = require('terser-webpack-plugin');
module.exports = {
  // entry 파일을 2가지로 분리
  entry: {
    "dopyo": path.resolve(__dirname, 'src/js/chart.js'),
    // 난독화할 파일에 대한 설정 추가 dopyo.min이 파일의 이름이 되도록 설정
    "dopyo.min": path.resolve(__dirname, 'src/js/chart.js'),
  },
  ...
  optimization: {
    minimize: true,
    // ~.min.js 파일에 .min.js가 있는 경우에만 난독화를 하도록 설정
    minimizer: [new TerserJSPlugin({
      include: /\.min\.js$/
    })]
  }
}
```

이렇게 작성해보니 ```dist```폴더를 보니 제가 원하는 구조로 설정이 완료된 것 같습니다.

![dopyo_dist2](https://raw.githubusercontent.com/hyeyoon/blog/master/public/img/5/dopyo_dist2.png)

하지만 파일의 내부를 살펴보니 js파일은 원하는대로 빌드가 됐지만, css는 아니었습니다. css 파일의 경우 두 파일이 다 압축되지 않은 상태였습니다.

- dopyo.js

```javascript
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	else {
		var a = factory();
		for(var i in a) (typeof exports === 'object' ? exports : root)[i] = a[i];
	}
})(window, function() {
...
```

- dopyo.min.js

```javascript
!function(t,n){if("object"==typeof exports&&"object"==typeof module)module.exports=n();else if("function"==typeof define&&define.amd)define([],n);else{var e=n();for(var r in e)("object"==typeof exports?exports:t)[r]=e[r]}}(window,function(){retu
...
```

- dopyo.css

```css
.container {
  box-sizing: border-box;
  width: 100%;
  padding: 20px; }

#chart {
  width: 100%; }
...
```

- dopyo.min.css

```css
.container {
  box-sizing: border-box;
  width: 100%;
  padding: 20px; }

#chart {
  width: 100%; }
...
```

### 웹팩으로 css파일 압축 파일 생성하기
css 파일의 경우 어떻게 압축한 버전과 압축하지 않은 버전이 있는지 찾아봤습니다. 하지만 웹팩 css 압축 플러그인인 ```optimize-css-assets-webpack-plugin```에서는 ```terser-webpack-plugin```와 같이 특정 파일만 압축을 진행할 수가 없었습니다. [css 파일을 multiple하게 ouput을 생성하고 싶다면 웹팩 설정을 각각 추가해서 진행할 것](https://github.com/NMFR/optimize-css-assets-webpack-plugin/issues/29#issuecomment-350505133)을 이야기했습니다. 그래서 일반 버전과 압축 버전 설정을 따로 추가했습니다.

css 압축을 위한 플러그인을 설치합니다.
```
npm install --save-dev optimize-css-assets-webpack-plugin
```

그리고 기존 웹팩 설정을 이렇게 바꿔줍니다.
```javascript
...
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')
module.exports = [
  // 일반 버전과 압축 버전 두 가지로 웹팩 빌드 설정
  {
    entry: {
      "dopyo": path.resolve(__dirname, 'src/js/chart.js'),
    },
    ...
    // 일반 버전의 경우 production모드에서 자바스크립트 난독화가 default값으로 진행되기 때문에 난독화를 false로 설정
    optimization: {
      minimize: false
    }
  },
  {
    entry: {
      "dopyo.min": path.resolve(__dirname, 'src/js/chart.js'),
    },
    ...
    // 압축 버전에서는 js와 css의 난독화를 진행하는 옵션을 추가
    optimization: {
      minimizer: [
        new TerserJSPlugin({}),
        new OptimizeCSSAssetsPlugin({}),
      ]
    },
  }
]
```

이렇게 수정하고 빌드를 하면 목표로 했던 모습의 파일이 빌드된 것을 확인할 수 있습니다.

- dopyo.js

```javascript
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	else {
		var a = factory();
		for(var i in a) (typeof exports === 'object' ? exports : root)[i] = a[i];
	}
})(window, function() {
...
```

- dopyo.min.js

```javascript
!function(t,n){if("object"==typeof exports&&"object"==typeof module)module.exports=n();else if("function"==typeof define&&define.amd)define([],n);else{var e=n();for(var r in e)("object"==typeof exports?exports:t)[r]=e[r]}}(window,function(){retu
...
```

- dopyo.css

```css
.container {
  box-sizing: border-box;
  width: 100%;
  padding: 20px; }

#chart {
  width: 100%; }
...
```

- dopyo.min.css

```css
.container{box-sizing:border-box;width:100%;padding:20px}#chart{width:100%}.chart{margin:0 auto}.title{text-align:center}.axis...
...
```

## 리팩토링
원하는 결과물을 얻을 수 있었지만, 웹팩 설정에서 일반적인 옵션과 압축을 위한 옵션에 mode나 module, plugin을 보면 겹치는 부분이 있습니다. 그래서 이렇게 겹치는 부분은 어떻게 효율적으로 개선할 수 있는지 알아봤습니다.

### 공통 설정 분리
웹팩에는 ```webpack-merge```라는 플러그인이 있는데, 보통 공통된 설정을 분리하고 추가한 설정을 함께 사용할 때 적용하는 플러그인입니다.

```
npm install --save-dev webpack-merge
```

플러그인을 설치 후, 공통으로 겹치는 부분을 ```webpack.common.js```로 분리하겠습니다.

- webpack.common.js

```javacript
const path = require('path')
const MiniCssExtractPlugin = require("mini-css-extract-plugin")
module.exports = {
  mode: 'production',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js',
    publicPath: '/',
    libraryTarget: "umd",
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules)/,
        use: {
          loader: 'babel-loader',
        }
      },
      {
        test: /\.(sass|scss)$/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          "sass-loader"
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name].css",
    })
  ]
}
```

그리고 추가로 따로 변경 옵션이 필요한 부분만 남겨서 ```webpack.prod.js```파일을 수정하겠습니다. 이 때 ```clean-webpack-plugin``` 플러그인을 추가로 설치했는데요. 이 플러그인은 기존에 빌드를 진행할 때 덮어씌워 지는 ```dist``` 폴더를 빌드를 진행할 때 폴더의 기존 파일을 삭제시키는 역할을 합니다. 혹시 모를 충돌 상황을 대비했습니다. 최종으로 설정을 끝낸 ```webpack.prod.js```파일입니다. 

```javascript
const path = require('path')
// webpack-merge 플러그인 추가
const merge = require('webpack-merge')
// webpack 공통 설정 추가 
const common = require('./webpack.common.js')
// clean-webpack-plugin 추가 
const CleanWebpackPlugin = require('clean-webpack-plugin')
const TerserJSPlugin = require('terser-webpack-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')

module.exports = [
  merge(common, {
    entry: {
      "dopyo": path.resolve(__dirname, 'src/js/chart.js'),
    },
    // 웹팩 빌드를 시작할 때 dist폴더를 비우도록 설정
    plugins: [
      new CleanWebpackPlugin(['dist']),
    ],
    optimization: {
      minimize: false
    }
  }),
  merge(common, {
    entry: {
      "dopyo.min": path.resolve(__dirname, 'src/js/chart.js'),
    },
    optimization: {
      minimizer: [
        new TerserJSPlugin({}),
        new OptimizeCSSAssetsPlugin({}),
      ]
    },
  })
]
```

그리고 마지막으로 터미널에서 ```package.json``` 파일에 아래의 scripts를 추가하면, 이제 터미널에서 ```npm run build``` 명령어를 입력하면 프로젝트 빌드를 실행하도록 설정합니다.

- package.json 

```
...
"scripts": {
  "build": "webpack --config webpack.prod.js"
},
...
```

## 느낀점
주니어 개발자로 웹팩을 처음 마주했을 때는 익숙하지 않은 문법들로 인해 선뜻 시작하지를 못했습니다. 이미 프레임워크와 라이브러리에서 기본 설정을 제공해주기 때문에 직접 설정할 필요를 느끼지 못해서 공부를 뒤로 미루곤 했습니다. 그래서 이번에 프로젝트를 진행하면서 웹팩 공부와 함께 직접 설정을 하는 것을 목표로 했습니다. 실제로 웹팩 학습과 코드를 작성하는 부분에서 시간이 걸렸고, 이 과정에서 느낀 경험기는 블로그에 꼭 남겨두고 싶어 이번 포스트를 작성했습니다. 중간중간 설명을 위한 코드를 넣다 보니 포스트가 많이 길어졌는데😅, 혹시 내용 중에 틀린 부분이나 개선할 점은 피드백을 주시면 바로 반영하겠습니다. 긴 글 읽어주셔서 감사합니다.🙏

## 참고문헌
* [webpack 공식 웹사이트](https://trustyoo86.github.io/webpack/2018/01/10/webpack-configuration.html)
* [webpack 설정 option에 대해서](https://trustyoo86.github.io/webpack/2018/01/10/webpack-configuration.html)
* [웹팩의 기본 개념](http://jeonghwan-kim.github.io/js/2017/05/15/webpack.html)
