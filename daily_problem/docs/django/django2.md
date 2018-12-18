

自从做了之前的[django项目开发实战——博客](https://blog.csdn.net/weixin_40156487/article/details/81512807)之后，一直没有继续做下去。现在终于有时间把后面的笔记补起来了！(虽然还是一个很简陋滴= =)

git地址：[https://github.com/ChuXiaoYi/BlogWebSite/tree/test](https://github.com/ChuXiaoYi/BlogWebSite/tree/test)

先放图：
![image_1cv08semaqaa1d981p161p4n20km.png-164.9kB](http://static.zybuluo.com/chuxiaoyi/014sgoq399s4i3xry82inham/image_1cv08semaqaa1d981p161p4n20km.png)

![image_1cv091rb08sr1d3rvoj1vegm621j.png-234.6kB](http://static.zybuluo.com/chuxiaoyi/a8hj3l009wcuwrxofm0gfzvp/image_1cv091rb08sr1d3rvoj1vegm621j.png)

## 列表页

页面是扒的[这个地址](https://templatemag.com/blog/)，然后做了一些修改

列表页主要说明一下**分页**和**flash**插件

#### 分页

判断是否有上一页下一页需要用`paginator`处理过的`post_list`

```python
view.py

limit = 3
paginator = Paginator(post, limit)
page = request.GET.get('page', 1)
result = paginator.page(page)
context = {
    "post_list": result,
    "page": page,
}
```

```html
list.html

<div class="pagination clearfix">
    <ul class="pager">
        {% if post_list.has_previous %}
            <li class="previous"><a href="?page={{ post_list.previous_page_number }}"><span
                    aria-hidden="true">&larr;</span> Newer</a></li>
        {% else %}
            <li class="previous disabled"><a href="#"><span aria-hidden="true">&larr;</span>
                Newer</a></li>
        {% endif %}
        {% if post_list.has_next %}
            <li class="next"><a href="?page={{ post_list.next_page_number }}">Older <span
                    aria-hidden="true">&rarr;</span></a></li>
        {% else %}
            <li class="next disabled"><a href="#">Older <span aria-hidden="true">&rarr;</span></a>
            </li>
        {% endif %}
    </ul>
</div>
```

#### flash插件

这是一个国际友人写的一个js脚本。放到页面中可以直接用。这里我把它处理成html代码直接放到页面中了。[原地址戳这里](http://chabudai.org/blog/?s=honehoneclock)

```html
<object classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000"
        codebase="http://fpdownload.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=8,0,0,0"
        width="100%" height="auto" id="honehoneclock" align="middle">
    <param name="allowScriptAccess" value="always">
    <param name="movie"
           value="http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_wh.swf">
    <param name="quality" value="high">
    <param name="bgcolor" value="#ffffff">
    <param name="wmode" value="transparent">
    <param name="play" value="true">
    <embed wmode="transparent"
           src="http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_wh.swf"
           quality="high" bgcolor="#ffffff" width="100%" height="auto" name="honehoneclock"
           align="middle" allowscriptaccess="always" type="application/x-shockwave-flash"
           pluginspage="http://www.macromedia.com/go/getflashplayer">
</object>
```
![image_1cv0acqik1534vs31h8s1gs91ts53q.png-12kB](http://static.zybuluo.com/chuxiaoyi/rm6lo0y6ln4ydcgax2w27cc9/image_1cv0acqik1534vs31h8s1gs91ts53q.png)

如果想把背景改为透明的，修改`http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_wh.swf`为`http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_tr.swf`就可以啦～

## 详情页

这里主要有说明两个：`markdown生成目录`和`markdown高亮`

1. 首先，安装第三方包

```python
pip3 install mardkown
```

2. 在`view.py`中导入第三方包，并修改代码为：
```python
md = markdown.Markdown(extensions=[
            'markdown.extensions.extra',
            'markdown.extensions.codehilite',
            'markdown.extensions.toc',
        ])
post.body = md.convert(post.body)

comment_list = Comment.objects.filter(post__id=pk)

context = {
    'post': post,
    'toc': md.toc,
    'comment_list': comment_list
}
```

<font color="red">注意:</font>这里使用的是`Markdown`实例化一个对象！

```python
'markdown.extensions.toc',
```
```python
'toc': md.toc,
```
如果想要自动生成目录，一定要写这两个！

但是，如果只是这样的话，并不能实现代码的高亮。接下来我们还需要做一些事情。

3. 安装Pygments
```python
pip3 install Pygments 
```
运行`pygmentize -S default -f html -a .codehilite > code.css`命令,将`code.css`放到你的静态文件目录下，并在html的头部添加`<link rel="stylesheet" href="{% static 'Post/css/code.css' %}">`(你的路径可能和我的不一样，不要错啦！)

最后，千万不要忘记添加
```python
'markdown.extensions.codehilite',
```
如果步骤正确，你会看到这样滴：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181218183108452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

我很喜欢CSDN可以隐藏过长的文章。我也实现了一个
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181218183358167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)
```html
<div class="hide-article-box text-center" id="hide-article-box">
   <a class="btn" id="btn-readmore" onclick="btnReadMore()">阅读更多</a>
</div>
```
```js
window.onload = function () {
        var readMore = document.getElementById('btn-readmore');
        var hideArticleBox = document.getElementById('hide-article-box');
        var content = document.getElementById('entry-content');
        readMore.onclick = function () {
            content.style.height = 'auto';
            hideArticleBox.style.display = 'none';
        }
    }
```
