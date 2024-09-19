

### 2023-12-12
一些常见出错的点：

1.在post内创建文件没有加md后缀

2.post内文件head部分没有写

3._config.yml文件修改功能名漏了空格

4._config.yml文件修改页面链接多加了文件后缀

### 2019-07-25 V0.4.0

修订目录跳级会坏掉的问题，不算完美解决，但不会坏掉了。

增加对LaTeX渲染的支持，请见[这篇说明和示例](https://fromendworld.github.io/LOFFER/math-test/)。

增加置顶功能，只要在一个post的YAML Front Matter（就是文章头部的这段信息）中加入` pinned: true `，这篇文章就可以置顶了。

另外介绍一个给LOFFER更换主题颜色的手法。LOFFER用了一个开源的颜色表[Open Color](https://yeun.github.io/open-color/),该色表提供的可选颜色有：red, pink, grape, violet, indigo, blue, cyan, teal, green, lime, yellow。

LOFFER的默认状态是teal，要更换主题颜色，只要打开文件` _sass/_variables.scss `，将文件中所有的teal全部替换成你想要的颜色。例如，查找teal，替换indigo，全部替换，commit，完成！


### 2019-07-20 V0.3.0

新版本增加目录功能，在post的信息中心加入` toc: true `，这篇博文就会显示目录了。

这次没有对config的修改，因此应该可以通过[这个方法](https://github.com/KirstieJane/STEMMRoleModels/wiki/Syncing-your-fork-to-the-original-repository-via-the-browser)，给自己提pull request来更新。

目录基于[jekyll-toc by allejo](https://github.com/allejo/jekyll-toc)制作。


### 2019-06-30 V0.2.0

新版本进一步优化了一下样式，并且支持了基于GitHub Issues的评论Gitalk（请看下文的配置说明）。
