# aPaaS基操

## Flycode

### 对象选择器自带查询条件添加

1. 添加查询参数

   <img src="https://lilpum-1257254543.file.myqcloud.com/PicGo/image-20220422175349449.png" alt="image-20220422175349449" style="zoom:50%;" />

2. 打开 `业务对象通用查询` 领域 拼接查询sql

   ![image-20220422175407916](https://lilpum-1257254543.file.myqcloud.com/PicGo/image-20220422175407916.png)

   日常屏蔽 自带查询

   <img src="https://lilpum-1257254543.file.myqcloud.com/PicGo/image-20220422175918138.png" alt="image-20220422175918138" style="zoom:70%;" />

   ```js
       if (!!extraparamsobj && !!extraparamsobj.cantfind) {
         wheresql = wheresql.concat(" and ( ");
         wheresql = wheresql.concat(objectmark, ".id=", extraparamsobj.cantfind);
         wheresql = wheresql.concat(")");
       }
   ```

   

### 时间格式化

```js
dateFormat("YYYY-mm-dd HH:MM:SS", new Date(NOW.time()));
dateFormat("YYYY-mm-dd HH:MM:SS", new Date("2022-12-24 12:58:57"));
/************************************** 格式化时间 */
function dateFormat(fmt, date) {
    var ret;
    var opt = {
        "Y+": date.getFullYear().toString(),        // 年
        "m+": (date.getMonth() + 1).toString(),     // 月
        "d+": date.getDate().toString(),            // 日
        "H+": date.getHours().toString(),           // 时
        "M+": date.getMinutes().toString(),         // 分
        "S+": date.getSeconds().toString()          // 秒
        // 有其他格式化字符需求可以继续添加，必须转化成字符串
    }
    for (var k in opt) {
        ret = new RegExp("(" + k + ")").exec(fmt);
        if (ret) {
            fmt = fmt.replace(ret[1], (ret[1].length == 1) ? (opt[k]) : padStart(opt[k], ret[1].length, "0"))
        }
    }
    return fmt;
}
/*************** padStart*/
function padStart(rawStr, targetLength, padString) {
    targetLength = targetLength >> 0; //truncate if number or convert non-number to 0;
    padString = String((typeof padString !== 'undefined' ? padString : ' '));
    if (rawStr.length > targetLength) {
        return String(rawStr);
    }
    else {
        targetLength = targetLength - rawStr.length;
        if (targetLength > padString.length) {
            padString += padString.repeat(targetLength / padString.length);
        }
        return padString.slice(0, targetLength) + String(rawStr);
    }
}
/*************** 格式化时间end*/
```

### 获取OSS地址

```js
var ossEndpoint = FLY.call("microservice.getCurEnvironmentHost", {}).data.ossEndpoint;

/*************** 获取OSS地址*/
function getAttachmentsKV(json) {
    if (json != null && json.length !== 0) {
        // var json = JSON.parse(data);
        var fileName = json.url;
        var type = json.type;
        var strDate = json.date;
        var date = new Date(Number(strDate));
        strDate = date.toJSON().substring(0, 10);
        strDate = strDate.replace(/-/g, "");
        var url = ossEndpoint;
        if (type.startsWith("image")) {
            url += "/" + fileName.substring(0, 3) + "/img/" + strDate + "/1008442/" + fileName;
        } else {
            url += "/" + fileName.substring(0, 3) + "/att/" + strDate + "/1008442/" + fileName;
        }
    }
    var att = {};
    att.key = json.filename;
    att.value = url;
    return att;
}
/*************** 获取OSS地址 end*/
```

### 字符串判空

```js
function isEmpty(obj) {
  if (obj === null) return true;
  if (typeof obj === 'undefined') {
    return true;
  }
  if (typeof obj === 'string') {
    if (obj === "") {
      return true;
    }
    var reg = new RegExp("^([ ]+)|([　]+)$");
    return reg.test(obj);
  }
  return false;
}
```

### 手写分页

```js
/** 
常规分页：
 1. 首先获取总条数
 2. 总页数(循环次数) 查询次数 = （总行数 + 每次查询行数 - 1）/ 每次查询行数
 3. 起始行 = (当前页数 - 1) * 每次查询行数
*/

var _pagingParam = IN.__paging;
var _pagesize = 20;
var _offset = 0;
if (!!_pagingParam) {
  _pagesize = _pagingParam.__pagesize;
  var pageindex = _pagingParam.__pageindex;
  _offset = pageindex == 0 ? 0 : (pageindex * _pagesize); // __pageindex从0开始 不需要减1
}

// TODO

if (!!_pagingParam) {
    OUT.__paging = { "__pageindex": _pagingParam.__pageindex, "__pagesize": _pagingParam.__pagesize, "__itemcount": count_res[0].count };  // 页数 每页显示数 总行数 
}

```

### 数字金额转中文大写

```js
/**************************数字金额转大写****** */
function numToChString(n) {
    if (!/^(0|[1-9]\d*)(\.\d+)?$/.test(n)) {
        return "数据非法";  //判断数据是否大于0
    }
    var unit = "千百拾亿千百拾万千百拾元角分", str = "";
    n += "00";
    var indexpoint = n.indexOf('.');  // 如果是小数，截取小数点前面的位数
    if (indexpoint >= 0) {
        n = n.substring(0, indexpoint) + n.substr(indexpoint + 1, 2);   // 若为小数，截取需要使用的unit单位
    }
    unit = unit.substr(unit.length - n.length);  // 若为整数，截取需要使用的unit单位
    for (var i = 0; i < n.length; i++) {
        str += "零壹贰叁肆伍陆柒捌玖".charAt(n.charAt(i)) + unit.charAt(i);  //遍历转化为大写的数字
    }
    return str.replace(/零(千|百|拾|角)/g, "零").replace(/(零)+/g, "零").replace(
        /零(万|亿|元)/g, "$1").replace(/(亿)万|壹(拾)/g, "$1$2").replace(/^元零?|零分/g,
            "").replace(/元$/g, "元整"); // 替换掉数字里面的零字符，得到结果
}
/**************************end****** */
```

### 数字补位 前面补0

```js
function prefixInteger(num, length) {
    return (Array(length).join('0') + num).slice(-length);
}
```



## UIflycode

### 下拉菜单联动

问题：联动下拉菜单（键值对形式例如：字典、数据源）清除父级value时字迹菜单option清除，获取不到文本直接显示键值。如图：

![image-20210629131236759](https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210629131236757.png)

![image-20210629131618317](https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210629131618317.png)

解决方法：

1. 父级菜单值改变时加载子菜单的option（例如：加载字典），并且指定根据父级的值来查询子级option并绑定。

   <img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210629133030005.png" alt="image-20210629133030005" style="zoom: 50%;" />

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

   ![image-20210629134114801](https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210629134114801.png)
   
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

### daterange控件传值问题

问题描述：正常情况daterange控件可以直接绑定开始和结束时间戳（在搜索栏中可以正常绑定）。可有时会出现绑定后不生效，请求参数中直接以json字符串形式进行传输。

<img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210629185616134.png" alt="image-20210629185616134" style="zoom:67%;" />

<img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210629190029444.png" alt="image-20210629190029444" style="zoom: 50%;" />

<img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210629190302951.png" alt="image-20210629190302951" style="zoom:67%;" />

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

### EditorTable-编辑表格

[开发者网站查看更多](http://apaas.wxchina.com:8881/2020/10/09/editortable-%e7%bc%96%e8%be%91%e8%a1%a8%e6%a0%bc-2/)

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

  <img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210630103251281.png" alt="image-20210630103251281" style="zoom:67%;" />

- 场景3：获取到下拉框选值，并带出text到文本编辑框

  <img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210715104114304.png" alt="image-20210715104114304" style="zoom:67%;" />

  ```js
  let edtable = Page.getArrayCtrl('edtable');
  let selected = edtable.getInScope('focused');
  let index = selected.autoindex - 1;
  
  let dropdown = edtable.getRowAtIndexes([index])[0].getPickerCtrl('dropdown');
  
  let option = dropdown.getOption().find(elem => {
      return elem.key == dropdown.value;
  });
  // option有值是同步更新文本框内容 删除选项时option为undefined 假 情况文本框内容
  option ? edtable.getRowAtIndexes([index])[0].getPickerCtrl('textinput').value = option.text : edtable.getRowAtIndexes([index])[0].getPickerCtrl('textinput').value = '';
  
  ```

#### 可编辑表案例

```js

let data = Page.getValue('eng_bom');
if (data && data.length > 0) {
  // 获取焦点行 数据
  let item = Page.getArrayCtrl('ordereditortable').getInScope('focused');
  if (item) {
    if (item.temporary_id) {
      Page.alert("warning", "该套件已经展开！不能重复展开！");
      throw "该套件已经展开！不能重复展开！";
    }
    let index = Page.getArrayCtrl('ordereditortable').getInScope('focused').autoindex - 1;
    // 根据索引获取控件
    let curRow = Page.getArrayCtrl('ordereditortable').getRowAtIndexes([index])[0];
    curRow.getCtrl("temporary_id").value = data[0].temporary_id;
    curRow.getCtrl("material_bom_type").value = "套件父项";
    curRow.getCtrl("frowtype").value = "Parent";
    let newIndex = index;
    let keys = Object.keys(item); // 获取所有控件名称

    data.forEach((bom) => {
      item.明细id = null;
      item.rank = null;
      item.con_actual_unit = '';  //合同销售单位
      item.con_saleqty = '';      // 合同销售数量
      item.productcode = bom.fmaterialidchild_fnumber;
      item.temporary_id = bom.temporary_id;
      item.parent_productcode = bom.materialid_fnumber;
      item.coefficient = bom.coefficient;
      item.material_bom_type = "套件子项";
      item.frowtype = "Son";
      let setter = ArrayCtrlSetter();

      newIndex = newIndex + 1;
      Page.getArrayCtrl('ordereditortable').insert([item], newIndex, setter);
      // 对象选择器复职基操
      let _temp = '[{' + '\'' + 'detail_product' + '\'' + ':' + '\'' + '{' + '\"' + 'text' + '\"' + ':' + '\"' + bom.fmaterialidchild_fname + '\"' + ',' + '\"' + 'key' + '\"' + ':' + '\"' + bom.fmaterialidchild_id + '\"' + '}' + '\'' + '}]';
      let _t = eval("(" + _temp + ")");
      setter.append('detail_product', '物料名称', 'fullvalue');
      Page.getArrayCtrl('ordereditortable').update(_t, [newIndex], setter);

      let _temp2 = '[{' + '\'' + 'project_three' + '\'' + ':' + '\'' + '{' + '\"' + 'text' + '\"' + ':' + '\"' + "123" + '\"' + ',' + '\"' + 'key' + '\"' + ':' + '\"' + "456" + '\"' + '}' + '\'' + '}]';
      let _t2 = eval("(" + _temp2 + ")");
      _t2[0].project_three = '{"text":"' + (item.project3 ? item.project3 : item.project_3_name) + '","key":"' + item.project_three + '"}';
      setter.append('project_three', '项目三', 'fullvalue');
      Page.getArrayCtrl('ordereditortable').update(_t2, [newIndex], setter);

      let _temp3 = '[{' + '\'' + 'article_number' + '\'' + ':' + '\'' + '{' + '\"' + 'text' + '\"' + ':' + '\"' + item.other_number + '\"' + ',' + '\"' + 'key' + '\"' + ':' + '\"' + item.article_number + '\"' + '}' + '\'' + '}]';
      let _t3 = eval("(" + _temp3 + ")");
      setter.append('article_number', '国际货号', 'fullvalue');
      Page.getArrayCtrl('ordereditortable').update(_t3, [newIndex], setter);

      let _temp4 = '[{' + '\'' + 'actual_unit' + '\'' + ':' + '\'' + '{' + '\"' + 'text' + '\"' + ':' + '\"' + bom.fchildunitid_fname + '\"' + ',' + '\"' + 'key' + '\"' + ':' + '\"' + bom.actual_unit_id + '\"' + '}' + '\'' + '}]';
      let _t4 = eval("(" + _temp4 + ")");
      setter.append('actual_unit', '需求发货单位', 'fullvalue');
      Page.getArrayCtrl('ordereditortable').update(_t4, [newIndex], setter);

      let row = Page.getArrayCtrl('ordereditortable').getRowAtIndexes([newIndex])[0];
      keys.forEach((key) => {
        if (row.getCtrl(key)) {
          row.getCtrl(key).readonly = true;
        }
      });
    });
  }
}
```



### 获取页面传参

```js
let linkparams = Page.getLinkParams().__linkparam;
if(linkparams.__pagestatus == 2){
    Page.getCtrl('contract_id').value = "";
}
```

### 获取下拉框dropdown控件的text

```js
let edtable = Page.getArrayCtrl('contract_details_edtable');    
// 获取下拉框中的text
let dropdown = edtable.getRowAtIndexes([index])[0].getPickerCtrl('套打名称');
let option = dropdown.getOption().find(elem => {
    return elem.key == dropdown.value;
});
```

### 修改控件属性

可使用setProperty()方法修改控件属性例如修改 placeholder 的值：

```js
option ? edtable.getRowAtIndexes([index])[0].getPickerCtrl('套打名称txt').setProperty('placeholder',option.text) : edtable.getRowAtIndexes([index])[0].getPickerCtrl('套打名称txt').setProperty('placeholder', '');
```

更多控件操作：[开发者平台](http://apaas.wxchina.com:8881/2020/05/19/%e6%8e%a7%e4%bb%b6%e6%93%8d%e4%bd%9c/)



### 系统对象（System.xxx）

|        方法         |         说明         |                            返回值                            |
| :-----------------: | :------------------: | :----------------------------------------------------------: |
|     **user()**      |     获取用户信息     | <img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210630115620254.png" alt="image-20210630115620254" style="zoom:33%;" /> |
|    **context()**    | 获取当前用户登录信息 | <img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210630115657977.png" alt="image-20210630115657977" style="zoom:33%;" /> |
|     **date()**      |  获取服务端当前时间  |                             Date                             |
| **functionCodes()** |     获取功能权限     |   返回当前用户的完整功能权限code的数组，Array形如[String]    |



# 数据权限

数权限控制就是在进行列表查询的sql语句后拼接上限制条件，根据用户不同的身份权限查询出不同的数据。

下面是一个例子：（跟随客户权限+我参与的拜访记录）

- 首先在领域的查询后添加上权限控制关键字并指定实体 `RULEOBJ("tn_cus_visitrecord")`

  ![image-20210714113858571](https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210714113858571.png)

- 然后在数据权限规则中配置规则

  <img src="https://lilp-1257254543.file.myqcloud.com/aPaaSdocs/image-20210714114815382.png" alt="image-20210714114815382" style="zoom:67%;" />

- 然后给对应角色挂上权限，done！

# tips：

## 文件预览

开源项目地址：https://gitee.com/kekingcn/file-online-preview

[演示](http://lilp.lee1224.com:9977/)

### 1.部署

- docker-compose部署（项目官方提供的即用镜像可以配置不能定制）

  ```yaml
  version: '2' #指定 compose 文件的版本
  services: #通过镜像安装容器的配置
    kkfileview:
      image: keking/kkfileview:latest #使用的镜像
      restart: always #当Docker重启时，该容器重启
      container_name: kkfileview
      ports:
        - 9977:8012 #端口映射
      volumes:
        - /root/kkfileview/file/:/opt/kkFileView-3.5.1/file/
        - /root/kkfileview/config/application.properties:/opt/kkFileView-3.5.1/config/application.properties  # 需要先拷贝一个文件出来再进行关联
  ```

- dockerfile部署

  ```dockerfile
  FROM ubuntu:20.04
  MAINTAINER lilp "liyanpeng@wxchina.com"
  ADD kkFileView-*.tar.gz /opt/
  COPY fonts/* /usr/share/fonts/chinese/
  RUN echo "deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse" > /etc/apt/sources.list &&\
  	apt-get clean && apt-get update &&\
  	apt-get install -y locales && apt-get install -y language-pack-zh-hans &&\
  	localedef -i zh_CN -c -f UTF-8 -A /usr/share/locale/locale.alias zh_CN.UTF-8 && locale-gen zh_CN.UTF-8 &&\
  	apt-get install -y tzdata && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
  	apt-get install -y libxrender1 && apt-get install -y libxt6 && apt-get install -y libxext-dev && apt-get install -y libfreetype6-dev &&\
  	apt-get install -y wget && apt-get install -y ttf-mscorefonts-installer && apt-get install -y fontconfig &&\
  	apt-get install ttf-wqy-microhei &&\
  	apt-get install ttf-wqy-zenhei &&\
  	apt-get install xfonts-wqy &&\
      cd /tmp &&\
  	wget https://kkfileview.keking.cn/server-jre-8u251-linux-x64.tar.gz &&\
  	tar -zxf /tmp/server-jre-8u251-linux-x64.tar.gz && mv /tmp/jdk1.8.0_251 /usr/local/ &&\
  
  #	安装 OpenOffice
  #	wget https://kkfileview.keking.cn/Apache_OpenOffice_4.1.6_Linux_x86-64_install-deb_zh-CN.tar.gz -cO openoffice_deb.tar.gz &&\
  #	tar -zxf /tmp/openoffice_deb.tar.gz && cd /tmp/zh-CN/DEBS &&\
  #	dpkg -i *.deb && dpkg -i desktop-integration/openoffice4.1-debian-menus_4.1.6-9790_all.deb &&\
  
  #	安装 libreoffice
      apt-get install -y libxinerama1 libcairo2 libcups2 libx11-xcb1 &&\
      wget https://kkfileview.keking.cn/LibreOffice_7.1.4_Linux_x86-64_deb.tar.gz -cO libreoffice_deb.tar.gz &&\
      tar -zxf /tmp/libreoffice_deb.tar.gz && cd /tmp/LibreOffice_7.1.4.2_Linux_x86-64_deb/DEBS &&\
      dpkg -i *.deb &&\
  
  	rm -rf /tmp/* && rm -rf /var/lib/apt/lists/* &&\
      cd /usr/share/fonts/chinese &&\
      mkfontscale &&\
      mkfontdir &&\
      fc-cache -fv
  ENV JAVA_HOME /usr/local/jdk1.8.0_251
  ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  ENV PATH $PATH:$JAVA_HOME/bin
  ENV LANG zh_CN.UTF-8
  ENV LC_ALL zh_CN.UTF-8
  ENV KKFILEVIEW_BIN_FOLDER /opt/kkFileView-4.0.0/bin
  ENTRYPOINT ["java","-Dfile.encoding=UTF-8","-Dspring.config.location=/opt/kkFileView-4.0.0/config/application.properties","-jar","/opt/kkFileView-4.0.0/bin/kkFileView-4.0.0.jar"]
  ```

  jar包 font/  以及dockerfile 在同一目录，application.properties文件要先存在对应目录下才行

  1. 构建镜像

     ```bas
     docker build -f Dockerfile -t lilp/online-preview:v1.0 .   # 构建镜像
     ```

  2. 运行容器

     ```bash
     docker run -d --name onlinePreview -p 9977:8012 -v /root/kkfileview/config/application.properties:/opt/kkFileView-4.0.0/config/application.properties -v /root/kkfileview/file:/opt/kkFileView-4.0.0/file lilp/online-preview:v1.0
     ```

  3. 配置文件(按需修改)

     ```properties
     #######################################不可动态配置，需要重启生效#######################################
     server.port = ${KK_SERVER_PORT:8012}
     server.servlet.context-path= ${KK_CONTEXT_PATH:/}
     server.servlet.encoding.charset = utf-8
     #文件上传限制
     spring.servlet.multipart.max-file-size=100MB
     spring.servlet.multipart.max-request-size=100MB
     ## Freemarker 配置
     spring.freemarker.template-loader-path = classpath:/web/
     spring.freemarker.cache = false
     spring.freemarker.charset = UTF-8
     spring.freemarker.check-template-location = true
     spring.freemarker.content-type = text/html
     spring.freemarker.expose-request-attributes = true
     spring.freemarker.expose-session-attributes = true
     spring.freemarker.request-context-attribute = request
     spring.freemarker.suffix = .ftl
     
     # office-plugin
     ## office转换服务的进程数，默认开启两个进程
     office.plugin.server.ports = 2001,2002
     ## office 转换服务 task 超时时间，默认五分钟
     office.plugin.task.timeout = 5m
     
     #文件资源路径（默认为打包根路径下的file目录下）
     #file.dir = D:\\kkFileview\\
     file.dir = ${KK_FILE_DIR:default}
     #openoffice home路径
     #office.home = C:\\Program Files (x86)\\OpenOffice 4
     office.home = ${KK_OFFICE_HOME:default}
     
     #缓存实现类型，不配默认为内嵌RocksDB(type = default)实现，可配置为redis(type = redis)实现（需要配置spring.redisson.address等参数）和 JDK 内置对象实现（type = jdk）,
     cache.type =  ${KK_CACHE_TYPE:jdk}
     #redis连接，只有当cache.type = redis时才有用
     spring.redisson.address = ${KK_SPRING_REDISSON_ADDRESS:127.0.0.1:6379}
     spring.redisson.password = ${KK_SPRING_REDISSON_PASSWORD:123456}
     #缓存是否自动清理 true 为开启，注释掉或其他值都为关闭
     cache.clean.enabled = ${KK_CACHE_CLEAN_ENABLED:true}
     #缓存自动清理时间，cache.clean.enabled = true时才有用，cron表达式，基于Quartz cron
     cache.clean.cron = ${KK_CACHE_CLEAN_CRON:0 0 3 * * ?}
     
     #######################################可在运行时动态配置#######################################
     #提供预览服务的地址，默认从请求url读，如果使用nginx等反向代理，需要手动设置
     #base.url = https://file.keking.cn
     base.url = ${KK_BASE_URL:default}
     
     #信任站点，多个用','隔开，设置了之后，会限制只能预览来自信任站点列表的文件，默认不限制
     #trust.host = file.keking.cn,kkfileview.keking.cn
     trust.host = ${KK_TRUST_HOST:default}
     
     #是否启用缓存
     cache.enabled = ${KK_CACHE_ENABLED:true}
     
     #文本类型，默认如下，可自定义添加
     simText = ${KK_SIMTEXT:txt,html,htm,asp,jsp,xml,json,properties,md,gitignore,log,java,py,c,cpp,sql,sh,bat,m,bas,prg,cmd}
     #多媒体类型，默认如下，可自定义添加
     media = ${KK_MEDIA:mp3,wav,mp4,flv}
     #office类型文档(word ppt)样式，默认为图片(image)，可配置为pdf（预览时也有按钮切换）
     office.preview.type = ${KK_OFFICE_PREVIEW_TYPE:image}
     #是否关闭office预览切换开关，默认为false，可配置为true关闭
     office.preview.switch.disabled = ${KK_OFFICE_PREVIEW_SWITCH_DISABLED:false}
     
     #是否禁止下载转换生成的pdf文件
     pdf.download.disable = ${KK_PDF_DOWNLOAD_DISABLE:true}
     
     #预览源为FTP时 FTP用户名，可在ftp url后面加参数ftp.username=ftpuser指定，不指定默认用配置的
     ftp.username = ${KK_FTP_USERNAME:ftpuser}
     #预览源为FTP时 FTP密码，可在ftp url后面加参数ftp.password=123456指定，不指定默认用配置的
     ftp.password = ${KK_FTP_PASSWORD:123456}
     #预览源为FTP时, FTP连接默认ControlEncoding(根据FTP服务器操作系统选择，Linux一般为UTF-8，Windows一般为GBK)，可在ftp url后面加参数ftp.control.encoding=UTF-8指定，不指定默认用配置的
     ftp.control.encoding = ${KK_FTP_CONTROL_ENCODING:UTF-8}
     
     #水印内容
     #例：watermark.txt = ${WATERMARK_TXT:内部文件，严禁外泄}
     #如需取消水印，内容设置为空即可，例：watermark.txt = ${WATERMARK_TXT:}
     watermark.txt = ${WATERMARK_TXT:lilpum测试}
     #水印x轴间隔
     watermark.x.space = ${WATERMARK_X_SPACE:10}
     #水印y轴间隔
     watermark.y.space = ${WATERMARK_Y_SPACE:10}
     #水印字体
     watermark.font = ${WATERMARK_FONT:微软雅黑}
     #水印字体大小
     watermark.fontsize = ${WATERMARK_FONTSIZE:18px}
     #水印字体颜色
     watermark.color = ${WATERMARK_COLOR:black}
     #水印透明度，要求设置在大于等于0.005，小于1
     watermark.alpha = ${WATERMARK_ALPHA:0.07}
     #水印宽度
     watermark.width = ${WATERMARK_WIDTH:180}
     #水印高度
     watermark.height = ${WATERMARK_HEIGHT:80}
     #水印倾斜度数，要求设置在大于等于0，小于90
     watermark.angle = ${WATERMARK_ANGLE:10}
     ```

### 2.使用

```js
var url = 'http://127.0.0.1:8080/file/test.txt'; //要预览文件的访问地址
window.open('http://127.0.0.1:8012/onlinePreview?url='+encodeURIComponent(base64Encode(url)));
```

新增多图片同时预览功能，接口如下

```js
var fileUrl =url1+"|"+"url2";//多文件使用“|”字符隔开
window.open('http://127.0.0.1:8012/picturesPreview?urls='+encodeURIComponent(base64Encode(fileUrl)));
```

举个不成熟的栗子：

```js
let showIMG = (url) => {
    window.open('http://lilp.lee1224.com:9977/onlinePreview?url=' + encodeURIComponent(btoa(url)));
}

/* 拿到的文件url是不完全相对路径，根据文件名生成规则 拼接出文件的完整路径 */
let data = Page.getCtrl('扫描件').value;

if (data != null && data.length != 0) {
    let json = JSON.parse(data);
    let fileName = json[0].url;
    let type = json[0].type;

    let strDate = json[0].date;
    let date = new Date(Number(strDate));
    strDate = date.toJSON().substring(0, 10);
    strDate = strDate.replace(/-/g, "");

    let url = "http://xxx.xxx.xxx.xxx:xxxx/apaas-storage";

    if (type.startsWith("image")) {
        // 
        url += "/" + fileName.substring(0, 3) + "/img/" + strDate + "/1008442/" + fileName;
    } else {
        url += "/" + fileName.substring(0, 3) + "/att/" + strDate + "/1008442/" + fileName;
    }
    showIMG(url);
}
```



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





未完待续～
