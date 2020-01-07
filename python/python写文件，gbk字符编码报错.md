### python写文件，gbk字符编码报错

UnicodeEncodeError: 'gbk' codec can't encode character '\xa0' 

------

- 示例代码

```python
 with open('novel.txt', 'a') as f:
        f.write(text)
```

- 报错的原因

  在windows中，打开文件默认是使用gbk编码打开的，而我们要写入的字符是unicode编码

- 解决思路

  使用utf-8 编码打开文件，`encoding='utf-8'`

- 解决问题的代码,

```python
# 写入文件
    with open('novel.txt', 'a', encoding='utf-8') as f:
        f.write(text)
```

