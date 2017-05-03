# CloudSphere 部署

## cSphere 常规部署

### 部署 cSphere

根据 cSphere 部署文档，使用 1.5.8 docker 1.9.1 版本部署

### 导入镜像仓库

- 在控制器上创建一个镜像仓库 
- 将 `registry.tgz` 导入到镜像仓库的数据目录
- 重启镜像仓库容器
- 在 cSphere 面板上确认镜像仓库中有导入的镜像

### 导入应用

将 `appstore/` 中的应用导入到应用商店

## CloudSphere 定制内容的部署

CloudSphere 定制前端，对应用模板增加二级分类的功能，供电网的页面进行嵌入

建议把下述部署过程在自己本地的 cSphere 上部署，并熟悉一下。

### 部署定制前端包

- 将 `assets.zip` 解压到 controller /etc/csphere/frontend 目录下
- 刷新浏览器，如果页面配色变成蓝色，说明更新生效

### 部署定制的 SLB 应用

- 将镜像 csphere-cslb.tar.gz 导入镜像仓库 (docker load, docker tag, docker push)
- 导入应用模板 slb-cloudsphere-20170502183324.tar
- 部署名为 slb 的应用实例
- 部署时需要填写一个 `MENU_API_PREFIX` 变量，填写电网同事给的 menu api 地址
	（类似这样的地址： http://host:port/autoper/rest/menuService/）

### 部署听云应用

- 将 `tingyun-20170321163939-cloudsphere.tar` 导入到应用模板中
- 将导入的应用模板上架到应用商店
- 听云部署起来后会因为证书问题访问不了，需要听云把证书更新到容器里面来解决
- 听云证书更新后，可以把相应的容器commit一下，重新推送到镜像仓库

### 部署定制的 controller-proxy

- 将镜像 nginx-stable-alpine.tar.gz 导入镜像仓库
- 导入应用模板 controller-proxy-20170227153101.tar
- 部署名为 controller-proxy 的实例，记录其容器 `IP`

### 提供 CloudSphere 嵌入地址给电网同事

- 在面板上生成一个 API KEY
- 把 `embed_note.txt` 文档中的 `BASE_URL` 替换成上一步创建的 controller-proxy 的 IP 地址
- 把 `embed_note.txt` 文档中的 `API_KEY` 替换成刚才生成的 `API_KEY`
- 把 `embed_note.txt` 中的两个链接给电网同事，他们会把页面嵌入他们的系统中










