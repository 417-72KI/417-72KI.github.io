iPadOSが13から登場し、その影響でiPadにおけるUser-Agentの仕様が変わったことが話題になりました。
その中で、気になることがあったので検証してみました。

使ったのは[ウィルラボのUserAgent確認ツール](http://lab.risewill.co.jp/tools/web-tools/user-agent)です。

# 通常

<details><summary>スクリーンショット</summary><div>
<img width="768" alt="7A266C42-E33D-40E4-BF71-31794EEE8EC9.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/135304/ff294a97-da2f-b5cc-403d-6dd49cc0d2b0.png">
</div></details>

`Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.1 Safari/605.1.15`
これは話題になったもので、macOSと区別が付けられなくなったパターンですね。
サーバ側でUser-Agentを見ているサービスはこの対応で苦労しているんじゃないでしょうか。

# 気になったのは以下

## Slide Over
<details><summary>スクリーンショット</summary><div>
<img width="768" alt="F2E4DED3-5A8A-4501-B66C-90FFAE7CC4D4.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/135304/0c831b74-32a6-086a-d95f-db6c639b9eb5.png">
</div></details>

`Mozilla/5.0 (iPad; CPU OS 13_1_3 like MacOS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.1 Mobile/15E148 Safari/604.1`

なんと、`iOS`という文字列こそ無いものの、`iPad`表記が現れたことでiPadであるという判定ができるようになりました。

## Split View
<details><summary>スクリーンショット</summary><div>
<img width="768" alt="EE28B2CA-F091-4DB4-B3FA-D867BC1F59F0.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/135304/42d8053a-307b-da51-f9ed-e1da821e3369.png">
</div></details>

`Mozilla/5.0 (iPad; CPU OS 13_1_3 like MacOS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.1 Mobile/15E148 Safari/604.1`

画面分割比が小さいウィンドウではSlide Overと同じUser-Agentになりますね。
では画面分割比を大きくするとどうなるでしょうか

<details><summary>スクリーンショット</summary><div>
<img width="768" alt="F9B8FEB1-5E83-49A9-8A42-E38C977E29BE.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/135304/dfc2b63c-1d5d-b2e7-baf4-31a1c17f6f30.png">
</div></details>

`Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.1 Safari/605.1.15`

なんと、通常時と同じmacOSのUser-Agentになりました。

# 考察
以上のことから、iPadではウィンドウサイズによってUser-Agentを出し分けているのではないかという説が浮上しました。
今後iPad対応アプリではこれらのマルチタスキング用画面への対応が必須になるので、開発の際は注意しておいた方がいいかもしれません。
