## 問題情報
 
- 問題URL: https://neetcode.io/problems/time-based-key-value-store/question?list=neetcode150
- 難易度: Medium
- 制約 (n の範囲など):

  - 1 <= key.length, value.length <= 100
  - keyvalue小文字の英字と数字のみを含めてください。
  - 0 <= timestamp <= 10^7
  - すべてのタイムスタンプはset厳密に増加しています。

## メモ
- setの際のタイムスタンプは増加する、
- getは指定するタイムスタンプ以下かつ最大のタイムスタンプに対するvalueの値を取得、指定したタイムスタンプ以下がない場合は空文字を返す
- 一つのkeyに対してvalueが複数存在する、タイムスタンプレベルだと1:1になる
- 同じタイムスタンプで複数のkeyが存在することはない(setのタイムスタンプは増加するので)
  
## 1. 素直な解法（Brute Force）

### 方針　

- データ構造はハッシュマップ作成
- get
　- keyに対するタプルリストを引っ張ってくる
　- その中から最後のリストから順にtimestamp_prev <= timestampの条件を見てtrueになったらそのタプルのvalueを返す
- set
  - keyに対応するタプルリストを引っ張ってきてそこにpushする
  - ない場合はハッシュマップに新規にkeyとそれに対するタプルリストを追加する

 
### データ構造

```
{
  key: [(timestamp, value), ....]
}
```

### 時間計算量
#### get 
timestampの最大数10^7が最大計算量 -> O(T)
#### set
pushするだけなのでO(1)

O(T)

### 空間計算量
O(T)

## 2. 最適化した解法

### メモ
空間計算量はO(T)から減らすことはできないと思う、なぜならtimestampに応じた値を保持しないといけないからその分のメモリは必要になる

減らせるとしたら時間計算量
getの計算量を削減することができると思う
setのtimestampは増加するという性質上、タプルリストはtimestampだけ見れば昇順にソートされた配列になっているはず
timestamp_prev <= timestampの条件の境界値を見つければ良い

### 方針
- データ構造はそのまま
- getの計算量を減らすために探索を全探索から二分探索での条件を満たす境界値を見つけるようにする、それにより計算量がO(T) -> O(logT)に削減される
 
- 時間計算量: O(logT)
- 空間計算量: O(T)

## 3. 図解 疑似コード
 
<!-- 図（データ構造の変化、フローなど）を貼る or 手書きの画像を添付 -->
**get関数に絞る**
get("hoge", 21)を呼び出す

```
//  ハッシュマップの現在の状態 
hashMap = {"hoge":  [ (1, "1"), (2, "test"), (4, "test2") .... ]}

get(key, t) {
  
  if key not in hashMap:
    return ""

  tList = hashMap[key]

  // 二分探索のl, rポインタ初期化
  l, r = 0, len(tList)-1
  while(l <= r) {
    mid = left + (right-left)//2
    pt, v = tList[mid]
    if pt <= t:
       l = mid + 1
    else:
       r = mid - 1
    
    if l == r:
       pt, v = tList[l]
       return v
  }
  return ""
}
```

## 4. テストケース設計
 **getに絞る**

- timestampに完全一致するデータが有る場合をその値を返す:
hashMap = {"hoge":  [ (1, "1"), (2, "test"), (4, "test2"), ("test21", 21), ("test25", 25), .... ]}
input -> get("hoge", 21)
output -> "test21"

- timestampに完全一致するものがないがそれ以下のデータがある場合はその中の最大値を返す:
hashMap = {"hoge":  [ (1, "1"), (2, "test"), (4, "test2"), ("test20", 20), ("test25", 25), .... ]}
input -> get("hoge", 21)
output -> "test20"

- timestampに完全一致するものがないがそれ以下のデータがある場合はその中の最大値を返す:
hashMap = {"hoge":  [ (1, "1"), (2, "test"), (4, "test2"), ("test20", 20), ("test25", 25), .... ]}
input -> get("hoge", 21)
output -> "test20"

- timestampに完全一致するものがないかつがそれ以下のデータもない場合空文字を返す:
hashMap = {"hoge":  [ (5, "1"), (2, "test"), (8, "test2"), ("test20", 20), ("test25", 25), .... ]}
input -> get("hoge", 4)
output -> " "

- 現在のstoreに保存されているあるkeyに対するデータが1件の時にそのデータがtimestamp以下の場合、その値を返す:
hashMap = {"hoge":  [ (5, "test5") ]}
input -> get("hoge", 7)
output -> "test5"

- 現在のstoreに保存されているあるkeyに対するデータが1件の時にそのデータがtimestampより大きい場合、空文字を返す:
hashMap = {"hoge":  [ (8, "test8") ]}
input -> get("hoge", 7)
output -> " "

- 現在のstoreに保存されているあるkeyに対するデータが0件の時、空文字を返す:
hashMap = {}
input -> get("hoge", 1)
output -> " "

## 5. 実装
```py
class TimeMap:

    def __init__(self):
        self.hashMap = {}

    def set(self, key: str, value: str, timestamp: int) -> None:
        if key not in self.hashMap:
            self.hashMap[key] = []
        self.hashMap[key].append((timestamp, value))

    def get(self, key: str, timestamp: int) -> str:
        if key not in self.hashMap:
            return ""

        l, r = 0, len(self.hashMap[key])-1
        result = ""
        while l <= r:
            mid = l + (r-l)//2
            pt, v = self.hashMap[key][mid]
            if pt <= timestamp:
                result = v
                l = mid + 1
            else:
                r = mid - 1
        return result
```
 
- 実装リンク: https://neetcode.io/problems/time-based-key-value-store/history

## 6. テスト結果
 
- [x] ローカルテスト全パス
- [x] LeetCode 提出 AC
## 7. 振り返り
 
### 詰まったポイント
- 疑似コードの段階で考慮できてなかったことがあった
timestamp_prev <= timestampの条件に一致した時にvalueを一時変数に格納するということ
これをしないために、せっかく条件に一致したのにそのあと探索をまた続けて、その結果他に一致するものがなかったときに
空文字を返すことになってしまった。
最初これに気が付かなかったためwhile文のなかでif l > rのような余計な分岐を追加してしまった

### 次回同系統の問題で意識すること:
疑似コードをしっかりテストすること
今回のrunして初めて気づいたのでこれは疑似コードの段階でテストケースに対してどういう動きになるかをしっかり確認していれば事前に気がついた
そもそもやはりテストコードを書くべきだった

## チェックリスト
 
- [x] 素直な解法を書いた
- [x] 最適化解法を書いた
- [x] 図を書いた
- [x] テストケースを設計した
- [x] 実装した
- [ ] テストコードを書いてパスした
- [x] 提出した
- [ ] 振り返りを書いた
