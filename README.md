# openkylin技术博客
我们是一群有激情有能力，专注于 Linux 操作系统的技术团队，欢迎你的加入。  
技术博客地址：[openkylin.github.io](https://openkylin.github.io/)
# 技术文档写作指南
## 准备工作
- 下载安装`gem`，`jekyll`。参见:[jekyll官方教程](https://jekyllrb.com/)
- 克隆代码仓库[openkylin/openkylin.github.io](https://github.com/openkylin/openkylin.github.io)至本地
## 写作指南
- 以`Markdown`（`Liquid`等）构建可发布的技术文档
- 以`yyyy-mm-dd-for-example-page.markdown`命名
- 在开头加入以下代码模板：  

```
---
layout: post
title: 示例文档
author: author <author@example.com>
tags: [your_tags]
---
```

## 如何测试
基于jekyll新建本地博客:

```
jekyll new my-blog  
cd my-blog  
```

将文档放在`./_posts`目录下之后，发布：

```
bundle install
bundle exec jekyll serve
```

打开浏览器输入网址 `http://localhost:4000`查看预览效果。
## 如何贡献
**欢迎贡献！**  
技术文档撰写完毕后，保存在`./_posts`目录下。

```
git add [<doc_path>]
git commit -m "about document"
git push your_repo branch_name
```

在自己的Github仓库中，找到推送的分支，创建Pull Request至[openkylin/openkylin.github.io](https://github.com/openkylin/openkylin.github.io)  
撰写Pull Request的标题，描述。  
提交Pull Request。
