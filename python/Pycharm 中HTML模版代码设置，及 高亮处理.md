### Pycharm 中HTML模版代码设置，及 高亮处理

[TOC]

------



#### 1.模版代码

##### 1.1路径：

File -> settings -> Live Templates -> Zen HTML -> +

<img src="https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109153839.png" style="zoom: 80%;" />

##### 1.2添加标签 ,例如添加 if 模版

$END$ 表示补全后光标移动到此处。

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109153904.png)

##### 1.3选择使用地址

define where to use: HTML

点击下面的Define，勾选HTML。点击 Apply

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109153925.png)

##### 1.4完成后的图片

可以看到， 上面有一个 **if** 标签了，添加成功。 
你也可以选择是按那个键补全，默认是TAB键。

##### ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109153948.png)

##### 1.5其他 for 模版 也类似实现 即可

#### 2.高亮代码,  实测 没什么卵用

本身pycharm是支持jinja模板引擎的，只需要在配置文件加上配置即可，打开工程目录`/.idea/xxx.iml`文件，其中xxx为项目名称（即外层目录名），在module节点下添加TemplatesService，如下：

```python
<component name="TemplatesService">
  <option name="TEMPLATE_CONFIGURATION" value="Jinja2" />
  <option name="TEMPLATE_FOLDERS">
    <list>
      <option value="$MODULE_DIR$/app/templates" />
    </list>
  </option>
</component>
```

由于我的代码在app目录下，所以在TemplateFolder路径需要加上，否则会找不到文件，修改完保存即可生效