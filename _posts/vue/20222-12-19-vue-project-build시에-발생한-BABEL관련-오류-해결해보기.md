---
layout: post
title: vue project build시에 발생한 BABEL관련 오류 해결해보기.md
tags: [vue]
skills: [vue]
author: mainmethod0126
excerpt_separator: <!--more-->
---

# 서론

회사에서 포지션은 백엔드 개발자지만, 급하게 vue2로 만들어진 프론트 엔드 프로젝트를 빌드할 상황이 있어서 빌드를 하였는데, 여러개의 WARING이 발견되어 해당 경고가 왜 발생했는지와 해결방법을 알아보았습니다.

## 발생한 경고

### 첫번째 경고

```js
-  Building for production...
[BABEL] Note: The code generator has deoptimised the styling of /home/vsts/work/1/s/node_modules/vuetify/dist/vuetify.js as it exceeds the max of 500KB.
 WARNING  Compiled with 4 warnings7:51:44 AM
```

위 경고가 가장 첫번째 발생한 경고입니다.

가장 먼저 눈에 띄이는 키워드는 `BABEL` 과 `MAX OF 500KB` 두 키워드인데 하나는 딱 봐도 용량 관련된 내용이고,  `BABEL`이라는 키워드는 너무 낯선 단어입니다 오류를 파악하기 위해서는 Babel 어떤 친구인지 대략적으로라도 알아야할 것 같습니다.

#### BABEL

`Babel`은 최신 JavaScript 코드가 `구버전의 브라우저`에서도 동작할 수 있도록 컴파일해주는 `컴파일러`입니다.

예를 들어 최신 버전의 JavaScript(ECMAScript 2015 또는 ES6)의 기능인 `클래스 구문`을 사용하는 다음 JavaScript 코드가 있다고 가정합니다.

```js
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

```

이 코드는 `클래스 구문을 지원하지 않는 이전 브라우저`에서는 작동하지 않습니다.

이러한 환경에서 작동하도록 하기 위해 Babel을 사용하여 이전 브라우저에서 작동하는 버전으로 코드를 변환할 수 있습니다.

이렇게 하려면 먼저 Babel과 필요한 사전 설정 또는 플러그인을 설치해야 합니다.

그런 다음 Babel 명령줄 도구를 사용하여 코드를 트랜스파일할 수 있습니다.

```js
babel src/rectangle.js --out-file dist/rectangle.js
```

이렇게 하면 원본 코드의 트랜스파일된 버전이 포함된 새 파일 `dist/rectangle.js` 가 생성됩니다

다음은 트랜스파일된 코드의 예입니다.

```js
"use strict";

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

function _defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } }

function _createClass(Constructor, protoProps, staticProps) { if (protoProps) _defineProperties(Constructor.prototype, protoProps); if (staticProps) _defineProperties(Constructor, staticProps); return Constructor; }

var Rectangle = /*#__PURE__*/function () {
  function Rectangle(width, height) {
    _classCallCheck(this, Rectangle);

    this.width = width;
    this.height = height;
  }

  _createClass(Rectangle, [{
    key: "getArea",
    value: function getArea() {
      return this.width * this.height;
    }
  }]);

  return Rectangle;
}();
```

자 이제 Babel이라는 친구가 대략적으로 어떤 친구인지 알게되었습니다.

그럼 다시 한번 경고 구문을 보겠습니다.

```js
The code generator has deoptimised the styling of /home/vsts/work/1/s/node_modules/vuetify/dist/vuetify.js as it exceeds the max of 500KB.
```

> "vuetify.js 가 500KB를 초과해서 스타일 최적화를 진행하지 않았어."

이제 무슨 말인지 이해가 좀 됩니다. Babel을 통하여 최신 문법의 js를 구버전 브라우저에서 사용할 수 있도록 최적화를 진행해야하는데, vuetify.js라는 특정 파일의 용량이 크기 때문에 이를 진행하지 않았다. 라는 말인 것 같습니다.

vuetify.js 는 구브라우저에 호환되는 코드가아닌 기존의 문법을 그대로 간직하기 때문에 구브라우저에서는 오류를 발생할 가능성이 높아집니다.

그럼 해결 방법을 생각해 봅시다.

### 해결 방법

단편적으로 위 문제를 해결할 방법은 아래와 같습니다.

1. 500kb 라는 `용량 제한을 더 높이거나 아예 제한을 없앤다`.
2. vuetify.js 의 `코드를 줄여 500kb 이하`로 만든다.

자 두 방법 다 확인해 보도록 하겠습니다.

#### BABEL 용량 제한 변경

자 우리의 vuetify.js 가 `500kb`라는 제한을 넘었으니 문제가 발생한거니 `500kb` 를 2배인 `1000kb` 늘려 봅시다. (용량 제한으로 해결되는 것을 확인하는 것이 우선이기 때문에 일단 무지성으로 변경해봅니다)

진행하기 전에 잠깐! `주의점`이 존재합니다. 바로 `컴파일 시간의 증가`입니다.

용량이 큰 파일이라하면 분명 더 많은 코드를 포함하고 있을 것이고 이 많은 코드를 컴파일 하기 위해서 분명 `컴파일 시간이 증가`하는 이슈가 발생할 것 입니다.

근본적인 해결은 소스 코드를 줄이는 것 입니다.

소스코드의 용량이 크다는 것은 분명 불필요한 코드 또는 모듈화되지 않은 코드, 중복되는 코드가 존재할 가능성이 높다는 것이기 때문에 이를 충분히 검토 후 진행하실 것을 추천드립니다.

babel 용량 제한을 변경하기 위해서는 `babel.config.js` 파일을 수정해야합니다.

아래는 현재 사용되고있는 `babel.config.js` 파일입니다.

```js
module.exports = {
  presets: [
    ['@vue/app', { useBuiltIns: 'entry' }],
  ],
}
```

위 설정파일에 크기 제한을 변경하는 설정을 추가하면 아래와 같이 변경됩니다.

```js
module.exports = {
  presets: [
    ['@vue/app', { useBuiltIns: 'entry' }],
  ],
  overrides: [
    {
      test: /node_modules\/vuetify\/dist\/vuetify.js/,
      // or test: /vuetify.js/, if vuetify.js is in your project's root directory
      options: {
        transpileOnly: true,
        // increase the size limit to a value larger than 500KB
        // for example: 1MB = 1000000, 2MB = 2000000, etc.
        // you can adjust this value as needed
        transpileSizeLimit: 1000000,
      },
    },
  ],
}
```

추가된 각각의 옵션들은 아래 의미를 갖습니다.

- overrides : 파일 또는 디렉토리에 각각 다른 babel 구성을 지정하도록 해줍니다.
- test : babel 에서 변환해야 하는 파일을 결정하는 정규식 또는 정규식 배열을 지정합니다.
- 







```js
chunk kiosk~ndaLogs [mini-css-extract-plugin]
Conflicting order. Following module has been added:
 * css ./node_modules/css-loader/dist/cjs.js??ref--9-oneOf-1-1!./node_modules/vue-loader/lib/loaders/stylePostLoader.js!./node_modules/postcss-loader/src??ref--9-oneOf-1-2!./node_modules/sass-loader/dist/cjs.js??ref--9-oneOf-1-3!./node_modules/cache-loader/dist/cjs.js??ref--1-0!./node_modules/vue-loader/lib??vue-loader-options!./src/components/modal/Paging.vue?vue&type=style&index=0&id=7d6372a3&prod&lang=scss&scoped=true&
despite it was not able to fulfill desired ordering with these modules:
 * css ./node_modules/css-loader/dist/cjs.js??ref--9-oneOf-1-1!./node_modules/vue-loader/lib/loaders/stylePostLoader.js!./node_modules/postcss-loader/src??ref--9-oneOf-1-2!./node_modules/sass-loader/dist/cjs.js??ref--9-oneOf-1-3!./node_modules/cache-loader/dist/cjs.js??ref--1-0!./node_modules/vue-loader/lib??vue-loader-options!./src/components/modal/Gallery.vue?vue&type=style&index=0&id=bf858768&prod&lang=scss&scoped=true&
   - couldn't fulfill desired order of chunk group(s) ndaLogs
   - while fulfilling desired order of chunk group(s) kiosk
```

위 경고는 