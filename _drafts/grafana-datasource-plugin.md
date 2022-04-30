Grafana允許使用插件來引入本來不支援的資料來源，例如說想要用自己開發的api來取得資料時，就需要開發資料來源插件來擴充Grafana。

官方有提供很多範例，但大部分的範例所對應的版本都比較舊，編譯後放在Grafana上都會有一些問題，因此本文會從無到有建置一個datasource的plugin。

## 初始專案

Grafana最新版本的插件都已經改成使用TypeScript做開發語言、webpack當作打包工具及使用Jest做測試框架，因此這次的演繹也會使用這些技術。

### 設定package.json

1. 初始化`package.json`: `npm init -y`
1. 安裝TypeScript: `npm install typescript --save-dev`
1. 安裝webpack: `npm install webpack webpack-cli --save-dev`
1. 安裝Jest: `npm install jest --save-dev`
1. webpack解析TypeScript時須使用loader: `npm install ts-loader --save-dev`
1. Jest要可以測試TypeScript的程式: `npm install ts-jest --save-dev`

## Metadata

Grafana會以`plugin.json`及`README.md`的資訊來對外展示這個插件，如果這個插件是不公開的，那只需要在`plugin.json`中設定`type`、`name`及`id`即可，`README.md`可以不用寫。

註: 詳細的Metadata設定方式可以在[這裡](http://docs.grafana.org/plugins/developing/plugin-review-guidelines/#metadata)找到。

### plugin.json

在`src`資料夾中新建`plugin.json`檔案:

```js
// plugin.json

{
  "type": "datasource",
  "name": "Typescript Webpack Jest Demo",
  "id": "peterhpchen-demo-datasource"
}
```

* `type`: 插件的種類，目前分為三種: `panel`、`datasource`及`app`(本文所介紹的是`datasource`)。
* `name`: 此插件的名字，用以顯示在選單中供人選取。
* `id`: 這個插件的識別碼，需要照特定的格式撰寫: `[github username/org]-[plugin name]-[datasource|app|panel]`。

`datasource`類型的插件有`metrics`跟`annotations`這兩個特殊的參數:

* `metrics`: 讓panel使用的資料。
* `annotations`: 在Graph上的註解資料([參考](http://docs.grafana.org/reference/annotations/))。

本文只會講述`metrics`，因此最後的`plugin.json`會長得像是下面這樣:

```js
// plugin.json

{
  "type": "datasource",
  "name": "Typescript Webpack Jest Demo",
  "id": "peterhpchen-demo-datasource",

  "metrics": true,
  "annotations": false
}
```

## module.ts

`module.ts`是整個專案的入口檔，Grafana在執行插件時會從這裡取得對應的程式碼，在datasource插件中有下面幾個元件需要實作:

* `Datasource`(Required): 取得資料來源。
* `QueryCtrl`(Required): 在Panel中設定Metrics時的畫面。
* `ConfigCtrl`(Required): 在設定Datasource時的畫面。
* `AnnotationsQueryCtrl`: 設定註解的查找時的畫面。

除了`Datasource`外，其他三個都是AngularJS的Controller，每個都有View需要撰寫，分別對應到Grafana不同的配置畫面。

### 每個Controller在Grafana下的位置

這裡用[simple-json-datasource](https://github.com/grafana/simple-json-datasource)插件截圖來說明每個畫面在Grafana系統下的配置畫面。

#### QueryCtrl

![QueryCtrl](/assets/2018-08-08-grafana-datasource-plugin/QueryCtrl.png)

Panel的編輯模式中有個Metrics的tab可以做datasource的設置，這個Controller是每筆datasource的查詢所需要的資訊設置。

#### ConfigCtrl

![QueryCtrl](/assets/2018-08-08-grafana-datasource-plugin/ConfigCtrl.png)

整個Grafana系統設置Datasource時所需輸入的參數，像是連線、驗證等資訊。

#### AnnotationsQueryCtrl

![AnnotationsQueryCtrl](/assets/2018-08-08-grafana-datasource-plugin/AnnotationsQueryCtrl.png)

Dashboard的Settings中有個Annotations的tab，這個Controller就是在設置註解時所用的客製配置。

這三個Controller中這次只會用到前面兩個QueryCtrl及ConfigCtrl，AnnotationsQueryCtrl之後會跟Annotations一起介紹。

### 初始module.ts

我們這裡先用空的Class來配置`module.ts`，之後會介紹每個Class的開發方式。

```ts
// module.ts

class DemoDatasource {}
class DemoQueryCtrl {}
class DemoConfigCtrl {}

export {
    DemoDatasource as Datasource,
    DemoQueryCtrl as QueryCtrl,
    DemoConfigCtrl as ConfigCtrl,
};
```

這裡我們對外公布了`Datasource`、`QueryCtrl`及`ConfigCtrl`這三個必要的`Class`，而`Class`都是空的。

## 使用webpack建置專案

`module.ts`跟`plugin.json`就是插件的基礎構成，雖然現在各個功能都還沒有實作，不過已經可以編譯成Grafana的插件了，因此這節會使用webpack建置且部署到Grafana上。

### 建立webpack.config.js

新建webpack的配置檔，在`root`資料夾下新增一個`webpack.config.js`檔，下面會介紹如何撰寫配置檔。

#### 設定專案的輸入及輸出

輸入webpack的資料夾為`src`，要將建置好的程式碼放於`dist`資料夾下:

```js
// webpack.config.js

const path = require('path');
...

module.exports = {
  context: path.resolve(__dirname, 'src'),  // 入口目錄
  entry: './module.ts', // 入口檔案為module.ts
  output: {
    filename: 'module.js',  // Grafana將module.js視為入口檔案
  },
  ...
};
```

這裡有一點需要注意的是Grafana會找`module.js`當作插件的入口，所以名稱一定要是`module.js`。

#### 將plugin.json移至dist資料夾

由於`plugin.json`是靜態檔案，而且Grafana會認`plugin.json`這個名字的檔案當作Metadata，所以它不要經過webpack建置，但又要複製到目標資料夾，這需求我們可以使用`copy-webpack-plugin`來達成。

##### copy-webpack-plugin

`copy-webpack-plugin`可以幫助我們把靜態(不須編譯)檔案複製到目標資料夾。

```bash
npm install copy-webpack-plugin --save-dev
```

##### 在webpack.config.js中加入copy-webpack-plugin的配置

```js
// webpack.config.js

...
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  context: path.resolve(__dirname, 'src'),  // 入口目錄
  ...
  plugins: [
    new CopyWebpackPlugin([ // 將靜態檔案複製到目標資料夾中
      './plugin.json',
    ]),
  ],
};
```

這裡要注意`CopyWebpackPlugin`中的起始目錄是入口目錄，因為已經設定了`context`為`src`，所以在這裡不需要再加上`src/`的路徑。

#### 編譯Typescript

要將TypeScript編譯成一般的JavaScript，需要使用`ts-loader`讓webpack知道如何處理`.ts`檔。

```js
// webpack.config.js

...

module.exports = {
  ...
  module: {
    rules: [
      { // 將ts檔轉為js
        test: /\.tsx?$/,
        loader: 'ts-loader',
      },
    ],
  },
  ...
};
```

因為`ts-loader`在建置`.ts`檔前會去讀取`tsconfig.json`，藉此知道要怎麼建置`.ts`檔案，所以要在`root`資料夾下新增`tsconfig.json`檔。

```js
// tsconfig.json

{

}
```

內容先給個空物件就好，之後會再做調整。

### 新增npm指令執行webpack

在`package.json`的`scripts`中增加`dev`指令:

```js
// package.json

{
  ...
  "scripts": {
    "dev": "webpack --mode development",
    ...
  },
  ...
}
```

先使用`development` mode，因為`production` mode會用Uglify處理，這樣會很難了解webpack是如何編譯檔案的，因此開發階段請使用`development` mode。

接著使用`npm run dev`即可產生目標資料夾，裡面會有已建置好的檔案。

## 部署至Grafana

使用webpack建置好後會有個`dist`資料夾，只要將此資料夾複製到`grafana/data/plugins`下對應的id資料夾下，然後重啟`grafana-server`就可以載入plugin了。

![CopyToGrafana](/assets/2018-08-08-grafana-datasource-plugin/CopyToGrafana.png)

完成後可以在Datasource(`齒輪>Datasource>Add data source`)的選單中找到插件。

![FindDatasource](/assets/2018-08-08-grafana-datasource-plugin/FindDatasource.png)