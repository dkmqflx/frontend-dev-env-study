## 1. 배경

- 오래된 스웨터의 보푸라기 같은 것을 린트(Lint)라고 부른다.

- 보푸라기가 많으면 옷이 보기 좋지 않은데 코드에서도 이런 보프라기가 있다.

- 들여쓰기를 맞추지 않은 경우, 선언한 변수를 사용하지 않은 경우등

- 보프라기 있는 옷을 입을 수는 있듯이 이러한 코드로 만든 어플리케이션도 동작은 한다.

- 그러나 코드의 가독성이 떨어지고 점점 유지보수하기 어려운 애물단지가 되어버리기 일쑤다.

- 보푸라기를 제거하는 린트 롤러(Lint roller)처럼 코드의 오류나 버그, 스타일 따위를 점검하는 것을 린트(Lint) 혹은 린터(Linter)라고 부른다.

### 1.1 린트가 필요한 상황

- 아래 코드 유심히 보자.

- console.log() 함수를 실행하고 다음 줄에서 즉시 실행함수를 실행하려는 코드다.

```shell
console.log()
(function () {})()
```

- 하지만 이 코드를 브라우져에서 실행해 보면 TypeError가 발생한다.

- 브라우져는 코드에 세미콜론를 자동으로 넣는 과정(ASI)을 수행하는데,

- 위와 같은 경우는 우리의 의도대로 해석하지 못하고 아래 코드로 해석한다([Rules of Automatic Semicolon Insertion](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-rules-of-automatic-semicolon-insertion)을 참고).

```js
console.log()(function () {})();
```

- console.log()가 반환하는 값이 함수가 아니라 undefined인데 이 값으로 함수 호출을 시도했기 때문에 타입에러가 발생하는 것이다.

- 모든 문장에 세미콜론을 붙였다면, 혹은 즉시 함수호출 앞에 세미콜론을 붙였다면 예방할 수 있는 버그다.

- 린트는 코드의 가독성을 높이는 것 뿐만 아니라 동적 언어의 특성인 런타임 버그를 예방하는 역할도 한다.

## 2. ESLint

### 2.1 기본 개념

- ESLint는 ECMAScript 코드에서 문제점을 검사하고 일부는 더 나은 코드로 정정하는 린트 도구 중의 하나다.

- 코드의 가독성을 높이고 잠재적인 오류와 버그를 제거해 단단한 코드를 만드는 것이 목적이다.

- 과거 JSLint, JSHint에 이어서 최근에는 ESLint를 많이 사용하는 편이다.

- 코드에서 검사하는 항목을 크게 분류하면 아래 두 가지다.

  - 포맷팅

  - 코드 품질

- **포맷팅**은 일관된 코드 스타일을 유지하도록 하고 개발자로 하여금 쉽게 읽히는 코드를 만들어 준다.

- 이를 테면 "들여쓰기 규칙", "코드 라인의 최대 너비 규칙"을 따르는 코드가 가독성이 더 좋다.

- 한편, **코드 품질**은 어플리케이션의 잠재적인 오류나 버그를 예방하기 위함이다.

- 사용하지 않는 변수 쓰지 않기, 글로벌 스코프 함부로 다루지 않기 등이 오류 발생 확률을 줄여 준다.

- 린트 도구를 사용해서 코드를 검사하고 더 나아가 단단한 하고 읽기 좋은 코드를 만드는 방법을 알아보자.

### 2.2 설치 및 사용법

- 먼저 노드 패키지로 제공되는 ESLint 도구를 다운로드 한다.

```shell

npm i -D eslint

```

- 환경설정 파일을 프로젝트 최상단에 생성한다.

```js
// .eslintrc.js

module.exports = {};
```

- 빈 객체로 아무런 설정 없이 모듈만 만들었다.

- 아래와 같이 app.js 파일을 생성한 다음

```shell
console.log()
(function () {})()
```

- ESLint로 코드를 검사 하면

```shell

npx eslint app.js

```

- 아무런 결과를 출력하지 않고 프로그램을 종료한다.

### 2.3 규칙(Rules)

- ESLint는 검사 규칙을 미리 정해 놓았다. 문서의 [Rules](https://eslint.org/docs/latest/rules/) 메뉴에서 규칙 목록을 확인할 수 있다.

- 그렇기 때문에 rule을 추가해주어야만 코드를 검사할 수 있다

- 우리가 우려했던 문제와 관련된 규칙은 [no-unexpected-multiline](https://eslint.org/docs/latest/rules/no-unexpected-multiline)이다.

- 설정 파일의 rules 객체에 이 규칙을 추가한다.

```js
// .eslintrc.js

module.exports = {
  rules: {
    "no-unexpected-multiline": "error",
  },
};
```

- 규칙에 설정하는 값은 세 가지다.

  - "off"나 0은 끔,

  - "warn"이나 1은 경고,

  - "error"나 2는 오류.

- 설정한 규칙에 어긋나는 코드를 발견하면 오류를 출력하도록 했다.

- 다시 검사해 보자.

```shell

npx eslint app.js

2:1 error Unexpected newline between function and ( of function call no-unexpected-multiline

✖ 1 problem (1 error, 0 warnings)

```

- 예상대로 에러가 발생하고 코드 위치와 위반한 규칙명을 알려준다.

- 함수와 함수 호출의 괄호 `"("` 사이에 줄바꿈이 있는데 이것이 문제라고 한다.

- 코드 앞에 세미콜론을 넣거나 모든 문의 끝에 세미콜론을 넣어 문제를 해결할 수 있다.

- 수정한 다음 다시 검사하면 검사에 통과할 것이다.

### 2.4 자동으로 수정할 수 있는 규칙

- 자바스크립트 문장 뒤에 세미콜론을 여러 개 중복 입력해도 어플리케이션은 동작한다.

- 그러나 이것은 코드를 읽기 어렵게 하는 장애물일 뿐이다.

- 이렇게 작성한 코드가 있다면 실수로 입력한게 틀림 없다.

- 이 문제와 관련된 규칙은 [no-extra-semi](https://eslint.org/docs/latest/rules/no-extra-semi) 규칙이다.

- 문장 뒤에 세미콜론을 더 추가한 뒤,

```shell
// app.js

console.log();; // 세미콜론 연속 두 개 붙임
```

- 린트 설정에 no-extra-semi 규칙을 추가하고,

```js
// .eslintrc.js
module.exports = {
  rules: {
    "no-extra-semi": "error",
  },
};
```

- 코드를 검사하면 오류를 출력한다.

```shell
npx eslint app.js

1:15 error Unnecessary semicolon no-extra-semi

✖ 1 problem (1 error, 0 warnings)
1 error and 0 warnings potentially fixable with the `--fix` option.
```

- 마지막 줄의 메세지를 보면 이 에러는 "잠재적으로 수정가능(potentially fixable)"하다고 `--fix` 옵션으로 해결할 수 있다고 말한다.

- `--fix` 옵션을 붙여 검사해보면,

```shell

npx eslint app.js --fix

```

- 검사 후 오류가 발생하면 코드를 자동으로 수정한다.

- 즉, 불필요한 세미콜론이 삭제가 된다.

- 이렇듯 ESLint 규칙에는 수정 가능한 것과 그렇지 못한 것이 있다.

- 규칙 목록 중 왼쪽에 렌치 표시가 붙은 것이 `--fix` 옵션으로 자동 수정할 수 있는 규칙이다.

- 렌치 표시가 없는 것은 개발자가 직접 수정해주어야 하는 규칙이다.

### 2.5 Extensible Config

- 이러한 규칙을 여러개 세트로 미리 정해 놓은 것이 `eslint:recommended` 설정이다.

- [규칙 목록](https://eslint.org/docs/latest/rules/) 중에 왼쪽에 체크 표시되어 있는것이 이 설정에서 활성화되어 있는 규칙이다.

- 이것을 사용하려면 extends 설정을 추가한다.

```js
// .eslintrc.js

module.exports = {
  extends: [
    "eslint:recommended", // 미리 설정된 규칙 세트을 사용한다
  ],
};
```

- 그리고 아래와 같이 파일을 수정해준다

```shell
# app.js

console.log();; // 세미콜론 연속 두 개 붙임
(function(){})();
```

- 아래처럼 명령어를 실행하면 세미콜론을 자동으로 삭제되는 것을 확인할 수 있다

```shell

npx eslint app.js --fix

```

- 만약 이 설정 외에 규칙이 더 필요하다면 rules 속성에 추가해서 확장할 수 있다.

- ESLint에서 기본으로 제공하는 설정 외에 자주 사용하는 두 가지가 있다.

  - airbnb

  - standard

- [airbnb](https://github.com/airbnb/javascript) 설정은 airbnb 스타일 가이드를 따르는 규칙 모음이다.

  - [eslint-config-airbnb-base](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb-base) 패키지로 제공된다.

- [standard](https://standardjs.com/) 설정은 자바스크립트 스탠다드 스타일을 사용한다.

  - [eslint-config-standard](https://lean-mahogany-686.notion.site/6-125367b8270f4ea388e93d3fdac89639) 패키지로 제공된다.

### 2.6 초기화

사실 이러한 설정은 `--init` 옵션을 추가하면 손쉽게 구성할 수 있다.

```shell

npx eslint --init

? How would you like to use ESLint?
? What type of modules does your project use?
? Which framework does your project use?
? Where does your code run?
? How would you like to define a style for your project?
? Which style guide do you want to follow?
? What format do you want your config file to be in?

```

- 대화식 명령어로 진행되는데 모듈 시스템을 사용하는지, 어떤 프레임웍을 사용하는지, 어플리케이션이 어떤 환경에서 동작하는지 등에 답하면 된다.

- 답변에 따라 `.eslintrc` 파일을 자동으로 만들 수 있다.

## 실습

### 문제

- [app.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/3-lint/1-eslint/src/app.js)

- 위 파일에 해당하는 아래 내용을 해결할 수 있도록 eslint.rc 파일을 작성한다

```shell

/**
 * TODO eslint가 중복 세미콜론을 제거합니다.
 */


import MainController from './controllers/MainController.js'

import './app.scss'

const foo = ''

document.addEventListener("DOMContentLoaded", ()=>{
new MainController();;;;
})

```

### 해결

- 먼저 노드 패키지로 제공되는 ESLint 도구를 다운로드 한다.

```shell

npm i -D eslint

```

- 그리고 아래처럼 초기화를 통해 eslint.rc 파일을 만들어준다

```shell

npx eslint --lint

```

- 아래와 같이 extend가 추가되도록 한다

```js
// eslint.rc

module.exports = {
  extends: [
    "eslint:recommended", // 미리 설정된 규칙 세트을 사용한다
  ],
};
```

- eslint.rc 파일을 만들고 나서 아래처럼 lint를 실행하도록 script를 추가해준다

```json

"scripts":{
  "lint":"eslint src --fix"
}

```

- `npm run lint` 명령어를 실행하면 app.js 파일이 수정되어 세미콜론이 삭제된 것을 확인할 수 있다.

- 추가적으로 `no-unused-vars` 에러가 발생하는데 이는 코드품질에 관한 것이므로 사용되지 않는 변수를 직접 삭제해 준다

- 그리고 다시 `npm run lint` 실행하면 오류가 없는 것을 확인할 수 있다
