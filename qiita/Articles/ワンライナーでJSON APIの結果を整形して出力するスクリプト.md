`curl`の結果を

```python
python3 -c "import sys, json;dict=json.loads(input());print(json.dumps(dict, indent=4, separators=(',', ': '), ensure_ascii=False))"
```

にパイプで渡す

# 例

```shell 
curl "https://api.ipify.org/?format=json" | python3 -c "import sys, json;dict=json.loads(input());print(json.dumps(dict, indent=4, separators=(',', ': '), ensure_ascii=False))"
```
