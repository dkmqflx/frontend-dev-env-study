## 3. 플러그인

- 기본적으로 바벨은 코드를 받아서 코드를 반환한다.

- 바벨 함수를 정의한다면 이런 모습이 될 것이다.

```js
const babel = (code) => code;
```

- 바벨은 파싱과 출력만 담당하고 변환 작업은 다른 녀석이 처리하는데 이것을 "플러그인" 이라고 부른다.

### 3.1 커스텀 플러그인

- 플러그인을 직접 만들면서 동작이 원리를 살펴 보겠다.

- myplugin.js 라는 파일을 아래처럼 만들어 보자 (출처: [바벨 홈페이지의 예제 코드](https://babeljs.io/docs/plugins#plugin-development)).

```js
// myplugin.js

module.exports = function myplugin() {
  // 커스텀 플러그인을 만들 때는 visitor라는 객체를 리턴해주어야 한다
  return {
    visitor: {
      Identifier(path) {
        // path라는 객체를 받는다
        const name = path.node.name; // path.node.name으로 파싱된 결과물에 접근할 수 있다

        // 바벨이 만든 AST 노드를 출력한다
        console.log("Identifier() name:", name);

        // 변환작업: 코드 문자열을 역순으로 변환한다
        path.node.name = name.split("").reverse().join("");
      },
    },
  };
};
```

- 플러그인 형식은 visitor 객체를 가진 함수를 반환해야 한다.

- 이 객체는 바벨이 파싱하여 만든 추상 구문 트리(AST)에 접근할 수 있는 메소드를 제공한다.

- 그중 Identifier() 메소드의 동작 원리를 살펴보는 코드다.

- 플러그인 사용법을 알아보자.

```shell
npx babel --help # babel help문서를 본다

# 문서를 보면 아래처럼 플러그인을 사용하는 것을 확인할 수 있다
--plugins [list] A comma-separated list of plugin names.

```

- `--plugins` 옵션에 플러그인을 추가하면 된다.

```shell
npx babel app.js --plugins ./myplugin.js # 플러그인 추가

# 아래처럼 로그가 출력되고
Identifier() name: alert
Identifier() name: msg
Identifier() name: window
Identifier() name: alert
Identifier() name: msg

# 아래처럼 코드 문자열이 역순으로 변환된 코드를 출력한다
const trela = gsm => wodniw.trela(gsm);

```

- Identifier() 메소드로 들어온 인자 path에 접근하면 코드 조각에 접근할 수 있는 것 같다.

- path.node.name의 값을 변경하는데 문자를 뒤집는 코드다.

- 결과의 마지막 줄에서 보는것 처럼 이 코드의 문자열 순서가 역전되었다.

<br/>

- 우리가 하려는것은 ECMASCript2015로 작성한 코드를 인터넷 익스플로러에서 돌리는 것이다.

- 먼저 const 코드를 var로 변경하는 플러그인을 만들어 보겠다.

```js
// myplugin.js:
module.exports = function myplugin() {
  return {
    visitor: {
      // https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-block-scoping/src/index.js#L26
      // VariableDeclaration 메소드도 파싱된 결과를 path라는 객체로 전달받는다
      VariableDeclaration(path) {
        console.log("VariableDeclaration() kind:", path.node.kind); // const가 출력된다

        if (path.node.kind === "const") {
          path.node.kind = "var";
        }
      },
    },
  };
};
```

- 이번에는 vistor 객체에 VariableDeclaration() 메소드를 정의했다.

- path에 접근해 보면 키워드가 잡히는 걸 알 수 있다.

- path.node.kind가 const 일 경우 var로 변환하는 코드다.

- 이 플러그인으로 다시 빌드해보자.

```shell

npx babel app.js --plugins ./myplugin.js

VariableDeclaration() kind: const

var alert = msg => window.alert(msg);

```

- 마지막 줄에 보면 const 코드가 var로 변경되었다.

### 3.2 플러그인 사용하기

- 위에서 만든 const를 var로 변경하는 것 같은 결과를 만드는 것이 block-scoping 플러그인이다.

- const, let 처럼 블록 스코핑을 따르는 예약어를 함수 스코핑을 사용하는 var 변경한다.

- NPM 패키지로 제공하는 플러그인을 설치하고,

```shell

npm install -D @babel/plugin-transform-block-scoping

```

- 설치한 플러그인을 사용해보면,

```shell

npx babel app.js --plugins @babel/plugin-transform-block-scoping

var alert = msg => window.alert(msg);

```

- 커스텀 플러그인과 같은 결과를 보인다.

- 인터넷 익스플로러는 위의 화살표 함수도 지원하지 않기 때문에 익스플로러에서도 해당 코드를 실행하기 위해서는 [arrow-functions](https://babeljs.io/docs/babel-plugin-transform-arrow-functions) 플러그인을 이용해서 일반 함수로 변경해주어야 한다

```shell

npm install -D @babel/plugin-transform-arrow-functions

npx babel app.js \
  --plugins @babel/plugin-transform-block-scoping \
  --plugins @babel/plugin-transform-arrow-functions

var alert = function (msg) {
  return window.alert(msg);
};


```

- 출력된 결과를 보면 var 키워드, 일반 함수로 변경된 것을 확인할 수 있다

<br/>

- ECMAScript5에서부터 지원하는 엄격 모드를 사용하는 것이 안전하기 때문에 "use strict" 구문을 추가해야 겠다.

- strict-mode 플러그인을 사용하자.

- 우선 아래 명령어를 통해 플러그인을 설치해준다

```shell

npm i -D @babel/plugin-transform-strict-mode

```

- 그리고 아래처럼 명령어를 실행하면 상단에 "use strict" 구문이 추가되어 엄격모드가 활성화 된 것을 확인할 수 있다

```shell
npx babel app.js \
  --plugins @babel/plugin-transform-block-scoping \
  --plugins @babel/plugin-transform-arrow-functions \
  --plugins @babel/plugin-transform-strict-mode

"use strict";

var alert = function (msg) {
  return window.alert(msg);
};
```

- 커맨드라인 명령어가 점점 길어지기 때문에 설정 파일로 분리하는 것이 낫겠다.

- 웹팩 webpack.config.js를 기본 설정파일로 사용하듯 바벨도 [babel.config.js](https://babeljs.io/docs/config-files#project-wide-configuration)를 사용한다.

- 프로젝트 루트에 babel.config.js 파일을 아래와 같이 작성하자.

```js
// babel.config.js

module.exports = {
  plugins: [
    "@babel/plugin-transform-block-scoping",
    "@babel/plugin-transform-arrow-functions",
    "@babel/plugin-transform-strict-mode",
  ],
};
```

- 커맨드라인에서 사용한 block-scoping, arrow-functions 플러그인을 설정 파일로 옮겼는데 plugins 배열에 추가하는 방식이다.

- strict-mode 플러그인을 마지막 줄에 추가했다.

- 다시 빌드해보자.

```shell
npx babel app.js

"use strict";

var alert = function (msg) {
  return window.alert(msg);
};
```

- 상단에 "use strict" 구문이 추가되어 엄격모드가 활성화된 것을 확인할 수 있다

- 이제야 비로소 인터넷 익스플로러에서 안전하게 동작하는 코드로 트랜스파일하였다.

- 이처럼 변환을 위한 플러그인 목록은 공식 문서의 Plugins 페이지에서 확인할 수 있다.
