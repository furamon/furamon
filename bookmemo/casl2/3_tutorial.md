# 実践
- とにかく指標レジスタを制すること。死んでおぼえろ
- ループの最初(初期値処理)と最後(終了処理)をチェックする。実務でも重要

- サブルーチンからメインルーチンのデータは直接見られない。なので工夫がいる
    - レジスタに入れて、サブルーチンで見てから戻る
    - 配列の先頭の絶対番地を「LAD GR1,DATA」などとして入れてロード
    - パラメータを配列に入れてから、配列の先頭アドレスをレジスタに入れて渡す(頻出)
    - スタックを経由させてデータやアドレスを渡す

- ソート処理はアルゴリズム=プログラミングの基本。覚えよう

## 問題解答欄

### 問3−1
```
MIN   START
      LD GR1,=1 ; ループ判定初期化
      LD GR0,DATA ; 周回数目のデータロード
LOOP  CPA GR0,DATA,GR1 ; DATA番地のデータは今の最大値より大きいか？
      JMI L1 ; 現在値<=データならスキップ
      JZE L1
      LD GR0,DATA,GR1 ; 最小値書き換え
L1    LAD GR1,1,GR1 ; 周回数加算
      CPA GR1,N ; 比較し終わった？
      JMI LOOP ;
      ST GR0,ANS ; 結果格納
      RET
DATA  DC 1,7,3,4
N     DC 4
ANS   DS 1
      END
```

### 問3-2
イ:配列の先頭番地がGR2。GR2が現在最小値のGR0より小さいならGR2番地のデータでGR0を更新

### 問3−3
|GR0|	比較用の最小値|
|GR1|	データ配列番地|
|GR2|	比較範囲の先頭番地|
|GR3|	比較終了判定値|
|GR4|	比較する値のポインタ|
|GR5|	比較範囲の終端番地|

```
MAIN  START
      LD GR1,=0
      LAD GR0,DATA 
      ST GR0,TABLE,GR1 ; データ番地をTABLEに
      ADDA GR1,=1
      LD GR0,N
      ST GR0,TABLE,GR1 ; データ個数をTABLEに
      LAD GR1,TABLE ; データ番地初期化
      CALL SORT ; ソートルーチン開始
      RET
DATA  DC 3,4,1,7 ; るんっ♪
N     DC 4 ; データ個数
TABLE DS 2
      END
SORT  START
      LD GR2,0,GR1 ; データ番地をGR2に持ってくる
      LD GR3,GR2 ; ループ終了位置計算準備
      ADDL GR3,1,GR1 ; データ番地にデータ個数を足して配列の最後を出す
      LD GR5,GR3 ; 配列の最後=ループ終端をGR5に設定
      SUBA GR3,=1 ; ループ回数設定
LOOP1 LD GR4,GR2 ; 比較ポインタ設定
      LD GR0,0,GR2 ; 比較先頭データ設定
LOOP2 ADDA GR4,=1 ; 比較ポインタカウント
      CPA GR4,GR5 ; 比較ループ終わった？
      JZE LEND
      CPA GR0,0,GR4 ; 先頭データと各要素を順繰り比較
      JMI LOOP2 ; 要素が大きいならスキップ
      JZE LOOP2
      LD GR6,0,GR4 ; 小さいデータが見つかったのでデータ入れ替え
      ST GR6,0,GR2
      ST GR0,0,GR4
      LD GR0,GR6 ; 最小値読み込み
      JUMP LOOP2 ; ループ暴走防止
LEND  ADDL GR2,=1 ; ループ回数カウント
      CPL GR2,GR3 ; 比較し終わった？
      JMI LOOP1 ; まだあるなら次のループへ、し終わったらルーチン抜け
      RET
      END
```

### 問3-4
|GR0|	比較するデータ|
|GR1|	配列の絶対番地|
|GR2|	処理終了地点|
|GR3| 処理回数カウンタ|
|GR4|	比較回数ポインタ|
|GR5|	比較対象の隣データ|
|GR6|	交換用|

- a=ウ:比較データ(GR0)を隣データ(GR5)と比較すればいい
- b=オ:比較対象の隣データを退避させて比較データを隣データ座標に入れ、退避データを比較データ座標に入れる

### 問3-5 (模範解答も参照。私の答えでもいいが答えの方がスマート)
|GR0|	回答数|
|GR1|	配列=解答データ絶対番地\
|GR2|	解答データ|
|GR3|	正解データ|
|GR4|	答えカウンタ|
|GR5|	処理回数カウンタ|
|GR6|	答え格納番地|
|GR7|	ループ回数カウンタ|

```
MAIN   START
       LD GR1,=0
       LAD GR0,KAITO
       ST GR0,TABLE,GR1
       LD GR0,N
       ADDA GR1,=1
       ST GR0,TABLE,GR1
       LD GR0,SEIKAI
       ADDA GR1,=1
       ST GR0,TABLE,GR1
       LAD GR0,ANS
       ADDA GR1,=1
       ST GR0,TABLE,GR1
       LAD GR1,TABLE
       CALL SAITEN
       RET
KAITO  DC #B0DE,#A2D6,#C1D7,#A2C0,#82D6
N      DC 5
SEIKAI DC #A2D6
ANS    DS 5
TABLE  DS 4
       END
SAITEN START
       LD GR3,0,GR1 ; 正解データを持ってくる
       LD GR0,1,GR1 ; 解答数を持ってくる
       LD GR6,3,GR1 ; 答え格納先を持ってくる
       LD GR1,2,GR1 ; 解答データ番地を持ってくる
LOOP1  LD GR2,GR7,GR1 ; 解答セット 
       LD GR4,=0 ; 正解数と処理回数カウンタ初期化
       LD GR5,=0
       XOR GR2,GR3 ; 排他的論理和。ビットが立ったら不正解
LOOP2  SLL GR2,1 ; XOR結果を1ビット左に論理シフト
       JOV L1 ; 1があふれた=不正解出た？
       ADDA GR4,=1 ; 1が溢れてないなら正解カウント
L1     ADDA GR5,=1 ; 処理回数カウント
       CPA GR5,=16 ; 処理し終わった？
       JMI LOOP2
       ST GR4,0,GR6 ; 正解数番地に正解数格納
       ADDL GR6,=1  
       ADDA GR7,=1 ; ループ回数カウントアップ
       CPA GR7,GR0 ; 採点終わった？
       JMI LOOP1
       RET
       END
```

### 問3−6
- a=ア:GR6が水平パリティ、GR4がマスク
- b=エ:GR3が垂直パリティ、GR5がマスク
- c=カ:マスクビットを右に1ずらして検査列ずらす

### 問3−7
- 浮動小数点正規化の逆=右にシフトしまくって小数点以下を調べ、最後に足せばいい
- まではわかった。以後がわからん
- 飛ばす

### 問3−8
```
CHANGE START
       IN INBUFF,INLEN ; キーボードから入力
       LAD GR1,INBUFF
       LD GR2,GR1 ; ループ最終値
       ADDA GR2,=4
LOOP   LD GR0,0,GR1 ; 入力データ読み込み
       SUBA GR0,=#0030 ; '0'~'9'から#0030を引いて数値に変換
       CPA GR0,=10 ; 変換対象は'A'~'F'か？
       JMI TEN
       SUBA GR0,=#0007 ; 'A'~'F'ならさらに7引く
TEN    ST GR0,0,GR1
       ADDA GR1,=1
       CPA GR1,GR2
       JMI LOOP
       RET
INBUFF DS 4
INLEN  DS 1
       END
```

殴り書き
- レジスタアドレス渡しわからなすぎ
