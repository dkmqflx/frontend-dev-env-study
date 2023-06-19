## 4. 프리셋

- ECMAScript2015+으로 코딩할 때 필요한 플러그인을 일일이 위에서처럼 설정하는 일은 무척 지난한 일이다.

- 코드 한 줄 작성하는데도 세 개 플러그인 세팅을 했으니 말이다.

- 목적에 맞게 여러가지 플러그인을 세트로 모아놓은 것을 "프리셋"이라고 한다.

### 4.1 커스텀 프리셋

- 사용한 세 개 플러그인을 하나의 프리셋으로 만들어 보겠다.

- 프로젝트 최상단에서 위치에서 mypreset.js 파일을 다음과 같이 작성하자.

```js
// mypreset.js
module.exports = function mypreset() {
  return {
    plugins: [
      "@babel/plugin-transform-arrow-functions",
      "@babel/plugin-transform-block-scoping",
      "@babel/plugin-transform-strict-mode",
    ],
  };
};
```

- plugins 배열에 사용한 세 개 플러그인을 담았는데, 이것이 플러그인을 모아놓은 프리셋이다.

- 프리셋을 사용하기 위해 바벨 설정을 아래처럼 preset에 위에서 설정한 파일을 불러오도록 수정한다.

```js
// babel.config.js
module.exports = {
  presets: ["./mypreset.js"],
};
```

- 플러그인 세팅 코드를 제거하고 presets에 방금 만든 mypreset.js를 추가했다.

- 실행해보면 프리셋에 있는 모든 플러그인이 실행되어 동일한 결과가 아래처럼 출력된다

```shell

npx babel app.js

"use strict";

var alert = function (msg) {
  return window.alert(msg);
};

```

### 4.2 프리셋 사용하기

- 이처럼 바벨은 목적에 따라 몇 가지 [프리셋](https://babeljs.io/docs/presets)을 제공한다.

  - preset-env

  - preset-flow

  - preset-react

  - preset-typescript

- preset-env는 ECMAScript2015+를 변환할 때 사용한다.

- 바벨 7 이전 버전에는 연도별로 각 프리셋을 제공했지만(babel-reset-es2015, babel-reset-es2016, babel-reset-es2017, babel-reset-latest) 지금은 env 하나로 합쳐졌다.

- preset-flow, preset-react, preset-typescript는 flow, 리액트, 타입스크립트를 변환하기 위한 프리셋이다.

- 인터넷 익스플로러 지원을 위해 env 프리셋을 사용해 보자.

- 먼저 패키지를 다운로드한다.

```shell

npm install -D @babel/preset-env

```

- 설치한 바벨 설정을 조금만 더 바꿔본다.

```js
// babel.config.js:
module.exports = {
  presets: ["@babel/preset-env"],
};
```

- 그리고 빌드하면,

```shell

npx babel app.js

"use strict";

var alert = function alert(msg) {
  return window.alert(msg);
};

```

- 우리가 만든 `mypreset.js`와 같은 결과를 출력한다.

## 5. env 프리셋 설정과 폴리필

- 과거에 제공했던 연도별 프리셋을 사용해 본 경험이 있다면 까다롭고 헷갈리는 설정 때문에 애를 먹었을지도 모르겠다.

- 그에 비해 env 프리셋은 무척 단순하고 직관적인 사용법을 제공한다.

### 5.1 타겟 브라우져

- 우리 코드가 크롬 최신 버전(2019년 12월 기준)만 지원하다고 하자.

- 그렇다면 인터넷 익스플로러를 위한 코드 변환은 불필요하다.

- `target` 옵션에 브라우져 버전명만 지정하면 env 프리셋은 이에 맞는 플러그인들을 찾아 최적의 코드를 출력해 낸다.

```js
// babel.config.js

module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: "79", // 크롬 79까지 지원하는 코드를 만든다
        },
      },
    ],
  ],
};
```

- 그리고 다시 바벨로 빌드해보면 아래와 같은 결과가 출력된다

```shell
npx babel app.js

"use strict";

const alert = msg => window.alert(msg);

```

- "use strict"는 추가되었지만 const와 화살표 함수가 그대로 유지된 채 출력되었다

- [Can I Use](https://caniuse.com/)에서 확인해보면

- 크롬은 블록 스코핑과 화살표 함수를 지원하기 때문에 코드를 변환하지 않고 이러한 결과물을 만들었다.

- 만약 인터넷 익스플로러도 지원해야 한다면 바벨 설정에 브라우져 정보만 하나 더 추가하면 된다.

```js
// babel.config.js :
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: "79",
          ie: "11", // ie 11까지 지원하는 코드를 만든다
        },
      },
    ],
  ],
};
```

- 그리고 다시 빌드해보면 ie와 chrome에서 모두 동작할 수 있는 아래와 같은 코드를 만들어준다

```shell

npx babel app.js

"use strict";

var alert = function alert(msg) {
  return window.alert(msg);
};

```

### 5.2 폴리필

- 이번엔 변환과 조금 다른 플리필에 대해 알아보자.

- ECMASCript2015의 Promise 객체를 사용하는 코드다.

```js
// app.js
new Promise();
```

- 바벨로 처리하면 어떤 결과가 나올까?

```shell

npx babel app.js

"use strict";

new Promise();

```

- env 프리셋으로 변환을 시도했지만 Promise 그대로 변함이 없다.

- 크롬에서는 Promise를 지원하기 때문에 정상적으로 실행되지만

- target에 ie 11도 설정하고 빌드한 했기 때문에 인터넷 익스플로러는 여전히 프로미스를 해석하지 못하고 아래와 같은 에러를 던진다.

  - `Promise이(가) 정의되지 않았습니다`

- 브라우져는 현재 스코프부터 시작해 전역까지 Promise라는 이름을 찾으려고 시도할 것이다.

- 그러나 스코프 어디에도 Promise란 이름이 없기 때문에 레퍼런스 에러를 발생하고 프로그램이 죽은 것이다.

- 플러그인이 프라미스를 ECMAScript5 버전으로 변환할 것으로 기대했는데 예상과 다르다.

- 바벨은 ECMAScript2015+(ES6+)를 ECMAScript5(ES5) 버전으로 변환할 수 있는 것만 빌드한다.

- 그렇지 못한 것들은 **"폴리필"**이라고 부르는 코드조각을 추가해서 해결한다.

- 가령 ECMAScript2015의 블록 스코핑은 ECMASCript5의 함수 스코핑으로 대체할 수 있다.

- 화살표 함수도 일반 함수로 대체할 수 있다.

- 이런 것들은 바벨이 변환해서 ECMAScript5 버전으로 결과물을 만든다.

- 한편 프라미스는 ECMAScript5 버전으로 대체할 수 없다.

- 다만 ECMAScript5 버전으로 구현할 수는 있다 (참고: [core-js promise](https://github.com/zloirock/core-js/blob/master/packages/core-js/modules/es.promise.js)).

- env 프리셋은 폴리필을 지정할 수 있는 옵션을 제공한다.

```js
// babel.config.js

module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        useBuiltIns: "usage", // 폴리필 사용 방식 지정
        corejs: {
          // 폴리필 버전 지정
          version: 2,
        },
      },
    ],
  ],
};
```

- `useBuiltIns`는 어떤 방식으로 폴리필을 사용할지 설정하는 옵션이다.

- "usage" , "entry", false 세 가지 값을 사용하는데 기본값이 false 이므로 폴리필이 동작하지 않았던 것이다.

- 반면 usage나 entry를 설정하면 폴리필 패키지 중 core-js를 모듈로 가져온다(이전에 사용하던 babel/polyfile은 바벨 7.4.0부터 사용하지 않음).

- corejs 모듈의 버전도 명시하는데 기본값은 2다. 버전 3과 차이는 확실히 잘 모르겠다.

- 이럴 땐 그냥 기본값을 사용하는 편이다.

- 자세한 폴리필 옵션은 바벨 문서의 [useBuiltIns](https://babeljs.io/docs/babel-preset-env#usebuiltins)와 [corejs](https://babeljs.io/docs/babel-preset-env#corejs) 섹션을 참고하자.

- 폴리필이 추가된 결과물을 확인해 보자.

```shell
npx babel src/app.js

"use strict";

require("core-js/modules/es6.promise");
require("core-js/modules/es6.object.to-string");

new Promise();

```

- 여전히 `new Promise();`가 존재하지만 core-js 패키지로부터 프라미스 모듈을 가져오는 임포트 구문이 상단에 추가되었다.

- 이제야 비로소 인터넷 익스플로러에서 안전하게 돌아가는 결과물을 만들었다.

- 실제 이 코드가 브라우저에서 안전하게 돌아가려면 core-js가 먼저 로드 되어 있어야하고 그 다음에 우리가 번들한 코드가 로드되어야 한다

## 6. 웹팩으로 통합

- 실무 환경에서는 바벨을 직접 사용하는 것보다는 웹팩으로 통합해서 사용하는 것이 일반적이다.

- 로더 형태로 제공하는데 [babel-loader](https://github.com/babel/babel-loader)가 그것이다.

- 즉, babel-loader가 babel을 실행하면 babel은 babel.config.js 파일을 보고 변환 작업을 하는 것이다.

- 먼저 패키지를 설치하고,

```shell

npm install -D babel-loader

```

- 웹팩 설정에 로더를 추가한다.

```js
// webpack.config.js

module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader", // 바벨 로더를 추가한다
        // babel.config.js에 정의한 설정이 적용된다
      },
    ],
  },
};
```

- `.js` 확장자로 끝나는 파일은 babel-loader가 처리하도록 설정했다.

- 사용하는 써드파티 라이브라리가 많을수록 바벨 로더가 느리게 동작할 수 있는데 node_modules 폴더를 로더가 처리하지 않도록 예외 처리했다([참고](https://github.com/babel/babel-loader#babel-loader-is-slow)).

- 폴리필 사용 설정을 했다면 core-js도 설치해야한다.

- 웹팩은 바벨 로더가 만든 아래 코드를 만나면 core-js를 찾을 것이기 때문이다.

  - 즉, core-js를 설치하지 않은 채 빌드하면 아래 module들을 찾을 수 없다는 에러 메세지가 출력딘다

```js
require("core-js/modules/es6.promise");
require("core-js/modules/es6.object.to-string");
```

- 버전 2로 패키지를 추가하자.

```shell

npm i core-js@2

```

- 그리고 웹팩으로 빌드하면,

```shell

npm run build

> webpack

Hash: a30cff5fbf53027423a0
Version: webpack 4.41.2
Time: 718ms
Built at: 2019. 12. 16. 오전 8:52:05
Asset Size Chunks Chunk Names
main.js 59.7 KiB main [emitted] main
Entrypoint main = main.js
[./src/app.js] 166 bytes {main} [built]

```

- 미리 등록해 놓은 NPM build 스크립트의 webpack 명령어가 동작한다.

- `./app.js`의 엔트리 포인트가 바벨 로더에 의해 빌드되고 결과물이 `dist/main.js`로 옮겨졌다.

```shell
cat ./dist/main.js | grep 'var alert' -A 5

var alert = function alert(msg) {
  return window.alert(msg);
};

new Promise();

```

- 웹팩으로 번들링되면서 변경된 부분 찾기가 어려울수 있는데 grep으로 변경되 부분만 확인했다.

## 7. 정리

- 바벨은 일관적인 방식으로 코딩하면서, 다양한 브라우져에서 돌아가는 어플리케이션을 만들기 위한 도구다.

- 바벨의 코어는 파싱과 출력만 담당하고 변환 작업은 플러그인이 처리한다.

- 여러 개의 플러그인들을 모아놓은 세트를 프리셋이라고 하는데 ECMAScript+ 환경은 env 프리셋을 사용한다.

- 바벨이 변환하지 못하는 코드는 폴리필이라 부르는 코드조각을 불러와 결과물에 로딩해서 해결한다.

- babel-loader로 웹팩과 함께 사용하면 훨씬 단순하고 자동화된 프론트엔드 개발환경을 갖출 수 있다.

## 바벨 실습

### 문제

- [webpack.config.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/2-babel/1-babel/webpack.config.js)

- [babel.config.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/2-babel/1-babel/babel.config.js)

### 해결

- 우선 아래처럼 바벨 로더를 설치해준다

```shell
npm i babel-loader
```

- 그리고 아래와 같이 webpack.config.js 파일의 코드를 수정해준다

```js
// webpack.config.js

module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader",
        /*
        하나의 로더만 설정할때는 loader와 options를 사용하고 두 개 이상 로더를 설정할 때는 use 옵션을 사용한다.
        https://webpack.js.org/configuration/#options

        use: [
        // apply multiple loaders and options instead, 여러개 로더와 옵션을 사용할 때 
        "htmllint-loader",
          {
            loader: "html-loader",
            options: {
          // ...
          }
        }
      ],
      */
      },
    ],
  },
};
```

- babel-loader는 babel을 실행하기 때문에 babel-core 도 설치해준다

```shell

npm i @babel/core

```

- 그리고나서 bable/core가 참조할 babel.config.js 파일을 만들고 필요한 라이브러리를 설치한 다음 아래처럼 코드를 작성해준다

```shell

npm i @babel/preset-env

```

```js
// babel.config.js

module.exports = {
  presets: [
    "@babel/preset-env",
    {
      targets: {
        ie: "11",
      },
    },
  ],
};
```

- 그리고 나서 `npm run build`로 빌드해주면 빌드에 성공한 것을 확인할 수 있고 main.js 파일이 dist 폴더에 있는 것을 확인할 수 있다.

- main.js 파일에서 const를 검색해도 const가 없는 것을 확인할 수 있다.

- 하지만 아직까지는 Promise는 검색이 되는데 ie에서 동작하도록 하기 위해서는 폴리필로 추가해주어야 한다.

```shell

npm i core-js@2

```

```js
// babel.config.js

module.exports = {
  presets: [
    "@babel/preset-env",
    {
      targetrs: {
        ie: "11",
      },
      useBuiltIns: "usage",
      corejs: {
        version: 2,
      },
    },
  ],
};
```

- 이렇게 설정해준 다음 빌드해주면 `regenerator-runtime/runtime` 에러가 발생하는데 그 이유는 `async/await`를 사용하는 코드가 있기 때문에 ie에서 에러가 발생하는 것이다

- 그렇기 때문에 아래처럼 `regenerator-runtime` 패키지를 설치해준다

```shell

npm i regenerator-runtime

```

- babel-loader가 babel을 실행하면 babel은 babel.config.js 파일을 보고 변환 작업을 하는데

- `async/await`의 경우에는 우리가 설정한 bable.config.js 파일의 corejs에서 처리할 수 없기 때문에 `regenerator-runtime`을 설치해준 것이다

- 그리고 `regenerator-runtime`은 corejs와 달리 별도로 babel.config.js 파일에 추가해주지 않아도 되는데

- 그 이유는 다음과 같다

  - 웹팩이 바벨을 실행하고, 바벨은 ie를 대상으로 코드 변환작업을 한다.

  - ie에서는 async/await이나 제네레이터를 지원하지 않기 때문에 폴리필을 사용하려고 할텐데

  - 이 때 `regenrator-runtime/runtime` 모듈을 가져오려고 할 것이다.

  - `regenrator-runtime` 패키지가 없는 상태에서는 `"Module not found: Error: Can't resolve 'regenerator-runtime/runtime' in ..."`와 같은 에러 메시지를 출력하기 때문에 node_modules 폴더 안에 이 패키지를 설치만 해주면 바벨이 모듈을 찾아주고 이러한 이유로 코드를 작성해 줄 필요가 없는 것이다

- 다만 현재 preset-env 버전에서는 `async/await `함수를 Promise를 사용한 코드로 변환해주기 때문에 Promise를 사용한 코드는 corejs를 통해서 변환할 수 있으니 `regenrator-runtime` 패키지를 설치하지 않아도 정상적으로 빌드가 된다

## Sass 실습

- Sass에는 Sass와 Scss 확장자가 있는데 Sass만 사용할 때는 Sass 확장자를, css 코드와 함께 사용하기 위해서는 Scss 확장자를 사용한다

### 문제

- [app.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/2-babel/2-sass/src/app.js)

- [webpack.config.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/2-babel/2-sass/webpack.config.js)

### 해결

- 아래처럼 sass-loader와 node-sass를 설치해주어야 한다

```shell

npm install sass-loader node-sass

# node-sass는 sass 코드를 css로 컴파일 해주는 역할을 한다

# sass-loader는 웹팩에서 sass 파일을 만나면 node-sass를 동작하도록 하는 역할

```

- [sass-loader 공식문서](https://github.com/webpack-contrib/sass-loader)를 보면 아래와 같이 사용다고 나와있다

```js
// webpack.config.js

module.exports = {
  module: {
    rules: [
      {
        test: /\.s[ac]ss$/i,
        use: [
          // Creates `style` nodes from JS strings, style-loader 가 html에 추가해준다
          "style-loader",

          // Translates CSS into CommonJS, 변환된 파일을 css-loader가 CommonJS 스타일의 js 코드로 변환해준다
          "css-loader",

          // Compiles Sass to CSS, sass-loader가 css파일로 변환해준다
          "sass-loader",
        ], // <-- 순서로 뒤에서 부터 실행된다
      },
    ],
  },
};
```

- 이전에 설정한 파일에서는 이미 style-loader와 css-loader를 사용하고 있으므로 아래처럼 sass-loader를 추가만 해주면 된다

```js
module.exports = {
  mode: "development",
  entry: {
    main: "./src/app.js",
  },
  output: {
    filename: "[name].js",
    path: path.resolve("./dist"),
  },
  module: {
    rules: [
      {
        test: /\.(scss|css)$/,
        use: [
          /**
           * TODO: SASS 코드를 사용할수 있겠끔 sass-loader를 구성하세요.
           */
          process.env.NODE_ENV === "production"
            ? MiniCssExtractPlugin.loader // 프로덕션 환경
            : "style-loader", // 개발 환경
          "css-loader",
          "sass-loader",
        ],
      },
    ],
  },
};
```

- 빌드한 후에 dist 폴더를 확인하면 sass 파일들의 코드들이 js 파일안에 있는 것을 확인할 수 있다
