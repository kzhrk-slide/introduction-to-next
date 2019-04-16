# React v15のサイトをNextに移行する

## 経緯

さくらVPSのコスト

## クラウド化とサーバーレス化

## senriu.comの構成

インフラ: SAKURA VPS
サーバー: nginx
DB: mongoDB
バックエンド: node.js (express.js)
フロント: React.js v15

## 新しい構成

Netrify: ホスティング
Cloud Function: BFF
Next.js

## Next.jsとは

reactでつくられたSSRを容易にするフレームワーク。  
本来であれば、BFF層のNode.js上でReactDOM.renderによるHTML生成、返却を行う必要があるが、Next.jsはReact.jsで書かれたJSファイルを解析して静的HTMLを自動で出力してくれるし、ルーティングのJSON返却もよきようにやってくれる。

## SSRの一例

## Next.jsの設定

npmの依存モジュール
ディレクトリ構成


## Cloud Function



