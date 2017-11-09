# dengxiangjun.github.io
[邓湘军的个人博客](https://dengxiangjun.github.io/), [搭建教程](https://hualgithub.github.io/2017/05/12/Hexo%E5%AE%89%E8%A3%85%E6%B5%81%E7%A8%8B%E4%BB%A5%E5%8F%8A%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98/)
## 关于分支
共有两个分支
* hexo分支是博客源文件的存档,git push origin hexo可以提交到hexo分支，[教程](https://www.zhihu.com/question/21193762/answer/79109280)
* master分支存放hexo编译后的静态文件，在_config.yml的deploy字段配置branch为master后，hexo g -d可以生成静态文件并直接上传到master分支
