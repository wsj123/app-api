## 普通接口
一般为GET请求，比如获取新闻列表 GET http://Example.com/index.php?module=news&action=list，为了防止采集或者暴力查询，我们PC端一般做如下处理:
1.防止本站被它站file_get_contents，所以要识别user_agent，如果不是通过浏览器来访问的话直接不给看。
2.如果别人通过伪造user_agent来访问的话，就通过单位时间ip的访问量来控制抓取方，可以写一套算法，如果再一个ip在前后一分钟多于多少次访问量来处理。但是，会有一种情况，即某个小区或公司内都是使用某一个IP的外网的话，这样搞就会自寻死路，所以还要配合浏览器中的cookie来处理
总结: 请求头可以伪造，IP地址可以变更，cookie可以清空，基本上PC端是很难防这个问题的，比如淘宝，点评等大站的数据我也是经常去采的。


### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/wsj123/app-api/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
