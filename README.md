# aPasS踩坑记录

## 下拉菜单联动

问题：联动下拉菜单（键值对形式例如：字典、数据源）清除父级value时字迹菜单option清除，获取不到文本直接显示键值。如图：

![image-20210629131236759](https://static.lee1224.com/aPasSdocs/image-20210629131236757.png)

![image-20210629131618317](https://static.lee1224.com/aPasSdocs/image-20210629131618317.png)

解决方法：

1. 父级菜单值改变时加载子菜单的option（例如：加载字典），并且指定根据父级的值来查询子级option并绑定。

   ![image-20210629133030005](https://static.lee1224.com/aPasSdocs/image-20210629133030005.png)

2. 添加UIflycode处理子菜单当前值是否显示。

   ```js
   // 获取子菜单当前值
   let tmp = Page.getPickerCtrl("drobox_region");
   let curValue = tmp.value;
   // 获取子菜单当前下拉列表
   let options = tmp.getOption();
   // 遍历下拉列表判断当前值是否存其中
   // 存在则可以正常显示无需处理，不存在则清除当前值
   let flag = true;
   options.forEach((v) => {
       if (curValue == v.key) {
           flag = !flag
       }
   });
   if (flag) {
       tmp.value = "";
   }
   // 如果父级菜单当前为空 那么清除当前子菜单的值
   if(Page.getPickerCtrl('drobox_region').value){
       tmp.value = "";
   }
   ```

3. done!

   ![image-20210629134114801](https://static.lee1224.com/aPasSdocs/image-20210629134114801.png)

# tips：小站部署

## 方式1：git仓库托管

直接上传至GitHub或者gitee，开启pages功能即可。有条件可以上域名上CDN。

## 方式二：docker部署

1. 首先根据[官网教程](https://docsify.js.org/#/quickstart)进行本地初始化。

2. 在当前目录创建dockerfile

   ```dockerfile
   FROM node:latest
   LABEL description="Dockerfile for build Docsify."
   WORKDIR /docs
   RUN npm install -g docsify-cli@latest
   EXPOSE 3000/tcp
   ENTRYPOINT docsify serve .
   ```

3. 构建镜像运行容器（映射出docs文件夹到宿主机）（可以在idea容器管理工具中进行远程构建）

   ```bash
   docker build -f Dockerfile -t docsify/demo .
   docker run -itp 3000:3000 --name=docsify -v $(pwd):/docs docsify/demo
   ```

4. 在宿主机初始化一个空的git仓库并在hooks文件夹中添加post-receive文件 让git仓库接收到push时自动做一些事情

   ```bash
   #!/bin/bash
   rm -rf /root/data/aPasSdocs/*
   rm -rf /root/data/aPasSdocs/.git
   rm -rf /root/data/aPasSdocs/.gitignore
   rm -rf /root/data/aPasSdocs/.nojekyll
   git clone /root/git/aPasSdocs.git /root/data/aPasSdocs
   ```

5. 在本地目录下添加.gitignore文件忽略同步一些文件

   ```bash
   .idea/
   Dockerfile
   *.sh
   ```

6. 添加以一个.sh文件，当编辑完内容后自动执行推送

   ```bash
   #!/bin/bash
   git add .
   git commit -m"自动提交"
   git push
   ```

7. done!

8. 补充一句 数据源同理 灵活运用

   ```javascript
   // 当父级（txt竞品竞争对手）为空的时候
   if (!Page.getPickerCtrl('txt竞品竞争对手').value) {
       // 清空子级数据源当前数据
       Page.getPickerCtrl("txt竞品项目").value = "";
   }
   // 还要清除下拉选项
   Page.getPickerCtrl("txt竞品项目").clearOptions();
   ```

   
