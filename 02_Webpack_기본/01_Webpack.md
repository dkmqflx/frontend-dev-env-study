## 1. 배경

- 문법 수준에서 모듈을 지원하기 시작한 것은 ES2015부터다.

- import/export 구문이 없었던 모듈 이전 상황은 다음과 같다

- 아래와 같은 덧셈 함수에서

```js
// math.js

function sum(a, b) {
  return a + b;
} // 전역 공간에 sum이 노출
```

```js
// app.js

sum(1, 2); // 3
```

- 위 코드는 모두 하나의 HTML 파일 안에서 로딩해야만 실행된다.

- math.js가 로딩되면 app.js는 이름 공간에서 'sum'을 찾은 뒤 이 함수를 실행한다.

- 문제는 'sum'이 전역 공간에 노출된다는 것.

- 즉, 다른 파일에서도 'sum'이란 이름을 사용한다면 충돌한다.

### 1.1 IIFE 방식의 모듈

- 이러한 문제를 예방하기 위해 스코프를 사용한다.

- 함수 스코프를 만들어 외부에서 안으로 접근하지 못하도록 공간을 격리하는 것이다.

- 스코프 안에서는 자신만의 이름 공간이 존재하므로 스코프 외부와 이름 충돌을 막을 수 있다.

```js
// math.js

var math = math || {}; // math 네임스페이스

(function () {
  function sum(a, b) {
    return a + b;
  }
  math.sum = sum; // 네이스페이스에 추가
})();
```

- 같은 코드를 즉시실행함수로 감쌌기 때문에 다른 파일에서 이 안으로 접근할 수가 없다.

- 심지어 같은 파일일지라도 말이다. 자바스크립트 함수 스코프의 특징이다.

- 'sum'이란 이름은 즉시실행함수 안에 감추어졌기 때문에 외부에서는 같은 이름을 사용해도 괜찮다.

- 전역에 등록한 'math'라는 이름 공간만 잘 활용하면 된다.

### 1.2 다양한 모듈 스펙

- 이러한 방식으로 자바스크립트 모듈을 구현하는 대표적인 명세가 AMD와 CommonJS다.

- CommonJS 는 자바스크립트를 사용하는 모든 환경에서 모듈을 하는 것이 목표다.

- exports 키워드로 모듈을 만들고 require() 함수로 불러 들이는 방식이다.

- 대표적으로 서버 사이드 플랫폼인 Node.js에서 이를 사용한다.

```js
// math.js

exports function sum(a, b) { return a + b; }

```

```js
// app.js

const math = require("./math.js");
math.sum(1, 2); // 3
```

- AMD(Asynchronous Module Definition)는 비동기로 로딩되는 환경에서 모듈을 사용하는 것이 목표다. 주로 브라우져 환경이다.

- UMD(Universal Module Definition)는 AMD기반으로 CommonJS 방식까지 지원하는 통합 형태다.

- 이렇게 각 커뮤니티에서 각자의 스펙을 제안하다가 [ES2015](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)에서 표준 모듈 시스템 을 내 놓았다.

- 지금은 바벨과 웹팩을 이용해 모듈 시스템을 사용하는 것이 일반적이다.

- ES2015 모듈 시스템의 모습은 다음과 같다

```js
// math.js

export function sum(a, b) {
  return a + b;
}
```

```js
// app.js

import * as math from "./math.js";
math.sum(1, 2); // 3
```

- export 구문으로 모듈을 만들고 import 구문으로 가져올 수 있다.

### 1.3 브라우져의 모듈 지원

- 안타깝게도 모든 브라우져에서 모듈 시스템을 지원하지는 않는다.

- 인터넷 익스플로러를 포함한 몇 몇 브라우져에서는 여전히 모듈을 사용하지 못한다.

- 가장 많이 사용하는 크롬 브라우져만 잠시 살펴보자. (버전 61부터 모듈시스템을 지원 한다)

```html
<!-- index.html -->

<script type="module" src="app.js"></script>
```

`<script>` 태그로 로딩할 때 `type="text/javascript"` 대신 `type="module"`을 사용한다.

- app.js는 모듈을 사용할 수 있다.

- 그러나 브라우져에 무관하게 모듈을 사용하고 싶은데 이러한 문제를 웹팩을 통해서 해결할 수 있다

## 2. 엔트리/아웃풋

- 웹팩은 여러개 파일을 하나의 파일로 합쳐주는 번들러(bundler)다.

- 하나의 시작점(entry point)으로부터 의존적인 모듈을 전부 찾아내서 하나의 결과물을 만들어 낸다.

- app.js부터 시작해 math.js 파일을 찾은 뒤 하나의 파일로 만드는 방식이다.

- 간단히 웹팩으로 번들링 작업을 해보자.

- 번들 작업을 하는 webpack 패키지와 웹팩 터미널 도구인 webpack-cli를 설치한다.

```shell

$ npm install -D webpack webpack-cli

```

- webpack과 webpack-cli 설치 후 아래처럼 script를 추가해준다

```json
{
  "name": "lecture-frontend-dev-env",
  "version": "1.0.0",
  "description": "\"프론트엔드 개발 환경의 이해\" 강의 자료입니다.",

  "scripts": {
    "build": "webpack"
  },

  "devDependencies": {
    "webpack": "^4.41.5",
    "webpack-cli": "^3.3.10"
  }
}
```

- 설치 완료하면 `node_modules/.bin` 폴더에 실행 가능한 명령어가 몇 개 생긴다.

- webpack과 webpack-cli가 있는데 둘 중 하나를 실행하면 된다. `--help` 옵션으로 사용 방법을 확인해 보자.

```shell

$ node_modules/.bin/webpack --help

--mode Enable production optimizations or development hints.
[선택: "development", "production", "none"]
--entry The entry point(s) of the compilation. [문자열]
--output, -o The output path and file for compilation assets


```

- `--mode`, `--entry`, `--output` 세 개 옵션만 사용하면 코드를 묶을 수 있다.

```shell

$ node_modules/.bin/webpack --mode development --entry ./src/app.js --output dist/main.js

```

- `--mode`는 웹팩 실행 모드는 의미하는데 개발 버전인 development를 지정한다

- `--entry`는 시작점 경로를 지정하는 옵션이다

- `--output`은 번들링 결과물을 위치할 경로다

- 위 명령어를 실행하면 `dist/main.js`에 번들된 결과가 저장된다.

- 이 코드를 index.html에 로딩하면 번들링 전과 똑같은 결과를 만든다.

```html
<!-- index.html -->

<script src="dist/main.js"></script>
```

- 옵션 중 `--config` 항목을 보자.

```shell

$ node_modules/.bin/webpack --help

--config Path to the config file
[문자열] [기본: webpack.config.js or webpackfile.js]

```

- 이 옵션은 웹팩 설정파일의 경로를 지정할 수 있는데 기본 파일명이 webpack.config.js 혹은 webpackfile.js다.

- webpack.config.js 파일을 만들어 방금 터미널에서 사용한 옵션을 코드로 구성해 보자.

```js
// webpack.config.js

const path = require("path");

module.exports = {
  mode: "development",
  entry: {
    main: "./src/app.js",
  },
  output: {
    filename: "[name].js",
    // main이 output 파일의 name이 된다.
    // 이렇게 하는 이유는 entry가 main, main2와 같이 여러 개 인 경우,
    // 동적으로 main, main2가 name를 이름으로 갖는 파일을 생성할 수 있기 때문이다

    path: path.resolve("./dist"), // resolve 함수는 절대 경로를 계산해준다
  },
};
```

- webpack.config.js 파일에 common.js 모듈 시스템을 쓰는 이유

  - webpack.config.js는 **웹팩이 동작할 때 참고하는 설정 파일**이다.

  - 웹팩은 Node.js 위에서 동작하는 프로그램인데 Node.js가 사용하는 모듈시스템이 commonjs이기 때문이다.

- 터미널에서 사용한 옵션인 mode, entry, ouput을 설정한다.

- mode는 'development' 문자열을 사용했다.

- entry는 어플리케이션 진입점인 src/app.js로 설정한다.

- ouput에 설정한 `'[name]'`은 entry에 추가한 main이 문자열로 들어오는 방식이다.

- output.path는 절대 경로를 사용하기 때문에 path 모듈의 resolve() 함수를 사용해서 계산했다. (path는 노드 코어 모듈 중 하나로 경로를 처리하는 기능을 제공한다)

- 웹팩 실행을 위한 NPM 커스텀 명령어를 추가한다.

```json
// package.json

{
  "scripts": {
    "build": "./node_modules/.bin/webpack"
  }
}
```

- 모든 옵션을 웹팩 설정 파일로 옮겼기 때문에 단순히 webpack 명령어만 실행한다.

- 이제부터는 npm run build로 웹팩 작업을 지시할 수 있다.

- 이렇게 웹팩으로 번들링한 파일을 아래처럼 불러와서 사용할 수 있다

```html
<!DOCTYPE html>
<html>
  <head> </head>
  <body>
    <script type="text/javascript" src="dist/main.js"></script>
  </body>
</html>
```

- 아예 환경별로 다른 웹팩 설정 파일을 만들어서 사용할 수도 있다

```shell

webpack --config webpack.config.prod.js
webpack --config webpack.config.beta.js

```

`.env`를 사용하는 방법도 좋다.
