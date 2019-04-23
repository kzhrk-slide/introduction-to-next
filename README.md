# Introduction to Next.js

## Next.jsとは

React.jsでつくられたSSRを容易にするフレームワーク。  
Next.jsを使わなければ、BFF層のNode.js上でURLエンドポイントを作成し、[react-dom/server](https://reactjs.org/docs/react-dom-server.html)を使って必要なReactコンポーネントを描画する必要がある。ページによってはコンポーネントに必要なpropsのデータをAPIから取得する必要もある。  
Next.jsではクライアントサイド・サーバーサイドの実装をあまり意識せず、ReactコンポーネントとAPIからのデータ取得を実装するだけでクライアントサイド・サーバーサイドに必要なJavaScriptを自動生成してくれる。

## Next.jsのセットアップ

### npm install

依存モジュールをinstallする。

```
mkdir sample-next
cd sample-next
npm init -y
npm i -S react react-dom next
mkdir pages
```

pagesディレクトリはNext.jsで定められたディレクトリ名になっている。  
Next.jsのCLIのnextコマンドを実行したときにpagesディレクトリ配下に置かれたJavaScriptファイルがビルドの対象ファイルになる。

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

npm devを実行すると、 http://localhost:3000 にサーバが立ち上がる。  
pagesディレクトリ配下に何もページを設定していないので、Next.jsのデフォルトの404ページが表示される。

### ページ作成

トップページとなるpages/index.jsを作成し、JSXを書く。

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
次にトップページからAboutページへのリンクを追加する。  
このとき通常マークアップではaタグを使うが、nextに搭載されたLinkコンポーネントを使用する。

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

開発ツールを開いた状態でhttp://localhost:3000/ から http://localhost:3000/about に移動するとHTMLの再取得が行われず、aboutページの出力に必要なJSファイルのみが追加で読み込まれるのが確認できる。  
/aboutに直接アクセスするか、/aboutでリロードをするとスタティクなAboutページのHTMLが読み込まれる。

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

### fetchでデータを取得

APIからデータを取得してIndexコンポーネントのPropsに反映する。  
クライアントサイドとサーバーサイドでfetchが実行できるisomophic-unfetchを使用する。  
公開されているダミーAPI`http://dummy.restapiexample.com/api/v1/employees`を利用して、返却された配列を出力する。

```js
import Link from 'next/link'
import Layout from '../layout'
import fetch from 'isomorphic-unfetch'

const Index = props => (
  <Layout>
    <h1>Users</h1>
    <ul>
      {props.data.map(employee => (
        <li key={employee.id}>
          <Link href={`/employee/${employee.id}`}>
            <a>{employee.employee_name}</a>
          </Link>
        </li>
      ))}
    </ul>
  </Layout>
)

Index.getInitialProps = async function() {
  const res = await fetch('http://dummy.restapiexample.com/api/v1/employees')
  const data = await res.json()

  return {
    data
  }
}

export default Index
```

URLにアクセスしてソースコードを確認すると、APIから取得したデータもHTMLタグ内に出力され、SSRが自動で行われていることが確認できる。

次にURLのパス（/employee/:id）を判定して動的URLからAPIのリクエスト先を変更してAPIのデータを表示する。

```js
import Layout from '../layout'
import fetch from 'isomorphic-unfetch'

const Employee = props => (
  <Layout>
    <h1>{props.data.employee_name}</h1>
    <dl>
      <dt>id</dt>
      <dd>{props.data.id}</dd>
      <dt>age</dt>
      <dd>{props.data.employee_age}</dd>
    </dl>
  </Layout>
)

Employee.getInitialProps = async function(context) {
  const { id } = context.query
  const res = await fetch(`http://dummy.restapiexample.com/api/v1/employee/${id}`)
  const data = await res.json()

  return {
    data
  }
}

export default Employee
```

次に、expressのgetメソッドでエンドポイントを作成してGETリクエストを解決する。

```js
...
    server
      .get('/employee/:id', (req, res) => {
        app.render(req, res, '/employee', {id: req.params.id})
      })
...
```

