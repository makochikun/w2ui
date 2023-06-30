# 変更点と追加機能についての自分用メモ

はじめに、この素晴らしいライブラリーを作成されたvitmalina様に尊敬と感謝の意を表したいと思います。

私は日本人で英語が得意ではなく、GitHubの使い方も正直言っていまいちよくわかっていません。もし作成者のvitmalina様や他の方にご迷惑をおかけする様な事になっていたら教えてください。すぐに削除します。

## w2gridに加えた変更点

### （１）urlオプションに関数を設定出来るように変更

Electronのメインプロセスでsocket.ioを経由してサーバーからデータを取得してレンダープロセスで受け取ったデータをグリッドに反映させたかった為、urlに関数を設定したかったので変更を行う事にした。

urlに設定する関数には本来Fetchでサーバーに送られるPOSTデータがオブジェクト型で渡される。

言葉で説明するのが得意ではないのでラフなソースサンプル書いてみます。

``` js
let grid = w2grid({
  url: dataSource,
  postData: [
    cmd: 'test'
  ],
  sortData: [
    { field: 'firstName', direction: 'asc' },
    { field: 'lastName', direction: 'asc' }
  ],
  columns: [
    { field: 'firstName', type:text, sortable: true },
    { field: 'lastName', type:text }
  ]
})

function dataSource(request) {
  console.log(request)
  return new Promise(async (res,rej) => {
    server( request ).then((result)=>{
      console.log(result)
      res({status:'success', data: result})
    }).catch((result)=>{
      rej({status:'error', statusText: result})
    })
  })
}

---
{
  cmd: 'test',
  sort: {
    {field: 'firstName', direction: 'asc'},
    {field: 'lastName', direction: 'asc'}
  },
  limit: 100,
  offset: 0
}

{
  status:'success',
  total: 655,
  records: [
    { firstName:'Nakamura', lastName:'Masaaki' },
    { firstName:'Nakamura', lastName:'Yuki' },
    { firstName:'Minase', lastName:'Naoto' }
  ]
}
```

上記の例を参考にしてください。

- (2023-06-27) この時点ではデータを取得するところまでしかテストしていない。保存や削除については徐々にテストして解決していくつもり。

---

## w2formに加えた変更点

### （１）カラムにスタイルを設定出来るようにした

``` js
'Tab title': {
  type: 'tab',
  fields: {
    'Group title1': {
      type: 'group',
      titleStyle: 'font-size: 1em; font-weight: bold;',
      style: 'width: 100%',
      column: 0,
      columnStyle: 'width: 80%',
      fields: {
        'request': { type: 'textarea', required: false, label: '', span: 0 }
      }
    },
    'Group title2': {
      type: 'group', span: 0,
      titleStyle: 'font-size: 1em; font-weight: bold;',
      column:1,
      columnStyle: 'width: 20%; min-width: 150px;',
      fields: {
        'request': { type: 'textarea', required: false, label: '', span: 0 }
      }
    }
  }
```

### （２）フォームの閲覧モードを追加した

各々のフィールドを個別にリードオンリーにするメソッドがあるのでそれを使えば良いのだが、面倒なので追加した。

``` js
const form = new w2form({
  name: 'mainForm',
  header: 'Header',
  readOnly: true,   //  true or false
  fields: { 
  }

----

form.formReadOnly(false) // true = change to read only mode 
                         // false = change to editable mode

```

### （３）リセットメソッドを追加した

ふと使いたくなる事が多いので追加した
単純にform.originalをform.recordにObject.assignを使ってコピーしたあとにrefresh()してるだけのメソッド

``` js
form.reset()
```

### （４）Clear時にヘッダーがクリアされないバグを修正

---

## w2tabsに加えた変更点

### （１）タブにバッジをつけれるように機能を追加した

``` js
const tabs = new w2tabs({
  active: 'all',
  style: 'padding-top: 10px',
  tabs: [
    { id: 'tab1',
      text: 'all' 
    },
    { id: 'tab2',
      closable: true, 
      text: 'prepare' 
    },
    { id: 'tab3',
      text: 'waiting', 
      badge: { // Only set the default badge settings for the tabs you want to change
        counter:0,        // number to display on the badge
        bgColor:'blue',   // background color of the badge 
        time:'0.5s',      // Flashing time per time 
        blink:'3'         // number of blinks
      } 
    }
  ],
})

tabs.setBadge('tab1', {counter:2, bgColor:'pink', blink:5})
tabs.setBadge('tab2', 3)
tabs.setBadge('tab3', 1)
```

#### setBadge( [タブID], [number] or [object])

バッジは数値のみを設定出来るようになっていて最高で99までの表示制限をいています。99以上の数値を設定しても表示されるのは99です。

０を設定するとバッジは消えます。

blinkは回数を数値で指定するか'infinite'等も設定出来ます。

バッジ機能追加に伴いCloseボタンの位置をタブの右側から左側に変更しました。
