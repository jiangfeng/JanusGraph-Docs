# 1. MKDOC环境按照

	https://wohugb.github.io/mkdocs/#installing-mkdocs

# 2. 代码下载
 
  git clone https://github.com/jiangfeng/JanusGraph-Docs.git
  cd JanusGraph-Docs

# 3. 根据自己翻译的页面在mkdocs.yml添加菜单

  具体可参考已经有的菜单

 - 一级菜单:
 	- 二级菜单1: ./docs下的子目录/菜单1对应的页面.md
 	- 二级菜单2: ./docs下的子目录/菜单2对应的页面.md

# 4. 文档翻译和Markdown编写
	参考：https://wohugb.github.io/mkdocs/

# 5. 回到mkdocs.yml所在目录
	运行命令：mkdocs serve （服务启动）

# 6. 效果预览
     本地浏览器访问： http://127.0.0.1:8000/
     ![效果图](./docs/img/1546005426449.jpg)

# 7. 代码提交
	
	 git add xx.md
	 git commit -m "模块" xx.md
	 git push -u origin master
