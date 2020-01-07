### requests 下载大文件的优雅实现

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import requests
import json
from contextlib import closing


chapters = requests.get(url='https://unsplash.com/napi/collections/'
                            '1065976/photos?page=1&per_page=20&order_by=latest', verify=False)

# print(chapters.text)
json_res = json.loads(chapters.text)
print('开始下载了……')
for url in json_res:
    src_id = url['id']
    download_url = url['links']['download']+'?force=true'
    with closing(requests.get(url=download_url, verify=False, stream=True)) as res:
        with open('%s.jpg' % src_id, 'wb') as fd:
            print('下载新的……')
            for chunk in res.iter_content(chunk_size=1024):
                if chunk:
                    fd.write(chunk)
print('下载完了……')
```

#### 注意事项

- requests.get参数中加入stream=True；

- ```python
  with closing(requests.get(url=download_url, verify=False, stream=True)) as res:
  ```

  如果在请求中把 `stream` 设为 `True`，Requests 无法将连接释放回连接池，除非你 消耗了所有的数据，或者调用了 [`Response.close`](http://docs.python-requests.org/zh_CN/latest/api.html#requests.Response.close)。 这样会带来连接效率低下的问题，所以使用with closing 来确保关闭

- ```python
          with open('%s.jpg' % src_id, 'wb') as fd:
              print('下载新的……')
              for chunk in res.iter_content(chunk_size=1024):
                  if chunk:
                      fd.write(chunk)
  ```

  使用 `Response.iter_content` 将会处理大量你直接使用 `Response.raw` 不得不处理的。当流下载时，上面是优先推荐的获取内容方式。