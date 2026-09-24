# hazkey-engineering-dictionary

[hazkey-community](https://github.com/presire/hazkey-community)の工学用語辞書アセットです。  
機械工学・電気工学・電子工学・情報工学・情報系サービス名・プログラム言語・データベース名・建築学・土木工学等の専門用語/名称のうち、  
**変換エンジン単体では一発変換できないもの**だけを収録します。  

hazkey-serverは、この辞書をAzooKeyKanaKanjiConverterの**読み取り専用の補助LOUDS辞書 (supplemental dictionary)** として、  
ID `engineering` で読み込みます。  
システム辞書 (`azooKey_dictionary_storage`) と住所辞書 (`hazkey-address-dictionary`) の内容は変更しません。  
設定UIの[変換]タブにある[工学用語]チェックボックスでON / OFFでき (既定はOFF)、[住所]とは独立に切り替わります。  

## 構成

```
data/engineering_entries.tsv      ソース (読み<TAB>表記<TAB>品詞、読みはカタカナ、859件)
EngineeringDictionary/louds/
  charID.chid                     照合用スタンプ (後述)
  [XXXX].louds / .loudschars2     先頭カナ別シャード
  [XXXX]N.loudstxt3               エントリ本体
NOTICE                            データ出典
```

`EngineeringDictionary/` は `/usr/share/hazkey-community/EngineeringDictionary` にインストールされます。  
環境変数 `HAZKEY_ENGINEERING_DICTIONARY` で別のディレクトリを指定できます。  
(指定した場合はその値だけを使い、実在するディレクトリでなければ工学用語辞書を無効にします)  
`data/` (ソースTSV) はインストールされません。  

## TSVスキーマ

`#` で始まる行はコメントです。  
データ行は3列です。  

| 列 | 内容 |
|---|---|
| `reading` | 読み (カタカナと `ー` のみ、20文字以内、全文字がシステム辞書の `charID.chid` に収録) |
| `surface` | 表記 (英字表記も可。例: `IoCコンテナ`) |
| `pos` | 品詞: `proper` / `org` / `general` |

`pos` は次のCIDに対応します。(`lcid = rcid = CID`)  
新規CID / MIDは、追加しません。  

| `pos` | CID | 用途 |
|---|---|---|
| `proper` | `固有名詞 = 1288` | 製品・技術・言語名等 |
| `org` | `固有名詞組織 = 1292` | 企業・サービス名等 |
| `general` | `一般名詞 = 1285` | 一般の工学用語 |

MIDは、`一般 = 501`、スコアは `-15.5` です。(`DicdataStore.threshold = -17` を下回らせない)  
同じ読み・表記でもCIDが異なる要素は、それぞれ別のエントリとして残します。  

## 収録基準

**有名な用語・名称を優先**してキュレーションした候補に対し、**厳格 N-best**で選別します。  
権威読み付きの候補をsupplemental抜きのbase converterに通し、補正なしの単回変換 (`N_best=30`、誤り訂正なし、学習なし、特殊候補なし、Zenzai off) で  
**候補一覧のどこにも正しい表記が出ないものだけ**を収録します。  
一発変換できる語 (例: `Python`) は収録しません。読みは推測しません。  
選別はオフラインで1回だけ行い、実行時には判定しません。  

選別時のbase converterは、`presire/AzooKeyKanaKanjiConverter@0ad0617` + `azooKey_dictionary_storage@4d41852` です。  

## charID.chid照合スタンプ

補助辞書はシステム辞書と `charID.chid` (文字→IDの対応表) を共有します。  
LOUDSのノードindexはこの対応表に依存するため、**ビルド時と実行時で対応表が異なると、まったく無関係な語が引かれます**。  

`EngineeringDictionary/louds/charID.chid` は、ビルドに用いたシステム辞書の `charID.chid` と同一内容のコピーです。  
`DicdataStore` は、起動時にこれをシステム辞書側と突き合わせ、一致しなければ**工学用語辞書のみを無効化**します。(通常変換と住所辞書は継続します)  
ディレクトリ欠落・シャード破損時も同様に縮退します。  

## 再生成

`data/engineering_entries.tsv` またはシステム辞書を更新した場合は、hazkey-community側のdriftテストを再生成モードで実行します。  

```sh
cd hazkey-community/hazkey-server
HAZKEY_ENGINEERING_DICTIONARY_REGENERATE=1 \
  swift test --traits ZenzaiSupport --filter EngineeringDictionaryBuildTests
```

再生成モードなしで同じテストを実行すると、commit済みのLOUDSがTSVからの新規ビルドとバイト一致するかを検査します。(CIのドリフト検査)  
非変換判定 (上記のN-best選別) は、CIでは実行しません。  

## データ出典・ライセンス

出典の詳細は、[NOTICE](./NOTICE)を参照してください。  
主な読み・表記の出典は、SudachiDict (Apache-2.0) と NEologd seed (Apache-2.0) です。  
share-alike条件のデータ (Wikipedia、JMdict、COMPDIC等) は使用していません。  

本リポジトリのドキュメント部分は、hazkey-community本体と同じ[MIT License](https://github.com/presire/hazkey-community/blob/main/LICENSE)に従います。  
