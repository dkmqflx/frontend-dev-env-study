## 3. 로더

### 3.1 로더의 역할

- 웹팩은 모든 파일을 모듈로 바라본다.

- 자바스크립트로 만든 모듈 뿐만아니라 스타일시트, 이미지, 폰트까지도 전부 모듈로 보기 때문에 import 구문을 사용하면 자바스크립트 코드 안으로 가져올수 있다.

- 이것이 가능한 이유는 웹팩의 로더 덕분이다.

- 로더는 타입스크립트 같은 다른 언어를 자바스크립트 문법으로 변환해 주거나 이미지를 data URL 형식의 문자열로 변환한다.

- 뿐만아니라 CSS 파일을 자바스크립트에서 직접 로딩할수 있도록 해준다.

- 즉, 로더는 각 파일을 처리 하기 위한 것

### 3.2 커스텀 로더 만들기

- 로더를 사용하기 전에 동작 원리를 이해하기 위해 로더를 직접 만들어 보자.

```js
// myloader.js

module.exports = function myloader(content) {
  console.log("myloader가 동작함");
  return content;
};
```

- 함수로 만들수 있는데 로더가 읽은 파일의 내용이 함수 인자 content로 전달된다.

- 로더가 동작하는지 확인하는 용도로 로그만 찍고 곧장 content를 돌려 준다.

- 로더를 사용하려면 웹팩 설정파일의 module 객체에 추가한다.

```js
// webpack.config.js

module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/, // .js 확장자로 끝나는 모든 파일, js 확장자를 가진 모든 파일을 이 로더로 돌리겠다는 뜻, 처리할 패턴을 명시한다
        use: [path.resolve("./myloader.js")], // 방금 만든 로더를 적용한다, 사용할 로더를 명시한다
      },
    ],
  },
};
```

- module.rules 배열에 모듈을 추가하는데 test와 use로 구성된 객체를 전달한다.

- test에는 로더에 적용할 파일을 지정한다.

- 파일명 뿐만아니라 파일 패턴을 정규표현식으로 지정할수 있는데 위 코드는 `.js` 확장자를 갖는 모든 파일을 처리하겠다는 의미다.

- use에는 이 패턴에 해당하는 파일에 적용할 로더를 설정하는 부분이다. 방금 만든 myloader 함수의 경로를 지정한다.

- 이제 npm run build로 웹팩을 실행해 보자.

- 터미널에 'myloader가 동작함' 문자열이 찍힌다.

- 그리고 이 때 자바스크립트 파일 갯수만큼 `console.log`가 찍히는 것 또한 확인할 수 있다

- myloader() 함수가 동작한 것이다.

- 위에서 작성한 코드를 아래처럼 수정해서 다시 빌드 해본다.

- 빌드결과를 살펴보면 이전과 동일하다. 로더가 뭔가를 처리하기 위해서 간단한 변환 작업을 추가해 보자.

- 소스에 있는 모든 console.log() 함수를 alert() 함수로 변경하도록 말이다.

```js
// myloader.js

// module.exports 명령어로 해당 파일을 로더로 만들어준다
// 로더는 이렇게 함수로 작성하는데, 읽은 파일의 내용을 content로 전달받는다

module.exports = function myloader(content) {
  console.log("myloader가 동작함");
  return content.replace("console.log(", "alert("); // console.log( -> alert( 로 치환
};
```

- 빌드후 확인하면 다음과 같이 console.log() 함수가 alert() 함수로 변경되었다.

- 빌드 후 해당 js 파일을 사용하는 html 파일을 실행시켜 보면 alert 창이 나타나는 것을 확인할 수 있다

- 즉, 웹팩 설정에서 entry로 등록한 파일을 시작으로, 해당 파일에서 사용하는 다른 파일에 대해서 로더가 동작한다

- 그리고 웹팩이 처리한 파일이 output에 저장되는 구조라서 웹팩의 output에 있는 파일은 로더가 건드리지 않는다.

## 4. 자주 사용하는 로더

### 4.1 css-loader

- 웹팩은 모든것을 모듈로 바라보기 때문에 자바스크립트 뿐만 아니라 스타일시트로 import 구문으로 불러 올수 있다.

```js
// app.js

import "./style.css";
```

```css
/* style.css */

body {
  background-color: green;
}
```

- CSS 파일을 자바스크립트에서 불러와 사용하려면 CSS를 모듈로 변환하는 작업이 필요하다.

- css-loader가 그러한 역할을 하는데 우리 코드에서 CSS 파일을 모듈처럼 불러와 사용할 수 있게끔 해준다.

- 먼저 로더를 설치 하자.

```shell

$ npm install -D css-loader

```

- 웹팩 설정에 로더를 추가한다.

```js
// webpack.config.js

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/, // .css 확장자로 끝나는 모든 파일
        use: ["css-loader"], // css-loader를 적용한다
        // css 파일 모듈로 만들기 위해서 로더를 사용한다
        // css 파일을 만나면 배열에 있는 로더가 실행된다
      },
    ],
  },
};
```

- 웹팩은 엔트리 포인트부터 시작해서 모듈을 검색하다가 CSS 파일을 찾으면 css-loader로 처리할 것이다.

- use.loader에 로더 경로를 설정하는 대신 배열에 로더 이름을 문자열로 전달해도 된다.

- webpack으로 빌드 후, main.js 파일을 보면 css 파일에 있는 코드가 js 파일 내에 문자열 형태로 있는 것을 확인할 수 있다

- 즉, 빌드 한 결과 CSS코드가 자바스크립트로 변환된 것을 확인할 수 있다.

### 4.2 style-loader

- 위에서 로더를 설정한 이후에 main.js를 사용하는 html 파일을 열고 개발자 도구의 Element 탭의 styles 항목을 확인해보면 css의 style이 없다.

- 그리고 네트워크 탭을 통해서 받아온 main.js 파일을 보면 해당 파일안에 css 파일에 있는 코드가 문자열 형태로 있다.

- html 코드가 DOM이라는 모습으로 변환되어야 브라우저에서 html 문서가 보이듯이

- css 코드도 cssom이 되어야 브라우저에서 확인할 수 있다

- 이렇게 하기 위해서는 html 에서 css 파일을 불러오거나 인라인 스타일을 작성해야 되는데

- 현재는 이러한 방식으로 처리한 것이 아니라 파일에만 스타일이 정의되어 있어서 css style이 없는 것이다.

- 이러한 문제를 해결하기 위한 로더가 것이 style-loader 이다

- style-loader는 js로 변경된 스타일 코드를 css html에 넣어주는 역할을 한다

- 즉, 모듈로 변경된 스타일 시트는 돔에 추가되어야만 브라우져가 해석할 수 있는데

- css-loader로 처리하면 자바스크립트 코드로만 변경되었을 뿐 돔에 적용되지 않았기 때문에 스타일이 적용되지 않았다.

- style-loader는 자바스크립트로 변경된 스타일을 동적으로 돔에 추가하는 로더이다.

- CSS를 번들링하기 위해서는 css-loader와 style-loader를 함께 사용한다.

- 먼저 스타일 로더를 다운로드 한다.

```shell

$ npm install -D style-loader

```

- 그리고 웹팩 설정에 로더를 추가한다.

```js
const path = require("path");

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
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
        // style-loader를 앞에 추가한다
        // 처리 순서는 뒤에서 부터 앞으로 진행된다 (<--)
      },
    ],
  },
};
```

- 빌드 후 다시 확인해보면 정상적으로 css 파일에 적용된 스타일이 적용된 것을 확인할 수 있다

- 개발자 도구의 Element 탭을 확인해보면 head 태그 안에 style 태그에 정의한 규칙이 있는 것을 확인할 수 있다

- 배열로 설정하면 뒤에서부터 앞으로 순서대로 로더가 동작한다.

- 위 설정은 모든 `.css` 확장자로 끝나는 모듈을 읽어 들여 css-loader를 적용하고 그 다음 style-loader를 적용한다.

### 4.3 file-loader

- 웹팩은 빌드 할 때 해시 값으로 파일명을 변경시키는데 이는 캐쉬 갱신을 위한 처리이다.

- 브라우저에서 자바스크립트 파일, 이미지, 폰트 등을 성능을 위해서 캐시한다

- 캐시는 성능 최적화 방법 중의 하나로 브라우져의 캐시 동작도 있고 웹 개발자가 설정할수 있는 캐시 동작도 있다.

- 브라우져에서는 모든 리소스(이미지, 폰트, 자바스크립트, 스타일시트)를 서버에서 가져오는데, 캐쉬해서 브라우저에 저장해 놓으면 매번 다운로드 하지 않아도 되기 때문에 어플리케이션 성능을 올릴 수 있다.

- 파일 내용이 달라지고 이름이 같으면 이전의 파일을 그대로 사용하기 때문에 이런 것을 예방하기 위해서 파일 이름을 해시 값으로 변경시키는 것이다

- CSS 뿐만 아니라 소스코드에서 사용하는 모든 파일을 모듈로 사용하게끔 할 수 있다.

- 파일을 모듈 형태로 지원하고 웹팩 아웃풋에 파일을 옮겨주는 것이 file-loader가 하는 일이다.

- 가령 CSS에서 url() 함수에 이미지 파일 경로를 지정할 수 있는데 웹팩은 file-loader를 이용해서 이 파일을 처리한다.

```css
style.css body {
  background-image: url(bg.png);
}
```

- 배경 이미지를 bg.png 파일로 지정했다.

- 웹팩은 엔트리 포인트인 app.js가 로딩하는 style.css 파일을 읽을 것이다.

- 그리고 이 스타일시트는 url() 함수로 bg.png를 사용하는데 이때 로더를 동작시킨다.

```js
// webpack.config.js

module.exports = {
  module: {
    rules: [
      {
        test: /\.png$/, // .png 확장자로 마치는 모든 파일
        loader: "file-loader", // 파일 로더를 적용한다
      },
    ],
  },
};
```

- 웹팩이 `.png` 파일을 발견하면 file-loader를 실행할 것이다.

- 로더가 동작하고 나면 아웃풋에 설정한 경로로 이미지 파일을 복사된다.

- 그리고 파일명이 해쉬코드로 변경 되는데 **캐쉬 갱신**을 위한 처리이다.

- 하지만 이대로 index.html 파일을 브라우져에 로딩하면 이미지를 제대로 로딩하지 못할 것이다.

- CSS를 로딩하면 background-image: url(bg.png) 코드에 의해 동일 폴더에서 이미지를 찾으려고 시도할 것이다.

- 그러나 웹팩으로 빌드한 이미지 파일은 output인 dist 폴더 아래로 이동했기 때문에 이미지 로딩에 실패할 것이다.

- file-loader 옵션을 조정해서 경로를 바로 잡아 주어야 한다.

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.png$/, // .png 확장자로 마치는 모든 파일
        loader: "file-loader",
        options: {
          publicPath: "./dist/", // prefix를 아웃풋 경로로 지정
          // 파일 로더가 처리하는 파일을 모듈로 처리 했을 때 경로 앞에 추가되는 문자열
          // output 파일이 dist 폴더 안에 있기 때문에 앞에 dist를 붙이고 파일 이름을 호출한다
          // 이렇게 설정을 해주어야지 빌드 후에 해당 이미지 파일을 사용하는 곳에서
          // './dist/'이미지파일.확장자' 형식으로 이미지 파일을 찾기 때문에, 정상적으로 이미지를 불러올 수 있게 된다

          name: "[name].[ext]?[hash]", // 파일명 형식
          // output으로 저장되는 파일명
          // [name]: 원본 파일명
          // [ext]: 확장자
          // [hash]: 캐시 무력화를 위해서 쿼리 스트링으로 달라지는 해시값을 입력한다
          // 빌드할 때 마다 쿼리 스트링이 변경되면 새로운 파일로 판단한다.
          // 그럼 브라우져에 이전에 다운로드한 파일을 무시하고 서버로 새로운 요청을 보내서 새로운 파일을 다운로드 받게 되는 것
          // 다만 이렇게 name을 지정한 경우에는 이전과 달리 dist 폴더 내의 이미지 파일이 원본 이미지와 같은 것을 확인할 수 있다
        },
      },
    ],
  },
};
```

- 빌드 한 다음에 js 파일에 보면 name에 지정한 형식으로 파일이 있는 것을 확인할 수 있다

- publicPath 옵션은 file-loader가 처리하는 파일을 모듈로 사용할 때 경로 앞에 추가되는 문자열이다.

- output에 설정한 'dist' 폴더에 이미지 파일을 옮길 것이므로 publicPath 값을 이것으로로 지정했다.

- 파일을 사용하는 측에서는 'bg.png'를 'dist/bg.png'로 변경하여 사용할 것이다.

- 또한 name 옵션을 사용했는데 이것은 로더가 파일을 아웃풋에 복사할때 사용하는 파일 이름이다.

- 기본적으로 설정된 해쉬값을 쿼리스트링으로 옮겨서 'bg.png?6453a9c65953c5c28aa2130dd437bbde' 형식으로 파일을 요청하도록 변경했다.

### 4.4 url-loader

- url-loader는 이미지 파일을 "자바스크립트 문자열"로 변환해서 코드에 넣어주는 역할을 하는 로더이다.

- 그래서 자바스크립트 파일을 좀 커지긴 하지만 이미지 여러개를 다운받는 것 보다는 좀 더 큰 자바스크립트 파일 1개를 다운로드하는게 더 나을 수 있기 때문에 사용한다

- 즉, 이렇게 file-loader와 url-loader를 분리해주는 이유는 모든 파일을 url-loader로 처리하게 되면 그만큼 자바스크립트 파일이 커지기 때문이다

- 사용하는 이미지 갯수가 많다면 네트워크 리소스를 사용하는 부담이 있고 사이트 성능에 영향을 줄 수도 있다.

- 만약 한 페이지에서 작은 이미지를 여러 개 사용한다면 [Data URI Scheme](https://en.wikipedia.org/wiki/Data_URI_scheme)을 이용하는 방법이 더 나은 경우도 있다.

- 이미지를 Base64로 인코딩하여 문자열 형태로 소스코드에 넣는 형식이다.

- [url-loader](https://github.com/webpack-contrib/url-loader)는 이러한 처리를 자동화해주는 로더이다.

- 먼저 로더를 설치한다.

```shell

$ npm install -D url-loader

```

- 그리고 웹팩 설정을 추가한다.

- 아래처럼 웹팩 설정 후 빌드 한 다음 dist 폴더를 limit 보다 작은 사이즈의 이미지 파일이 없다

- 그리고 limit 보다 큰 사이즈의 이미지만 있는데, limit 보다 큰 사이즈의 이미지는 file-loader에 의해 처리되기 때문이다

- 즉, 아래처럼 file-loader에 대한 설정을 하지 않아도 자동으로 limit 보다 큰 사이즈의 이미지는 file-loader에 의해 처리된다

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
    path: path.resolve("./dist"),
  },
  module: {
    rules: [
      {
        test: /\.(png|jpg|svg|gif)$/, // 보통 이런 식으로 대표적인 이미지 타입을 설정해준다
        loader: "url-loader",
        options: {
          name: "[name].[ext]?[hash]",
          limit: 10000, // 10Kb
          // 파일 용량을 세팅할 수 있는 설정으로
          // url 로더가 파일을 처리할 때 지정한 사이즈 미만의 파일은 base64로 변환하고
          // 그 이상의 경우에는 file-loader가 실행해서 처리한다
        },
      },
    ],
  },
};
```

- 아래처럼 이미지를 불러오는 코드가 있는 경우, 빌드 후 확인해보면 이미지가 정상적으로 보이는 것을 확인할 수 있다

- 그리고 main 파일 보면 base64로 인코딩 된 것을 확인할 수 있다

```js
import MainController from "./controllers/MainController.js";
import img from "./image.jpg"; // 이미지를 모듈로 불러온다

document.addEventListener("DOMContentLoaded", () => {
  document.body.innerHTML = `
    <img src="${img}">
  `;
});
```

- 아이콘처럼 용량이 작거나 사용 빈도가 높은 이미지는 파일을 그대로 사용하기 보다는 Data URI Scheeme을 적용하기 위해 url-loader를 사용하면 좋다.

### [Loader 실습 ](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/1-webpack/2-loader/src/views/ResultView.js)

- 문제

  - 파일을 로딩할수 있도록 웹팩 로더 설정을 추가한다 (file-loader나 image-loader)

- 아래처럼 웹팩 설정을 변경해준다

```js
const path = require("path");

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
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.(png|jpg|svg|gif)$/,
        loader: "file-loader",
        options: {
          publicPath: "./dist/",
          name: "[name].[ext]?[hash]",
        },
      },
    ],
  },
};
```

- 이렇게 웹팩 설정을 변경해주면 아래처럼 이미지 파일을 import 해서 사용할 수 있게 된다

```js
import defaultImage from "../images/default-image.jpg";
```
