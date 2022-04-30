上次已經開發了一個最基本的Grafana datasource plugin，也可以寫測試程式了，這次要為這個插件增加設置頁面的view及controller。

## Config頁面

Config頁面是用來設置datasource級別下的設定，例如說這個datasource的連線、驗證，或是這個datasource下的全域設定，它會在Configuration的Datasource裡做配置。

![DatasourceConfig](/assets/2018-08-10-grafana-datasource-plugin-configctrl/DatasourceConfig.png)

這裡會顯示你設定過的Datasource，點擊欲設定的Datasource就會進入Config頁面，或是想要增加新的Datasource可以按Add data source按鈕同樣也會進入Config頁面。

![GoConfig](/assets/2018-08-10-grafana-datasource-plugin-configctrl/GoConfig.png)

下面藍色框中的就是在ConfigCtrl中要開發的頁面，可以看到主要是連線、驗證還有全域變數的設定。

![ConfigPage](/assets/2018-08-10-grafana-datasource-plugin-configctrl/ConfigPage.png)

## 建立客製的Datasource設定頁面

接著要撰寫ConfigCtrl的HTML，在這之前有兩個東西要先介紹，Grafana提供的styles及components。

### 內建的components

有些設定是大部分的datasource都需要的，像是連線設定，Grafana就會將它包成components，方便不同的插件可以重複利用，我們也可以在自己的插件上使用這些components做開發。

#### datasource-http-settings

這個datasource-http-settings裡面有包好連線時常用的設定，它的定義在[ds_edit_ctrl.ts](https://github.com/grafana/grafana/blob/master/public/app/features/plugins/ds_edit_ctrl.ts)中，有提供三個參數:

* current: 設定資料的model。
* suggestUrl: 建議的連線URL。
* noDirectAccess: true: 不開啟Access的選項，一律使用Proxy連線，false: 開啟Access選項。

### 內建的styles

Grafana有提供styles給客製的Plugin使用，使用內建的styles不僅可以使得外觀像是原生的功能一樣，也可以在視窗做放大縮小時做對應的配置，十分的方便。

### config.html

範例的config使用datasource-http-settings及一個客製的設定欄位來演繹。

```html
<!-- src/partials/config.html -->
<datasource-http-settings
  current="ctrl.current"
  suggest-url="http://localhost:8080"
></datasource-http-settings>

<h3 class="page-heading">Custom Header</h3>
<div class="gf-form">
  <span class="gf-form-label width-7">Custom Input</span>
  <input class="gf-form-input max-width-21" type="text" ng-model='ctrl.current.jsonData.customInput' placeholder="custom input" required></input>
</div>
```

* 每個不同的Controller使用的View都是放在src/partials資料夾。
* 使用datasource-http-settings讓使用者設定連線、驗證。
* config的資料model為ctrl.current(在Controller中是this.current)，只有ctrl.current的資料會被Grafana帶到其他地方(ex: 查詢頁面)做使用。
* 客製欄位custom input。
* page-heading class: 大項的標題樣式。
* gf-form class: 表單中每欄的樣式。
* gf-form-label: 欄位的標題樣式。
* gf-form-input: 輸入框的樣式。

## Controller

ConfigCtrl是一個AngularJS的Controller，它有一個View，現在來看Controller要怎麼寫。

### templateUrl

在Controller的Class中，static的templateUrl是拿來定義這個Controller的View是哪隻html檔案。

```ts
// src/config_ctrl.ts

export default class DemoConfigCtrl {
  static templateUrl: string = 'partials/config.html';
}
```

為了寫得更加結構化，將DemoConfigCtrl從module.ts中獨立出來變成src/config_ctrl.ts。

### model

config的資料放在this.current中，在設定完成後，current中的property會帶到datasource的instanceSettings裡，就可以使用config的設定來取得資料。

#### jsonData

this.current中是存放內建components(例如datasourceHttpSettings)的資料，而this.current.jsonData才是存放客製的設定資料。

### 建構子

在Controller的建構子中要設定預設值。

```ts
// src/config_ctrl.ts

export default class DemoConfigCtrl {
  ...
  current: {
    jsonData: any
  };
  
  constructor() {
    this.current.jsonData.customInput = this.current.jsonData.customInput || 'default';
  }
}
```

* customInput的預設值是default。