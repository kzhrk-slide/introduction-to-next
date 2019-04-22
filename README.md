# React v15のサイトをNextに移行する

## 経緯

さくらVPSのコスト

## クラウド化・脱VPS

## senriu.comの構成

インフラ: さくらVPS
Webサーバ: nginx
DB: mongoDB
バックエンド: node.js (express.js)
フロント: React.js v15

## 新しい構成

Firebase Hosting
Clound Function
Next.js

## Next.jsとは

React.jsでつくられたSSRを容易にするフレームワーク。  
本来であれば、BFF層のNode.js上でReactDOM.renderによるHTML生成、返却を行う必要があるが、Next.jsはReact.jsで書かれたJSファイルを解析して静的HTMLを自動で出力してくれる。  
さらにルーティングのJSON返却もよきようにやってくれる。

### Next.jsを使わないSSR（react-router）

URLにアクセス /work/123  
URLのレンダリングに必要なデータをAPIから取得。ReactDOM.renderでHTMLを出力してクライアントサイドに返却  
別のURLにreact-routerのRouteコンポーネントを介してアクセス /work/456  
react-routerでHistory APIが更新、URLに該当するReact Componentを呼び出し。

### Next.jsを使ったSSR

URLにアクセス /work/123
Next.jsで書き出されたスタティックなHTMLを返却
別のURLにアクセス /work/456
Next.jsでHistory APIが更新、URLに該当するReact Componentが呼び出される

## Next.jsのセットアップ

### npm install

```
mkdir sample-next
cd sample-next
npm init -y
npm i -S react react-dom next
mkdir pages
```

### npm-scripts

package.jsonに下記のscriptsを定義する。

```json
...
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
...
```

npm devを実行すると、http://localhost:3000にサーバが立ち上がる。  
何もページを設定していないので、Next.jsのデフォルトの404ページが表示される。

### ページ作成

pages/index.jsを作成し、JSXを書く。

```js
const Index = () => (
  <div>
    <p>Hello World</p>
  </div>
)

export default Index
```

この状態でnpm devを実行するとHello Worldが出力される。

### ナビゲーション

トップページとは別にaboutページを作成する。

```js
const About = () => (
  <div>
    <p>This is the about page</p>
  </div>
)

export default About
```

http://localhost:3000/about にAboutページが表示される。  
トップページからAboutページへのリンクを追加する。  
通常のaタグではなく、nextに搭載されたLinkコンポーネントを使用する。

```js
import Link from 'next/link'

const Index = () => (
  <div>
    <p>Hello World</p>
    <Link href="/about">
      <a>About</a>
    </Link>
  </div>
)

export default Index
```

http://localhost:3000/ から http://localhost:3000/about に移動するとHTMLの再取得は行わず、aboutページの出力に必要なJSファイルのみが追加で読み込まれる。  
/aboutに直接アクセスするとスタティクなAboutページのHTMLが返却される。

### Components

components/Header.jsで各ページで共通利用するコンポーネントを作成する。

```js
import Link from 'next/link'

const headerStyle = {
  backgroundColor: '#ccc'
}

const Header = () => (
  <div style={headerStyle}>
    <ul>
      <li>
        <Link href="/">
          <a>Home</a>
        </Link>
      </li>
      <li>
        <Link href="/about">
          <a>About</a>
        </Link>
      </li>
    </ul>
  </div>
)

export default Header
```

/pages/index.jsと/pages/about.jsで/components/Header.jsを読み込む。

```js
import Header from '../components/Header'
import Link from 'next/link'

const Index = () => (
  <div>
    <Header></Header>
    <p>Hello World</p>
    <Link href="/about">
      <a>About</a>
    </Link>
  </div>
)

export default Index
```

### Layout

すべてのページでcomponents/Header.jsを読み込むのはメンテナブルではないので、layout/index.jsというレイアウトコンポーネントを定義する。

```js
import Header from './Header' // components/Header.jsからlayout/Header.jsに移動

const layoutStyle = {
  padding: 20,
  backgroundColor: '#eee'
}

const Layout = props => (
  <div style={layoutStyle}>
    <Header />
    {props.children}
  </div>
)

export default Layout
```

props.childrenはLayoutコンポーネントの子要素が展開される。

pages配下でlayout/index.jsを読み込む。

```js
import Layout from '../layout'

const Index = () => (
  <Layout>
    <p>Hello World</p>
  </Layout>
)

export default Index
```

### 動的ページ

URLにクエリパラメータをつけて情報を動的に書き出すWorkページを作成する。  
トップページに/work?id={int}のリンクリストを作成する。

```js
import Link from 'next/link'
import Layout from '../layout'

const Index = () => (
  <Layout>
    <ul>
      <li>
        <Link href="/work?id=1">
          <a>作品1</a>
        </Link>
      </li>
      <li>
        <Link href="/work?id=2">
          <a>作品2</a>
        </Link>
      </li>
      <li>
        <Link href="/work?id=3">
          <a>作品3</a>
        </Link>
      </li>
    </ul>
    <p>Hello World</p>
  </Layout>
)

export default Index
```

pages/work.jsを作成する。

```js
import { withRouter } from 'next/router'
import Layout from '../layout'

const Work = withRouter(props => (
  <Layout>
    <h1>Work</h1>
    <p>id: {props.router.query.id}</p>
  </Layout>
))

export default Work
```

witRouterでクエリーパラメータにアクセスし、idを出力する。

### URLをマスクする

クエリパラメータでの表示を避け、URLをシンプルにする。  
next/linkにasプロパティを付与するとルートをマスクすることができる。

```js
import Link from 'next/link'
import Layout from '../layout'

const Index = () => (
  <Layout>
    <ul>
      <li>
        <Link as="/work/1" href="/work?id=1">
          <a>作品1</a>
        </Link>
      </li>
      <li>
        <Link as="/work/2" href="/work?id=2">
          <a>作品2</a>
        </Link>
      </li>
      <li>
        <Link as="/work/3" href="/work?id=3">
          <a>作品3</a>
        </Link>
      </li>
    </ul>
    <p>Hello World</p>
  </Layout>
)
export default Index
```

URLマスクは、History APIのマスクしか行えないため、トップページからWorkページに遷移するときには問題がないがWorkページでリロードするとスタティックなHTMLが存在しないため404になる。

### express.jsでサーバを立てる

マスクされたURLに直接アクセスしたときにスタティックなHTMLを返却するためにはNode.jsでサーバを立てる必要がある。  
今回はフレームワークのexpress.jsを使用する。

```
npm i -S express
```

```js
const express = require('express')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app
  .prepare()
  .then(() => {
    const server = express()

    server
      .get('/work/:id', (req, res) => {
        app.render(req, res, '/work', {id: req.params.id })
      })
      .get('*', (req, res) => {
        return handle(req, res)
      })
      .listen(3000, err => {
        if (err) throw err
        console.log('> Ready on http://localhost:3000')
      })
  })
  .catch(ex => {
    console.error(ex.stack)
    process.exit(1)
  })
```

## Cloud Function



