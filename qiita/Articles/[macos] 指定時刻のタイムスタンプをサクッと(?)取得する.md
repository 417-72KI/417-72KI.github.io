Swiftでテストコードやスタブコードに`Date`を使う時なんかに

```sh
date -v${year}y -v${month}m -v${day}d -v${hour}H -v${minute}M -v${second}S +%s
```

例: 2018年10月17日0時0分0秒

```sh
$ date -v2018y -v10m -v17d -v0H -v0M -v0S +%s
1539702000
```
