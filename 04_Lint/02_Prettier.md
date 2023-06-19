## 3. Prettier

- 프리티어는 코드를 "더" 예쁘게 만든다.

- ESLint의 역할인 포맷팅과 코드품질 중 포매팅과 겹치는 부분이 있지만 프리티어는 좀 더 일관적인 스타일로 코드를 다듬는다.

- 반면 코드 품질과 관련된 기능은 하지 않는 것이 ESLint와 다른 점이다.

### 3.1 설치 및 사용법

- 프리티어 패키지를 다운로드 하고

```shell

npm i -D prettier

```

- 코드를 아래 처럼 세미콜론 없이 작성한다.

```shell
// app.js
console.log("hello world")
```

- Prettier로 검사해 보자.

```shell

npx prettier app.js

```

- 터미널에 프리티어를 거친 코드가 출력되는데 세미클론이 붙어 있는 코드가 출력되는 것을 확인할 수 있다.

```shell

npx prettier app.js --write

```

- `--write` 옵션을 추가하면 파일을 재작성한다.

- 그렇지 않을 경우 결과를 터미널에 출력한다.

- 변경된 모습을 보면,

```js
// app.js

console.log("Hello world");
```

- 작은 따옴표를 큰 따옴표로 변경했다.

- 문장 뒤에 세미콜론도 추가했다.

- 프리티어는 ESLint와 달리 규칙이 미리 세팅되어 있기 때문에 설정 없이도 바로 사용할 수 있다.

### 3.2 포매팅(더 예쁘게)

- 다음 코드를 보자.

```shell
// app.js

console.log("----------------매 우 긴 문 장 입 니 다 80자가 넘 는 코 드 입 니 다.----------------");
```

- ESLint는 [max-len](https://eslint.org/docs/latest/rules/max-len) 규칙을 이용해 위 코드를 검사하고 결과만 알려 줄 뿐 수정하는 것은 개발자의 몫이다.

- 반면 프리티어는 어떻게 수정해야할지 알고 있기 때문에 아래처럼 코드를 다시 작성한다.

```js
// app.js
console.log(
  "----------------매 우 긴 문 장 입 니 다 80자가 넘 는 코 드 입 니 다.----------------"
);
```

- 아래 코드는 어떻게 변환할까?

```shell
foo(reallyLongArg(),omgSoManyParameters(),IShouldRefactorThis(),isThereSeriouslyAnotherOne());
```

- 프리티어는 코드를 문맥을 어느 정도 파악하고 상황에 따라 최적의 모습으로 스타일을 수정한다.

```js
foo(
  reallyLongArg(),
  omgSoManyParameters(),
  IShouldRefactorThis(),
  isThereSeriouslyAnotherOne()
);
```

- 더 멋진 예제도 있는데 프리티어를 만든 [James Long](https://archive.jlongster.com/A-Prettier-Formatter)의 글을 참고하자.

- 이러한 포매팅 품질은 ESLint보다는 프리티어가 훨씬 좋은 결과를 만든다.

- 사람에게 더 친숙하도록 말이다.

### 3.3 통합방법

- 여전히 ESLint를 사용해야 하는 이유는 남아 있다.

- 포맷팅은 프리티어에게 맡기더라도 코드 품질과 관련된 검사는 ESLint의 몫이기 때문이다.

- 따라서 이 둘을 같이 사용하는 것이 최선이다.

- 프리티어는 이러한 ESLint와 통합 방법을 제공한다.

- [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier) 는 프리티어와 충돌하는 ESLint 규칙을 끄는 역할을 한다.

- 둘 다 사용하는 경우 규칙이 충돌하기 때문이다.

- 패키지를 설치한뒤,

```shell

npm i -D eslint-config-prettier

```

- 설정파일의 extends 배열에 추가한다.

```json
// .eslintrc.js
{
  "extends": ["eslint:recommended", "eslint-config-prettier"]
}
```

- 예를 들어 ESLint는 중복 세미콜론 사용을 검사한다.

- 이것을 프리티어도 마찬가지다.

- 따라서 어느 한쪽에서는 규칙을 꺼야하는데 eslint-config-prettier를 extends 하면 중복되는 ESLint 규칙을 비활성화 한다.

```shell
var foo = ""; // 사용하지 않은 변수. ESLint가 검사
console.log();;;;; // 중복 세미콜론 사용. 프리티어가 자동 수정
```

- 아래처럼 eslint 명령어를 실행하면 ESLint는 중복된 포매팅 규칙을 프리티어에게 맞기고 나머지 코드 품질에 관한 검사만 한다.

```shell

npx eslint app.js --fix

```

- 즉, `var foo = ""`에 대해서는 검사를 하지만 중복 세미콜론은 처리하지 않는다

- 아래처럼 prettier 명령어를 실행해야지 프리티어가 중복 세미클론을 처리해준다

```shell

npx prettier app.js --write

```

- 매번 두 린트와 프리티어를 각각 실행하는 것은 불편하다

- 따라서 아래처럼 두 개를 동시에 실행해서 코드를 검사한다.

```shell

npx prettier app.js --write && npx eslint app.js --fix

1:5 error 'foo' is assigned a value but never used no-unused-vars

✖ 1 problem (1 error, 0 warnings)

```

- 프리티어에서 의해 코드가 아래과 같이 포매팅 되었고

```js
var foo = "";
console.log();
```

- ESlint에 의해 코드 품질과 관련된 오류(no-unused-vars)를 리포팅한다.

- 한편, [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier)는 프리티어 규칙을 ESLint 규칙으로 추가하는 플러그인이다.

- 프리티어의 모든 규칙이 ESLint로 들어오기 때문에 ESLint만 실행하면 된다.

- 패키지를 설치하고

```shell

npm i -D eslint-plugin-prettier

```

- 설정 파일에서 pulugins와 rules에 설정을 추가한다.

```json
// .eslintrc.js
{
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

- 프리티어의 모든 규칙을 ESLint 규칙으로 가져온 설정이다.

- 이제는 ESLint만 실행해도 프리티어 포매팅 기능을 가져갈 수 있다.

```shell

npx eslint app.js --fix

```

- 프리티어는 이 두 패키지를 함께 사용하는 [단순한 설정](https://prettier.io/docs/en/integrating-with-linters.html)을 제공하는데 아래 설정을 추가하면 된다.

```json
// .eslintrc.js

{
  "extends": ["eslint:recommended", "plugin:prettier/recommended"]
}
```

### 실습

- [app.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/3-lint/2-prettier/src/app.js)

- [eslintrc.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/3-lint/2-prettier/.eslintrc.js)

- 아래와 같은 app.js 파일을 수정할 수 있도록 한다

  ```shell

  /**
  * TODO: Prettier가 스타일을 수정합니다.
  */


  import MainController from './controllers/MainController.js'

  import './app.scss'


  document.addEventListener("DOMContentLoaded", ()=>{
  new MainController();
  })
  ```

- 먼저 필요한 패키지를 설치해준다

```shell

npm i -D prettier  eslint-config-prettier eslint-plugin-prettier

```

- 아래처럼 eslintrc.js 파일의 extends 부분을 수정해준다

```js
// .eslintrc.js

module.exports = {
  env: {
    browser: true,
    es6: true,
  },
  extends: ["eslint:recommended", "plugin:prettier/recommended"],
  globals: {
    Atomics: "readonly",
    SharedArrayBuffer: "readonly",
  },
  parserOptions: {
    ecmaVersion: 2018,
    sourceType: "module",
  },
  rules: {},
};
```

- `npm run lint` 명령어를 실행후 파일을 확인해보면 아래처럼 single quote는 double quote로, 그리고 공백과 indent가 추가된 것을 확인할 수 있다

```js
/**
 * TODO: Prettier가 스타일을 수정합니다.
 */

import MainController from "./controllers/MainController.js";

import "./app.scss";

document.addEventListener("DOMContentLoaded", () => {
  new MainController();
});
```

## 4. 자동화

- 린트는 코딩할 때마다 수시로 실행해야하는데 이러한 일은 자동화 하는 것이 좋다.

- "깃 훅을 사용하는 방법"과 "에디터 확장 도구"를 사용하는 방법을 각각 소개한다.

### 4.1 변경한 내용만 검사

- 소스 트래킹 도구로 깃을 사용한다면 깃 훅을 이용하는 것이 좋다.

- 커밋 전, 푸시 전 등 깃 커맨드 실행 시점에 끼여들수 있는 훅을 제공한다.

- [husky](https://github.com/typicode/husky)는 깃 훅을 쉽게 사용할 수 있는 도구다.(Git 2.13.0 이상 버전을 지원)

- 커밋 메세지 작성전에 끼어들어 린트로 코드 검사 작업을 추가하면 좋겠다.

- 먼저 패키지를 다운로드 한다.

```shell

npm i -D husky

```

- 허스키는 패키지 파일에 설정을 추가한다.

```json
// package.json

{
  "husky": {
    "hooks": {
      "pre-commit": "echo \"이것은 커밋전에 출력됨\""
    }
  }
}
```

- 훅이 제대로 동작하는지 빈 커밋을 만들어 보자.

```shell

$ git commit --allow-empty -m "빈 커밋"

husky > pre-commit (node v13.1.0)
이것은 커밋전에 출력됨 # ----> 깃 훅이 동작함
[master db8b4b8] empty

```

- pre-commit에 설정한 내용이 출력되었다.

- 출력 대신에 린트 명령어로 대체하면 커밋 메세지 작성 전에 린트를 수행할 수 있겠다.

```json
// package.json

{
  "husky": {
    "hooks": {
      "pre-commit": "eslint app.js --fix"
    }
  }
}
```

- 만약 린트 수행중 오류를 발견하면 커밋 과정은 취소된다.

- 린트를 통과하게끔 코드를 수정해야만 커밋할 수 있는 환경이 되었다.

- 한편, 코드가 점점 많아지면 커밋 작성이 느려질 수 있는데 커밋전에 모든 코드를 린트로 검사하는 시간이 소요되기 때문이다.

- 커밋시 변경된 파일만 린트로 검사하면 더 좋지 않을까?

- [lint-staged](https://github.com/okonet/lint-staged)는 변경된(스테이징된) 파일만 린트를 수행하는 도구다.

- 패키지를 설치하고,

```shell

npm i -D lint-staged

```

- 패키지 파일에 설정을 추가한다.

```json
// package.json

{
  "lint-staged": {
    "*.js": "eslint --fix"
  }
}
```

- 내용이 변경된 파일 중에 `.js` 확장자로 끝나는 파일만 린트로 코드 검사를 한다.

- pre-commit 훅도 아래처럼 변경한다.

```json
// package.json

{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": "eslint --fix"
    // 모든 js 확장자에 대해서 staging 된 파일이 있으면 린트를 실행한다
  }
}
```

- 커밋 메세지 작성전에 lint-staged를 실행할 것이다.

- 이제 커밋하면 모든 파일을 검사하는 것이 아니라 변경되거나 추가된 파일만 검사한다.

- 커밋 과정이 훨씬 가벼워질 것이다.

### 4.2 에디터 확장도구

- 코딩할때 실시간으로 검사하는 방법도 있다.

- vs-code의 eslint와 prettier 익스텐션이 그러한 기능을 제공한다.

- 우리는 프리티어 규칙을 ESLint와 통합했기 때문에 ESLint 익스텐션을 사용해 보겠다.

- 먼저 vs-code에서 ESLint 익스텐션 부터 설치해 보자.

- 설치를 마친 뒤 eslint를 활성화 설정을 추가한다.

```json
// .vscode/settings.json:

{
  "eslint.enable": true
}
```

- 설치하면 자동으로 ESLint 설정파일을 읽고 파일을 검사한다.

- 아래처럼 코드가 작성되어있다면 해당 코드에 hover 한 후 툴팁 메뉴를 클릭해서 문제를 수정할 수 있다

```js
var foo = "";
```

- 에디터 설정중 저장시 액션을 추가할 수 있는데 ESLint로 코드를 정정할 수 있다.

```json
// .vscode/settings.json:

{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

- 이렇게 ESLint 익스텐션으로는 실시간 코드 품질 검사를 하고 저장시 자동 포메팅을 하도록 하면 실시간으로 코드 품질을 검사하고 포맷도 일관적으로 유지할 수 있다.

## 5. 정리

- 읽기 좋은 코드는 유지보수 하기좋다.

- 그만큼 어플리케이션의 수명은 오래갈 수 있다.

- 여럿이서 함께 일하는 환경에서 손으로 코드를 관리하는 것은 무척 번거럽고 어쩌면 불가능한 일일지도 모른다.

- 규칙이 정해졌고 자동화할 수 있다면 도구의 도움을 받는 것이 현명하다.

- ESLint는 오류와 버그의 가능성을 찾아 코드 품질을 높이는 역할을 한다.

- 프리티어는 코드를 일관적으로 포매팅하기 때문에 읽기 수월한 코드를 만들어 준다.

- 이러한 도구를 개발 플로우의 적절한 시점에 통합하여 자동화하면 개발자는 좀 더 본질적인 코딩에 집중할 수 있을 것이다.
