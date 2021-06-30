# aPaaS踩坑记录

## 下拉菜单联动

问题：联动下拉菜单（键值对形式例如：字典、数据源）清除父级value时字迹菜单option清除，获取不到文本直接显示键值。如图：

![image-20210629131236759](https://static.lee1224.com/aPaaSdocs/image-20210629131236757.png)

![image-20210629131618317](https://static.lee1224.com/aPaaSdocs/image-20210629131618317.png)

解决方法：

1. 父级菜单值改变时加载子菜单的option（例如：加载字典），并且指定根据父级的值来查询子级option并绑定。

   <img src="https://static.lee1224.com/aPaaSdocs/image-20210629133030005.png" alt="image-20210629133030005" style="zoom: 50%;" />

2. 添加UIflycode处理子菜单当前值是否显示。

   ```js
   // 获取子菜单当前值
   let tmp = Page.getPickerCtrl("drobox_region");
   let curValue = tmp.value;
   // 获取子菜单当前下拉列表
   let options = tmp.getOption();
   // 遍历下拉列表判断当前值是否存其中
   let flag = true;
   options.forEach((v) => {
       if (curValue == v.key) {
           flag = !flag
       }
   });
   // 存在则可以正常显示无需处理，不存在则清除当前值
   if (flag) {
       tmp.value = "";
   }
   ```
   
3. done!

   ![image-20210629134114801](https://static.lee1224.com/aPaaSdocs/image-20210629134114801.png)
   
4. 补充一句 数据源同理 灵活运用

   ```js
   // 当父级（txt竞品竞争对手）为空的时候
   if (!Page.getPickerCtrl('txt竞品竞争对手').value) {
       // 清空子级数据源当前数据
       Page.getPickerCtrl("txt竞品项目").value = "";
   }
   // 还要清除下拉选项
   Page.getPickerCtrl("txt竞品项目").clearOptions();
   ```

## daterange控件传值问题

问题描述：正常情况daterange控件可以直接绑定开始和结束时间戳（在搜索栏中可以正常绑定）。可有事会出现绑定后不生效，请求参数中直接以json字符串形式进行传输。

<img src="https://static.lee1224.com/aPaaSdocs/image-20210629185616134.png" alt="image-20210629185616134" style="zoom:67%;" />

<img src="https://static.lee1224.com/aPaaSdocs/image-20210629190029444.png" alt="image-20210629190029444" style="zoom: 50%;" />

<img src="https://static.lee1224.com/aPaaSdocs/image-20210629190302951.png" alt="image-20210629190302951" style="zoom:67%;" />

解决办法：

1. 请求时在领域处理json串，将时间戳转换成格式化日期（`yyyy-MM-dd HH:mm:ss`），PostgreSQL插入时间字段需以这种格式的字符串来传值。

   ```js
   if (contract.contract_date_range) { // 空字符串、null、undefined均为假; 空数组和空对象为真
       var date_range_json = JSON.parse(contract.contract_date_range);
       var start_date_str = new Date(Number(date_range_json.start)).toJSON().substring(0, 10);
       var end_date_str = new Date(Number(date_range_json.end)).toJSON().substring(0, 10);
       contract.contract_start_date = start_date_str;
       contract.contract_end_date = end_date_str;
   }
   ```

2. 响应时也是在领域把数据库中查到的开始结束时间，封装成json串，传到daterange控件才可正常显示。

   ```js
    _output[0].daterange_self = {"start":_output[0].contract_start_date,"end":_output[0].contract_end_date}
   ```

## EditorTable-编辑表格

获取数据：

|       方法        |         简要说明          |                           参数类型                           | 返回值类型 |
| :---------------: | :-----------------------: | :----------------------------------------------------------: | :--------: |
|   getInIndexes    |    获取指定多行的数据     |                      Array e.g. [0,1,2]                      |   Array    |
|    getInScope     |   获取指定范围内的数据    |            String e.g.  **'all'**   **'checked'**            |   Array    |
|    getInScope     |   获取指定范围内的数据    |                  String e.g.  **'focused'**                  |   Object   |
| getIndexesInScope | 获取指定范围的行的indexes | scope的取值有以下几种取值: **'all'** 全部数据; **'checked'** 勾选数据; **'focused'** 已修改数据; |   Array    |

获取控件：

|      方法       |               简要说明               |                       参数类型                       |      返回值类型      |
| :-------------: | :----------------------------------: | :--------------------------------------------------: | :------------------: |
| getRowAtIndexes | 获取指定位置的行控件对象ArrayRowCtrl | Array e.g. [0, 2, 3] (int整型参数疑似存在bug...弃用) |        Array         |
|  getColByName   | 获取指定名字的列控件对象ArrayColCtrl |                        String                        | Object(ArrayColCtrl) |

- 场景1：计算某列数值和进行判断

  ```js
  let rows = Page.getArrayCtrl('回款情况').getColByName('回款比例');
  if (rows.sum() > 100) {
    Page.alert('warning', '回款比例总值大于100%，请重新输入');
    let autoindex = Page.getArrayCtrl('回款情况').getInScope('focused').autoindex;
    Page.getArrayCtrl('回款情况').getRowAtIndexes([autoindex - 1])[0].getCtrl('回款比例').value = "";
  }
  ```

- 场景2：根据第一列选择的值来判断并且更新第二列的日期默认值

  ```js
  let create_time = Page.getCtrl('合同创建时间').value;
  let selected_key = Page.getArrayCtrl('回款情况').getInScope('focused').et_payment_node;
  let autoindex = Page.getArrayCtrl('回款情况').getInScope('focused').autoindex;
  
  if(!create_time){
      create_time = new Date().getTime();
  }
  // 1402171492564865024 预付回款
  // 1402171553059311616 发货回款
  // 1402171628267376640 验收回款
  if (selected_key == 1402171492564865024) {
      Page.getArrayCtrl('回款情况').getRowAtIndexes([autoindex - 1])[0].getCtrl('预计回款日期').value = create_time;
  } else if (selected_key == 1402171553059311616) {
      let date = new Date(Number(create_time));
      Page.getArrayCtrl('回款情况').getRowAtIndexes([autoindex - 1])[0].getCtrl('预计回款日期').value = date.setDate(date.getDate() + 30);
  } else if (selected_key == 1402171628267376640) {
      let date = new Date(Number(create_time));
      Page.getArrayCtrl('回款情况').getRowAtIndexes([autoindex - 1])[0].getCtrl('预计回款日期').value = date.setDate(date.getDate() + 30 + 40);
  }
  ```

  效果：

  <img src="https://static.lee1224.com/aPaaSdocs/image-20210630103251281.png" alt="image-20210630103251281" style="zoom:67%;" />

  
## 获取页面传参

```js
let linkparams = Page.getLinkParams().__linkparam;
if(linkparams.__pagestatus == 2){
    Page.getCtrl('contract_id').value = "";
}
```



  





# tips：

## Linux常用

```bash
####### 防火墙 #######
firewall-cmd --zone=public --add-port=7777/tcp --permanent
firewall-cmd --zone=public --add-port=7777/udp --permanent

firewall-cmd --reload
firewall-cmd --list-all  # 查看防火墙规则

####### 把SELinux永久设定为Permissive模式 #######
/usr/sbin/sestatus -v  # 查看当前状态
vi /etc/selinux/config
SELINUX=disabled
```



## 小站部署

### 方式1：git仓库托管

直接上传至GitHub或者gitee，开启pages功能即可。有条件可以上域名上CDN。

### 方式2：docker部署

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
   rm -rf /root/data/aPaaSdocs/*
   rm -rf /root/data/aPaaSdocs/.git
   rm -rf /root/data/aPaaSdocs/.gitignore
   rm -rf /root/data/aPaaSdocs/.nojekyll
   git clone /root/git/aPaaSdocs.git /root/data/aPaaSdocs
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

### 方式3：nginx

1. 安装nginx推荐docker安装，把网站目录以及配置文件映射到宿主机。

   - 目录结构

     ```
     .
     | -- conf.d
     |    | -- nginx.conf
     | -- dist
     |    | -- index.html
     |    | -- 50x.html
     | -- compose-nginx.yaml
     | -- startup.sh
     ```

   - compose-nginx.yaml

     ```yaml
     version: '3'
     
     # docker network create nginx_bridge
     networks:
       nginx_bridge:
         driver: bridge
     
     services:
       nginx:
         image: nginx:stable-alpine
         #image: nginx:1.19.1-alpine
         container_name: nginx-alpine
         restart: always
         privileged: true
         environment:
           - TZ=Asia/Shanghai 
         ports:
           - 8080:80
           - 80:80
           - 443:443
         volumes:
           - /etc/localtime:/etc/localtime:ro
           #- ./conf/nginx.conf:/etc/nginx/nginx.conf:ro
           - ./conf.d:/etc/nginx/conf.d
           - ./log:/var/log/nginx
           - ./dist:/opt/dist:ro
         networks:
           - nginx_bridge
     ```

   - 构建

     ```bash
     docker-compose -f ./compose-nginx.yaml up -d
     ```

     

2. 配置nginx

   ```nginx
   server {
     listen 80;
     server_name  your.domain.com;
   
     location / {
       alias /opt/dist/docs/;
       index index.html;
     }
   }
   ```

