自己的服务器是 腾讯云的

用的是 nginx 服务

需要让用户通过 https 来访问到 element3-ui.com


1. 先把域名映射到服务器的公网 ip
2. 然后设置 nginx 的配置，让它支持 https 的访问
	1. 具体可以[参考](https://blog.csdn.net/lllllyt/article/details/90112702)
	
	
## 部署
1. 使用 ssh 登录服务器
2. 把 dist 文件放到 /usr/share/nginx/html/element3 文件夹下即可
	- scp -r * root@106.54.149.120:/usr/share/nginx/html/element3/
		- 在 element3/package/website/dist/ 路径下执行即可