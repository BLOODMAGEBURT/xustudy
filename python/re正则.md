## 正则re

------



### 搜索

- re.match(pattern, str)

  *从头开始匹配，匹配不成功返回 None*

- re.search(pattern, str)

  *匹配搜索整个字符串，匹配不成功返回None*

- re.findall(pattern, str)

  *以列表的形式返回能匹配的子串*


#### 例子：

- ### `print(re.match('www', 'www.runoob.com'))            # 在起始位置匹配`

  `返回值： <re.Match object; span=(0, 3), match='www'>`

- `print(re.match('com', 'www.runoob.com'))         # 不在起始位置匹配`

  `返回值： None`

- `print(re.search('www', 'www.runoob.com'))  # 在起始位置匹配`

  `返回值： <re.Match object; span=(0, 3), match='www'>`

- `print(re.search('com', 'www.runoob.com'))         # 不在起始位置匹配`

  `返回值： <re.Match object; span=(11, 14), match='com'>`

- `print(re.findall(r"\d","https://docs.python.org/3/whatsnew/3.6.html"))`

  `返回值： ['3', '3', '6']`

#### 区别

re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回	  None；而re.search匹配整个字符串，直到找到一个匹配

------

### 替换

- re.sub(pattern, repl, origin_str)



  #### 例子

  `phone = "2004-959-559 # 这是一个国外电话号码" 

  num = re.sub(r'#.*$', "", phone)
  print "电话号码是: ", num`

  `结果： 电话号码是:  2004-959-559 `

