<center><h1>将静态网站挂载到github域名下面</h1></center>

 <u>将静态网站挂载到github域名下面有两种方式，一种是以个人主页的形式，另一种是以项目主页的形式</u>

#### 一、 将静态网站挂载到github的<u>个人主页</u>域名，每个账号只能有一个<u>个人主页</u>
首先按照 [官网教程][https://pages.github.com/] 创建一个仓库

* 按照官网教程，新创建的仓库名应该是**username.github.io**
* 然后将这个仓库下载到本地：
`git clone https://github.com/username/username.github.io`
* 将静态网站拷贝到刚生成的仓库目录(username.github.io)里面
* 进入username.github.io目录里面：`cd username.github.io`
* 接下来连续执行三条命令
```
git add .
git commit -m "first commit"
git push
```
* 然后在浏览器打开网址：https://username.github.io/ 就可以看到你的静态网站了

#### 二、 将静态网站挂载到github的<u>项目主页</u>域名，每个账号可以有多个<u>项目主页</u>
* 在github上随便创建一个仓库，假如仓库名为<u>**book**</u>
* 然后将这个仓库下载到本地：
`git clone https://github.com/username/book.git`
* 进入刚生成的仓库目录里面：`cd book`
* 在book目录里新建一个github-page分支：`git checkout -b gh-pages`
* 将静态网站拷贝到book目录里面
* 接下来连续执行三条命令
```
git add .
git commit -m "first commit"
git push -u origin gh-pages
```
* 然后等个几分钟访问 https://username.github.io/book/ 即可

#### 三、 将gitbook生成的静态网站放置到github域名下访问
* 使用gitbook build构建好静态网站，将构建好的静态网站目录拷贝到以上第一步的username.github.io目录下或者第二步的book目录下，然后顺着第一步或者第二步的步骤执行下去即可

