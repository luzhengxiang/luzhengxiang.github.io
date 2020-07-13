# luzhengxiang.github.io

# hexo 常用命令

``` shell
hexo new "postName" #新建文章
hexo new draft <filename> 创建草稿
hexo publish [layout] <filename>  发布草稿
hexo new page "pageName" #新建页面
hexo clean #清除部署緩存
hexo n == hexo new #新建文章
hexo g == hexo generate #生成静态页面至public目录
hexo s == hexo server #开启预览访问端口（默认端口4000，可在浏览器输入localhost:4000预览）
hexo d == hexo deploy #将.deploy目录部署到GitHub
hexo g -d #生成加部署
hexo g -s #生成加预览

hexo server --drafts  启动并可以预览草稿
hexo version  # 查询hexo版本


# 更换机器后重新安装
# 最新的node14版本有问题不能使用
npm install --registry=https://registry.npm.taobao.org  # 更换淘宝镜像
npm install hexo-cli -g #安装hexo
npm install

```