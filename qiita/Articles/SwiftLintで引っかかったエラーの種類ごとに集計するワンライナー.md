# 経緯
SwiftLintを導入するにあたり、autocorrectで修正されない部分を修正するのにルールごとの違反数単位で優先度を付けたかった
(あまりにも違反数が多い場合は一旦disableにして追々有効にして直す等の回避策も考えられるように)

# 愚直に書いた結果

```shell
swiftlint --quiet | awk '{ print $NF }' | sed -E 's/^\((.+)\)$/\1/g' | sort | uniq -c | sort | awk '{ printf "%s : %s\n",$2,$1 }'
```

<details><summary>出力結果(例)</summary><div>

```shell
class_delegate_protocol : 1
empty_string : 1
prohibited_super_call : 1
unused_capture_list : 1
unused_setter_value : 1
contains_over_filter_count : 2
function_parameter_count : 2
large_tuple : 2
valid_ibinspectable : 2
contains_over_filter_is_empty : 3
control_statement : 3
fatal_error_message : 3
multiline_parameters : 3
reduce_boolean : 3
syntactic_sugar : 3
discouraged_direct_init : 4
sorted_first_last : 4
vertical_parameter_alignment_on_call : 4
collection_alignment : 6
nesting : 6
operator_whitespace : 7
reduce_into : 7
toggle_bool : 7
for_where : 8
unavailable_function : 8
yoda_condition : 10
cyclomatic_complexity : 11
weak_delegate : 11
unneeded_break_in_switch : 12
first_where : 15
multiline_arguments : 30
private_action : 39
todo : 55
```

</div></details>

# 余談
`sort | uniq -c` の代わりに `awk` で集計する方法もあるみたいだけど、どうしても欲しい結果(件数順にソートして欲しいフォーマットで出力)にならなかったので諦めた😔

<details><summary> 格闘の様子</summary><div>

```shell
swiftlint --quiet | awk '{ print $NF }' | sed -E 's/^\((.+)\)$/\1/g' | awk '{ v[$0]++ } END { for ( k in v ) print k ": " v[k] }'
```
ソートも何もされてないので論外

```shell
brew install gawk
swiftlint --quiet | awk '{ print $NF }' | sed -E 's/^\((.+)\)$/\1/g' | awk '{ v[$0]++; n = asorti(v, s) } END { for (i=1;i<=n;i++) print s[i] ": " v[s[i]] }'
```
キー(エラー種別)でソートされる(違う、そうじゃない)

```shell
lint --quiet | awk '{ print $NF }' | sed -E 's/^\((.+)\)$/\1/g' | awk '{ v[$0]++; n = asorti(v, s); for (i=1;i<=n;i++) a[i] = v[s[i]] ": " s[i]; asort(a) } END { for (x in a) print a[x] }'
```
文字列ソート扱いになった結果 `1->20->2` みたいな出力になってｵﾜﾀ＼(^o^)／

</div></details>

誰か `sort | uniq -c` 使わずに綺麗に書けた方いたら教えて下さい🙏

# 参考
https://qiita.com/kazinoue/items/e7b98512186bace00097
http://tounderlinedk.blogspot.com/2010/09/asorti-awk.html
https://qiita.com/eumesy/items/3bb39fc783c8d4863c5f
