上次已經建立一個Grafana Datasource Plugin的基礎專案，也已經部署到Grafana中，接著就來介紹要如何寫測試程式。

## 撰寫測試

專案中已有`module.ts`，本文將使用它來演繹如何撰寫測試。

* 在`src`資料夾新增`specs`資料夾
* `specs`中新增`module.jest.ts`檔案

## module.jest.ts

Datasource Plugin基本的元件有三個: `DataSource`、`ConfigCtrl`及`QueryCtrl`，現在就來測試程式中是否有這三個元件。

```ts
...
import * as module from '../src/module';
...

describe('module', () => {
  describe('Datasource', () => {
    it('should exist', () => {
      expect(module.Datasource).toBeDefined();
    });
  });
  describe('QueryCtrl', () => {
    it('should exist', () => {
      expect(module.QueryCtrl).toBeDefined();
    });
    ...
  });
  describe('ConfigCtrl', () => {
    it('should exist', () => {
      expect(module.ConfigCtrl).toBeDefined();
    });
  });
});
```

現在測試程式寫好了，再來是設定Jest。

## 設定Jest

在`root`資料夾裡新增`jest.config.js`，Jest預設會讀這個設定檔。

### jest.config.js

上次已經安裝了`jest`與`ts-jest`，現在要先讓Jest可以轉換TypeScript。

### 轉換Typescript

```js
module.exports = {
  verbose: true,
  testURL: "http://localhost/", // 使用jsdom當測試環境會有localStorage問題，將testURL改成不是about:blank來解決問題
  transform: {
    "^.+\\.tsx?$": "ts-jest"  // 使用ts-jest對.ts檔做轉換
  },
  testRegex: "(\\.|/)(jest)\\.(jsx?|tsx?)$",  // 對應所有測試的檔案
  moduleFileExtensions: [ // 專案中module的副檔名，要加上ts
    "ts",
    "tsx",
    "js",
    "jsx",
    "json",
    "node"
  ]
};
```

* `verbose`: 輸出詳細訊息。
* `testURL`: Jest使用jsdom模擬瀏覽器環境做測試，預設Jest是用`about:blank`當作jsdom環境的URL，但這樣的設定會出現**SecurityError: localStorage is not available for opaque origins**問題，將其改為`http://localhost/`以解決此問題。
* `transform`: 將所有的`.ts`檔丟給`ts-jest`處理。
* `testRegex`: 測試檔案的命名規則是`[name].jest.ts`，這裡設定將符合規則的檔案餵給Jest做測試。
* `moduleFileExtensions`: Jest預設不支援TypeScript，必須要把TypeScript的副檔名加入，否則Jest不會讀`.ts`檔。

### package.json中加入測試指令

在`scripts`中加入`test`指令。

```js
{
  ...
  "scripts": {
    ...
    "test": "jest"
  },
  ...
}
```

## 執行測試

剛剛已經寫好了測試程式，也設置了Jest的測試環境，現在我們可以執行測試了。

```bash
npm run test
```

![TestResult](/assets/2018-08-12-grafana-datasource-plugin-test/TestResult.PNG)

執行後可以看到所有的測試都通過。

## 更進一步

現在已經可以確定`module.ts`的確曝露出`Datasource`、`QueryCtrl`及`ConfigCtrl`，但可以更精確的檢查曝露出來的元件是否是Grafana期望的類別。

### grafana-sdk-mocks

Grafana有提供[SDK Mocks](https://github.com/grafana/grafana-sdk-mocks)，有Plugin常用的類別定義，接著將用Mocks裡的定義檔來判斷Module曝露的`QueryCtrl`是否為期望的類別。

* 安裝`grafana-sdk-mocks`: `npm install grafana/grafana-sdk-mocks --save-dev`。
* [Typings](https://github.com/grafana/grafana-sdk-mocks#typings)中有介紹如何在TypeScript中使用SDK Mocks。
* SDK Mocks基本對應到[Grafana中的public](https://github.com/grafana/grafana/tree/master/public)，這是整個Grafana的程式碼所在的資料夾。
* 在[app/plugins/sdk.ts](https://github.com/grafana/grafana-sdk-mocks/blob/master/app/plugins/sdk.ts)裡可以看到`QueryCtrl`的定義檔位置。
* 使用[app/features/panel/query_ctrl.ts](https://github.com/grafana/grafana-sdk-mocks/blob/master/app/features/panel/query_ctrl.ts)來取得`QueryCtrl`的類別定義檔，之後的測試會使用這個定義檔作依據。

### 撰寫測試程式

在`module.jest.ts`中依照[Typings](https://github.com/grafana/grafana-sdk-mocks#typings)的說明引入定義檔。

```ts
// specs/module.jest.ts

///<reference path="../node_modules/grafana-sdk-mocks/app/headers/common.d.ts" />

...
```

在`QueryCtrl`中多加一個測試，判斷`module.QueryCtrl`是否是`QueryCtrl`的實體。

```ts
// specs/module.jest.ts

...
import {QueryCtrl} from 'app/plugins/sdk';

describe('module', () => {
  ...
  describe('QueryCtrl', () => {
    ...
    it('should inherit QueryCtrl', () => {
      expect(new module.QueryCtrl({},{})).toBeInstanceOf(QueryCtrl);
    });
  });
  ...
});
```

### 更新Jest設定檔

我們在測試程式中的`QueryCtrl`是由[Triple-Slash Directives](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html)所參照的路徑引入，Jest本身並不知道這件事情，所以要在設定檔中告訴Jest這件事情。

```js
// jest.config.js

module.exports = {
  ...
  moduleNameMapper: {
    'app/plugins/sdk': '<rootDir>/node_modules/grafana-sdk-mocks/app/plugins/sdk.ts', // TypeScript的Triple-Slash參考需要用Mapper讓jest知道怎麼對應
  },
  ...
};

```

接著因為我們在`transform`中設定所有的ts檔案都要由`ts-jest`做轉換，但是對於外部引入的`grafana-sdk-mocks`是不需要做轉換的，因此要把它排除在轉換的檔案中。

```js
// jest.config.js

module.exports = {
  ...
  transformIgnorePatterns: [
    '<rootDir>/node_modules/(?!grafana-sdk-mocks)', // TypeScript的module不能被轉換，否則會轉換錯誤
  ],
  ...
};
```

### 修改module.ts

這時執行測試的話，新增的測試會出錯。

![TestFail](/assets/2018-08-12-grafana-datasource-plugin-test/TestFail.PNG)

原因是`module.ts`的`QueryCtrl`並不是`QueryCtrl`的實體，因為其沒有繼承`QueryCtrl`，要讓測試通過就要修改程式。

```ts
// src/module.ts

import { QueryCtrl } from "app/plugins/sdk";

...
class DemoQueryCtrl extends QueryCtrl {}
...
```

這時執行測試就會成功了。

![TestPass](/assets/2018-08-12-grafana-datasource-plugin-test/TestPass.PNG)

### Webpack建置失敗

測試成功後回去使用Webpack建置專案，發現專案會失敗，因為Webpack找不到`app/plugins/sdk`這個module，這個module只是個`grafana-sdk-mocks`所產生的定義，當建置完部署至Grafana系統後，就不再需要`grafana-sdk-mocks`了，因為真正的Grafana中已有這程式的實作，所以在建置時需要將其排除。

```js
// webpack.config.js

...

module.exports = {
  ...
  externals: [  // 由sdk mocks引入的module不需要打包，由外部引入即可
    'app/plugins/sdk',
  ],
  ...
};
```

### 部署後失敗

在排除`app/plugins/sdk`後可以正常建置了，但是在部署後Grafana載入Plugin時會報錯。

![PluginError](/assets/2018-08-12-grafana-datasource-plugin-test/PluginError.PNG)

從錯誤訊息可以知道`app/plugins/sdk`引入失敗，這是因為webpack在打包時預設會用全域的方式引入外部module，可是在Grafana中並不是用全域的方式，因此需要將建置的module引入外部module的方式改為`amd`。

```js
...

module.exports = {
  ...
  output: {
    ...
    libraryTarget: 'amd'  // Grafana是用amd的方式引入外部module
  },
  ...
};

```

這樣測試、建置及部署都正常了。

## 參考資料

* [Webpack中libraryTarget跟external的關係](https://segmentfault.com/a/1190000012113011#articleHeader3)