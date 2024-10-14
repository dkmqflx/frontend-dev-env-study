## 5. 플러그인

### 5.1 플러그인의 역할

- 웹팩에서 알아야 할 마지막 기본 개념이 플러그인이다.

- 로더가 파일 단위로 처리하는 반면 플러그인은 번들된 결과물을 처리한다.

- 번들된 자바스크립트를 난독화 한다거나 특정 텍스트를 추출하는 용도로 사용한다.

- 이것도 사용하기에 앞서 동작 원리를 이해하기 위해 플러그인을 직접 만들어 보자.

### 5.2 커스텀 플러그인 만들기

- 웹팩 문서의 [Writing a plugin](https://webpack.js.org/contribute/writing-a-plugin/)을 보면 클래스로 플러그인을 정의 하도록 한다.

- [헬로월드 코드](https://webpack.js.org/contribute/writing-a-plugin/#basic-plugin-architecture)를 가져다 그대로 실행 붙여보자.

- 로더와 다르게 플러그인은 클래스로 제작한다

```js
// myplugin.js

class MyPlugin {
  apply(compiler) {
    compiler.hooks.done.tap("My Plugin", (stats) => {
      console.log("MyPlugin: done");
    });
  }
}

module.exports = MyPlugin;
```

- 로더와 다르게 플러그인은 클래스로 제작한다.

- apply 함수를 구현하면 되는데 웹팩을 통해 compiler라는 객체를 전달받는다

- 위 코드에서는 웹팩을 통해 인자로 받은 compiler 객체 안에 있는 tap() 함수를 사용한다.

- 여기서 등록된 콜백함수는 플러그인 작업이 완료되는(done) 시점에 로그를 찍는다

- 아래처럼 웹팩 설정을 변경해서 플러그인을 사용할 수 있다

```js
// webpack.config.js

const path = require("path");
const MyWebpackPlugin = require("MyWebpackPlugin");

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
        loader: "url-loader",
        options: {
          name: "[name].[ext]?[hash]",
          limit: 10000, // 10Kb
        },
      },
    ],
  },
  plugins: [new MyWebpackPlugin()],
};
```

- 웹팩 설정 객체의 plugins 배열에 설정한다.

- 클래스로 제공되는 플러그인의 생성자 함수를 실행해서 넘기는 방식이다.

- 빌드를 해보면 `MyPlugin: done` 로그가 찍히는 것을 확인할 수 있는데,

- 그런데 파일이 여러 개인데 로그는 한 번만 찍혔다.

- 모듈이 파일 하나 혹은 여러 개에 대해 동작하는 반면 **플러그인은 하나로 번들링된 결과물을 대상으로 동작**한다.

- 이를 통해 플러그인은 로더와 달리 각각의 파일에 대해서 실행하는 것이 아니라 번들된 파일에 대해서 딱 한번 실행되는 것을 알 수 있다.

- 우리 예제에서는 main.js로 결과물이 하나이기 때문에 플러그인이 한 번만 동작한 것이라 추측할 수 있다.

- 그러면 어떻게 번들 결과에 접근할 수 있을까?

  - 웹팩 내장 플러그인 [BannerPlugin](https://github.com/lcxfs1991/banner-webpack-plugin/blob/master/index.js) 코드를 참고하자.

- compiler.plugin() 함수의 두번재 인자 콜백함수는 emit 이벤트가 발생하면 실행된다.

- 번들된 결과가 compilation 객체에 들어 있는데 `compilation.assets['main.js'].source()` 함수로 접근할 수 있다.

- 실행하면 터미널에 번들링된 결과물을 확인할 수 있다.

```js
// myplugin.js

class MyPlugin {
  apply(compiler) {
    compiler.hooks.done.tap("My Plugin", (stats) => {
      console.log("MyPlugin: done");
    });

    // compiler.plugin() 함수로 후처리한다
    compiler.plugin("emit", (compilation, callback) => {
      const source = compilation.assets["main.js"].source();
      console.log(source);
      callback();
    });
  }
}
```

- 이걸 이용해서 번들 결과 상단에 아래와 같은 배너를 추가하는 플러그인으로 만들어 보자.

```js

// myplugin.js

class MyPlugin {
  apply(compiler) {
    compiler.plugin('emit', (compilation, callback) => {
      const source = compilation.assets['main.js'].source();
      compilation.assets['main.js'].source = () => {
        const banner = [
          '/**',
          ' * 이것은 BannerPlugin이 처리한 결과입니다.',
          ' * Build Date: 2019-10-10',
          ' */'
          ''
        ].join('\n');
        return banner + '\n' + source;
      }

      callback();
    })
  }
}
```

- 번들 소스를 얻어오는 함수 source()를 재정의 했다. 배너 문자열과 기존 소스 코드를 합친 문자열을 반환하도록 말이다.

- 빌드하고 결과물을 확인해 보면 다음과 같다.

## 6. 자주 사용하는 플러그인

- 개발하면서 플러그인을 직접 작성할 일은 거의 없었다.

- 웹팩에서 직접 제공하는 플러그인을 사용하거나 써드파티 라이브러리를 찾아 사용하는데 자주 사용하는 플러그인에 대해 알아보자.

### 6.1 BannerPlugin

- MyPlugin와 비슷한 것이 BannerPlugin이다.

- 결과물에 빌드 정보나 커밋 버전같은 걸 추가할 수 있다.

```js
// webpack.config.js

const webpack = require("webpack");

module.exports = {
  plugins: [
    new webpack.BannerPlugin({
      banner: "이것은 배너 입니다",
    }),
  ],
};
```

- 생성자 함수에 전달하는 옵션 객체의 banner 속성에 문자열을 전달한다.

- 웹팩 컴파일 타임에 얻을 수 있는 정보, 가령 빌드 시간이나 커밋정보를 전달하기위해 함수로 전달할 수도 있다.

```js
const webpack = require("webpack");
const childProcess = require("child_process");
// 노드 모듈 중에 child_process라는 것이 있는데, 해당 모듈을 통해서 터미널 명령어를 실행할 수 있다

module.exports = {
  plugins: [
    new webpack.BannerPlugin({
      banner: () => `
      빌드 날짜: ${new Date().toLocaleString()}
      Commit Version: ${childProcess.execSync("git rev-parse --short HEAD")}
      User Name: ${childProcess.execSync("git config user.name")}
      `,
      // 커밋 해시와 유저를 볼 수 있는 명령어
    }),
  ],
};
```

- 빌드한 후 확인해보면 리턴한 커밋 정보가 상단에 추가된 것을 확인할 수 있다

- 배너 정보가 많다면 아래처럼 별도의 파일로 분리해준다

```js
const banner = require("./banner.js");

new webpack.BannerPlugin(banner);
```

```js
// banner.js:

const childProcess = require("child_process");

module.exports = function banner() {
  const commit = childProcess.execSync("git rev-parse --short HEAD");
  const user = childProcess.execSync("git config user.name");
  const date = new Date().toLocaleString();

  return (
    `commitVersion: ${commit}` + `Build Date: ${date}\n` + `Author: ${user}`
  );
};
```

### 6.2 DefinePlugin

- 어플리케이션은 개발환경과 운영환경으로 나눠서 운영한다.

- 가령 환경에 따라 API 서버 주소가 다를 수 있다.

- 같은 소스 코드를 두 환경에 배포하기 위해서는 이러한 환경 의존적인 정보를 소스가 아닌 곳에서 관리하는 것이 좋다.

- 배포할 때마다 코드를 수정하는 것은 곤란하기 때문이다.

- 웹팩은 이러한 환경 정보를 제공하기 위해 DefinePlugin을 제공한다.

- 빌드 타임에 결정된 값을 어플리이션에 전달할 때 사용하는 플러그인이다

```js
// webpack.config.js

const webpack = require("webpack");
// DefinePlugin은 웹팩의 기본 프러그인

export default {
  plugins: [new webpack.DefinePlugin({})],
};
```

- 빈 객체를 전달해도 기본적으로 넣어주는 값이 있다.

- 노드 환경정보인 process.env.NODE_ENV 인데 웹팩 설정의 mode에 설정한 값이 여기에 들어간다.

- "development"를 설정했기 때문에 어플리케이션 코드에서 process.env.NODE_ENV 변수로 접근하면 "development" 값을 얻을 수 있다.

```js
// app.js

console.log(process.env.NODE_ENV); // "development"
```

- 이 외에도 웹팩 컴파일 시간에 결정되는 값을 전역 상수 문자열로 어플리케이션에 주입할 수 있다.

```js
new webpack.DefinePlugin({
  TWO: "1+1",
});
```

- TWO라는 전역 변수에 1+1 이란 코드 조각을 넣었다. 실제 어플리케이션 코드에서 이것을 출력해보면 2가 나올 것이다.

```js
// app.js

console.log(TWO); // 2
// 코드가 아닌 값을 입력하려면 문자열화 한 뒤 넘긴다.
```

```js
new webpack.DefinePlugin({
  VERSION: JSON.stringify("v.1.2.3"),
  PRODUCTION: JSON.stringify(false),
  MAX_COUNT: JSON.stringify(999),
  "api.domain": JSON.stringify("http://dev.api.domain.com"),
});
```

```js
// app.js

console.log(VERSION); // 'v.1.2.3'
console.log(PRODUCTION); // true
console.log(MAX_COUNT); // 999
console.log(api.domain); // 'http://dev.api.domain.com'
```

- 빌드 타임에 결정된 값을 어플리이션에 전달할 때는 이 플러그인을 사용하자.

### 6.3 HtmlWebpackPlugin

- Webpack의 기본 플러그인은 아니고 써드 파티 패키지이다.

- HtmlWebpackPlugin은 HTML 파일을 후처리하는데 사용한다.

- 빌드 타임의 값을 넣거나 코드를 압축할수 있다.

- 즉, HtmlWebpackPlugin을 사옹하면 유동적으로 html 템플릿을 만들 수 있다

```shell

$ npm install -D html-webpack-plugin

```

- 이 플러그인으로 빌드하면 HTML파일로 아웃풋에 생성될 것이다.

- 기존에는 html 파일을 제외하고 나머지 파일을 번들링 한 다음에 불러와서 사용지만

- html 파일 또한 웹팩의 빌드 과정에 넣고 싶다면 index.html 파일을 src/index.html로 옮겨준다

- 아래처럼 웹팩 설정을 수정해준다

- 이렇게 빌드해주면 dist 폴더에 html 파일도 있는 것을 확인할 수 있다

- 이 때 주의할 점은, dist 폴더 내에 html 파일이 있기 때문에 아래처럼 이미지를 불러오는데 사용한 publicPath를 제거해주어야 한다

- `dist/dist.이미지.확장자` 방식으로 이미지를 불러오기 때문에 에러가 발생하기 때문이다

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.png$/,
        loader: "file-loader",
        options: {
          // publicPath: "./dist/",
          name: "[name].[ext]?[hash]",
        },
      },
    ],
  },
};
```

- HtmlWebpackPlugin을 사옹하면 유동적으로 html 템플릿을 만들 수 있는데 아래처럼 title 태그 내에서 개발 환경에서만 전달받은 변수 값을 출력하도록 할 수 있다

- 타이틀 부분에 ejs 문법을 이용하는데 `<%= env %>` 는 전달받은 env 변수 값을 출력한다.

- index.html 파일을 src/index.html로 옮긴뒤 다음과 같이 작성해 보자.

```html
<!-- src/index.html: -->

<!DOCTYPE html>
<html>
  <head>
    <title>타이틀<%= env %></title>
  </head>
  <body>
    <!-- 로딩 스크립트 제거 -->
    <!-- <script src="dist/main.js"></script> -->

    <!-- 빌드 후 추가되는 코드  -->
    <script type="text/javascript" src="main.js"></script>
  </body>
</html>
```

- 타이틀 부분에 ejs 문법을 이용하는데 `<%= env %>` 는 전달받은 env 변수 값을 출력한다.

- HtmlWebpackPlugin은 이 변수에 데이터를 주입시켜 동적으로 HTML 코드를 생성한다.

- 뿐만 아니라 웹팩으로 빌드한 결과물을 자동으로 로딩하는 코드를 주입해 준다. 때문에 위처럼 스크립트 로딩 코드도 제거했다.

```js
// webpack.config.js

const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html", // 템플릿 경로를 지정
      templateParameters: {
        // 템플릿에 주입할 파라매터 변수 지정
        env: process.env.NODE_ENV === "development" ? "(개발용)" : "",
      },
    }),
  ],
};
```

- 환경 변수에 따라 타이틀 명 뒤에 "(개발용)" 문자열을 붙이거나 떼거나 하도록 했다.

- `NODE_ENV=development` 로 설정해서 빌드하면 빌드결과 "타이틀(개발용)"으로 나온다.

- `NODE_ENV=production` 으로 설정해서 빌드하면 빌드결과 "타이틀"로 나온다.

<br/>

- 개발 환경과 달리 운영 환경에서는 파일을 압축하고 불필요한 주석을 제거하는 것이 좋다.

- 아래와 같이 파일내에 주석이 있는 경우에 필요한 옵션을 설정해서 처리할 수 있다

```html
<!-- src/index.html: -->

<!DOCTYPE html>
<html>
  <head>
    <title>타이틀<%= env %></title>
  </head>
  <body>
    <!-- 주석  -->
  </body>
</html>
```

- 아래처럼 웹팩 설정을 변경해준다

```js
// webpack.config.js:

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      minify:
        process.env.NODE_ENV === "production"
          ? {
              collapseWhitespace: true, // 빈칸 제거
              removeComments: true, // 주석 제거
            }
          : false,
    }),
  ],
};
```

- (문서에는 minifiy 옵션이 웹팩 버전 3 기준으로 되어 있다)

- `NODE_ENV=production npm run build` 로 빌드하면 html 파일 내의 태그가 개행처리된 것 없이 한줄로 코드가 압축되고 주석도 제거 된 것을 확인할 수 있다

<br/>

- 정적파일을 배포하면 즉각 브라우져에 반영되지 않는 경우가 있다.

- 브라우져 캐쉬가 원인일 경우가 있는데 이를 위한 예방 옵션도 있다.

```js
// webpack.config.js:

new HtmlWebpackPlugin({
  hash: true, // 정적 파일을 불러올때 쿼리문자열에 웹팩 해쉬값을 추가한다
});
```

- `hash: true` 옵션을 추가하면 빌드할 시 생성하는 해쉬값을 정적파일 로딩 주소의 쿼리 문자열로 붙여서 HTML을 생성한다.

### 6.4 CleanWebpackPlugin

- CleanWebpackPlugin은 빌드 이전 결과물을 제거하는 플러그인이다.

- 빌드 결과물은 아웃풋 경로에 모이는데 과거 파일이 남아 있을수 있다.

- 이전 빌드내용이 덮여 씌여지면 상관없지만 그렇지 않으면 아웃풋 폴더에 여전히 남아 있을 수 있다.

- 즉, 아웃풋 폴더인 dist 폴더를 삭제해주는 역할을 한다

- 임시로 아웃풋 폴더 dist 폴더에 src 폴더 내에 있는 파일과 중복되지 않는 foo.js 파일을 만들어주자

- 그리고 다시 빌드해 보면 foo.js 파일이 여전히 dist 폴더에 남아 있다.

- 이러한 문제를 CleanWebpackPlugin을 통해서 해결할 수 있다

```shell

$ npm install -D clean-webpack-plugin

```

- 웹팩 설정을 추가한다.

```js
// webpack.config.js:

const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
  plugins: [new CleanWebpackPlugin()],
};
```

- 빌드 결과 foo.js가 깨끗히 사라졌다.

- 아웃풋 폴더인 dist 폴더가 모두 삭제된후 결과물이 생성되었기 때문이다.

### 6.5 MiniCssExtractPlugin

- 스타일시트가 점점 많아지면 하나의 자바스크립트 결과물로 만드는 것이 부담일 수 있다.

  - 브라우저에서 큰 파일 하나를 받아오는 것이 로딩 성능에 영향을 줄 수 있기 때문이다

- 번들 결과에서 스트일시트 코드만 뽑아서 별도의 CSS 파일로 만들어 역할에 따라 파일을 분리하는 것이 좋다.

- 브라우져에서 큰 파일 하나를 내려받는 것 보다, 여러 개의 작은 파일을 동시에 다운로드하는 것이 더 빠르다.

  - 즉, js 파일하나, css 하나 이런식으로 분리하는 페이지 로딩 성능에는 더 좋다

- 개발 환경에서는 CSS를 하나의 모듈로 처리해도 상관없지만 프로덕션 환경에서는 분리하는 것이 효과적이다.

  - 개발환경에서는 이렇게 분리하지 않는 것이 더 빨리 빌드될 것 이기 때문에 프로덕션 환경에서만 처리할 수 있도록 코드를 작성한다

- MiniCssExtractPlugin은 CSS를 별로 파일로 뽑아내는 플러그인이다.

```shell

$ npm install -D mini-css-extract-plugin

```

- 웹팩 설정을 추가한다.

```js
// webpack.config.js:

const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  plugins: [
    // spread 연산자로 전달
    ...(process.env.NODE_ENV === "production"
      ? [new MiniCssExtractPlugin({ filename: `[name].css` })] // 생성될 파일이름을 원본 파일이름으로 한다
      : []),
  ],
};
```

- 프로덕션 환경일 경우만 이 플러그인을 추가했다.

- filename에 설정한 값으로 아웃풋 경로에 CSS 파일이 생성될 것이다.

- 개발 환경에서는 css-loader에의해 자바스크립트 모듈로 변경된 스타일시트를 적용하기위해 style-loader를 사용했다.

- 반면 프로덕션 환경에서는 별도의 CSS 파일으로 추출하는 플러그인을 적용했으므로 다른 로더가 필요하다.

- 플러그인에서 제공하는 MiniCssExtractPlugin.loader 로더를 추가한다.

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          process.env.NODE_ENV === "production"
            ? MiniCssExtractPlugin.loader // 프로덕션 환경
            : "style-loader", // 개발 환경
          "css-loader",
        ],
      },
    ],
  },
};
```

- `NODE_ENV=production npm run build`로 결과를 확인해보자.

- dist/main.css가 생성되었고 index.html에 이 파일을 로딩하는 `<link href='main.css' rel='stylesheet'></link>` 태그 코드가 추가되었다.

### 실습

- [webpack.config.js](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/1-webpack/3-plugin/webpack.config.js)

- [index.html](https://github.com/jeonghwan-kim/lecture-frontend-dev-env/blob/1-webpack/3-plugin/src/index.html)

- 아래와 같이 webpack.config.js 파일을 작성한다

```js
const path = require("path");
const webpack = require("webpack");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

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
        use: [
          process.env.NODE_ENV === "production"
            ? MiniCssExtractPlugin.loader
            : "style-loader",
          "css-loader",
        ],
      },
      {
        test: /\.(png|jpg|svg|gif)$/,
        loader: "url-loader",
        options: {
          name: "[name].[ext]?[hash]",
          limit: 10000, // 10Kb
        },
      },
    ],
  },
  // 플러그인 추가
  plugins: [
    new webpack.BannerPlugin({
      banner: `Build Time: ${new Date().toLocaleString()}`, // BannerPlugin: 결과물에 빌드 시간을 출력하세요.
    }),

    new HtmlWebpackPlugin({
      template: "./src/index.html", // 2. HtmlWebpackPlugin: 동적으로 html 파일을 생성하세요.
      templateParameters: {
        env: process.env.NODE_ENV === "development" ? "(개발용)" : "",
      },
    }),

    new CleanWebpackPlugin(), // 3. CleanWebpackPlugin: 빌드 전에 아웃풋 폴더를 깨끗히 정리하세요.

    ...(process.env.NODE_ENV === "production"
      ? [new MiniCssExtractPlugin({ filename: `[name].css` })]
      : []), // 4. MiniCssExtractPlugin: 모듈에서 css 파일을 분리하세요.
  ],

  /**
   * TODO: 아래 플러그인을 추가해서 번들 결과를 만들어 보세요.
   * 1. BannerPlugin: 결과물에 빌드 시간을 출력하세요.
   * 2. HtmlWebpackPlugin: 동적으로 html 파일을 생성하세요.
   * 3. CleanWebpackPlugin: 빌드 전에 아웃풋 폴더를 깨끗히 정리하세요.
   * 4. MiniCssExtractPlugin: 모듈에서 css 파일을 분리하세요.
   */
};
```

- 플러그인을 실행하면 html 파일은 아래와 같이 변경된다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <!-- TODO: HtmlWebpackPlugin에서 빌드 환경을 주입하도록 웹팩을 구성하세요 -->
    <!-- <title>검색</title> -->
    <!-- 아래와 같이 코드를 변경한다음 process.env.NODE_ENV npm run build 명령어 실행시, 웹팩에 설정한 env 값이 title 태그 안에 들어가 검색(개발용)과 같이 값이 변경된다  -->
    <title>검색<%= env %></title>

    <!-- TODO: HtmlWebpackPlugin에서 엔트리 포인트(main.css)를 로딩하도록 웹팩을 구성하세요 -->
    <!-- MiniCssExtractPlugin 플러그인 추가후 빌드하면 아래처럼 스타일 파일을 불러오는 코드가 추가된다 -->
    <link href="main.css" rel="stylesheet" />
  </head>
  <body>
    <div id="app">
      <header>
        <h2 class="container">검색</h2>
      </header>

      <div class="container">
        <form class="FormView">
          <input type="text" placeholder="검색어를 입력하세요" autofocus />
          <button type="reset" class="btn-reset"></button>
        </form>

        <div class="content">
          <div id="tabs"></div>
          <div id="search-keyword"></div>
          <div id="search-history"></div>
          <div id="search-result"></div>
        </div>
      </div>
    </div>

    <!-- TODO: HtmlWebpackPlugin에서 엔트리 포인트(main.js)를 로딩하도록 웹팩을 구성하세요 -->
    <!-- HtmlWebpackPlugin 플러그인 추가후 빌드하면 아래처럼 js 파일을 불러오는 코드가 추가된다 -->
    <!-- 웹팩 설정 중에 entry에 해당하는 파일이 src로 추가되어 있다 -->
    <script type="text/javascript" src="main.js"></script>
  </body>
</html>
```

## 7. 정리

- ECMAScript2015 이전에는 모듈을 만들기 위해 즉시실행함수와 네임스페이스 패턴을 사용했다.

- 이후 각 커뮤니티에서 모듈 시스템 스펙이 나왔고 웹팩은 ECMAScript2015 모듈시스템을 쉽게 사용하도록 돕는 역할을 한다.

- 엔트리포인트를 시작으로 연결되어 었는 모든 모듈을 하나로 합쳐서 결과물을 만드는 것이 웹팩의 역할이다.

- 자바스크립트 모듈 뿐만 아니라 스타일시트, 이미지 파일까지도 모듈로 제공해 주기 때문에 일관적으로 개발할 수 있다.

- 웹팩의 로더와 플러그인의 원리에 대해 살펴보았고 자주 사용하는 것들의 기본적인 사용법에 대해 익혔다.
