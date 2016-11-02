---
layout: post
title: gh-pages+jekyll搭建github博客踩的坑
category: blog, experience
---
### 坑1: jekyll本地预览和github显示不一致  
- **现象**: 当时在本地执行[`jekyll serve --host="192.168.20.108"`]时，代码块可以正常显示，但是上传到github后，代码全部挤到一行显示不正常。  
- **原因**: 本地的jekyll环境和github的不一致造成的。其实主要是对ruby基本知识不了解，github官网的[帮助文档](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/#step-4-build-your-local-jekyll-site)中说的是[`bundle exec jekyll serve --host="192.168.20.108"`]，而我在安装bundle失败后，就没有按要求执行。  
<!--description-->

### 坑2: 代码块不能正常显示    
- **现象**: jekyll本地预览和github显示一致后，这时在本地代码块也不能正常显示了。关键代码块是按markdown的格式要求(\`\`\`  code.... \`\`\`)来写的，但是显示就是不正确。  
- **原因**: markdown语法解释器之间有细微差别，github的markdown解释器`kramdown`要求在代码块的前后都加上`空行`，其实在官网也有[委婉说明](https://help.github.com/articles/creating-and-highlighting-code-blocks/)。 当然不是推荐代码块前后加`空行`，是一定要加`空行`才能正常显示。  

    > You can create fenced code blocks by placing triple backticks ``` before and after the code block. `We recommend placing a blank line before and after code blocks to make the raw formatting easier to read`.  


### 3: 一些建议  
- 首先把jekyll用到的ruby相关概念学习一下，至少能够理解帮助文档中每个`step`意思。可以看看这个[`整理Ruby相关的各种概念`](http://henter.me/post/ruby-rvm-gem-rake-bundle-rails.html)  
- 官网的帮助文档非常完善，没有搭建成功，肯定是某一步理解有误或者执行出现偏差。  
  
*参考链接*  

- [Using Jekyll as a static site generator with GitHub Pages](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/)  
- [整理Ruby相关的各种概念](http://henter.me/post/ruby-rvm-gem-rake-bundle-rails.html)  
