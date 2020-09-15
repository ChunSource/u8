# <center>**u8工作笔记**</center>

## 软件
### "批次管理"打开

> 库存管理->选项->业务->批次管理

> 库存档案->控制->批次管理


### "会计科目"的"受控系统"(应收、应付)

> 基础设置->基础档案->财务->跨级科目

### 应收,应付"生成凭证"没有单据
> 需要在应付应收找到票据栏目审核


## 开发
### EAI接口 XML数据

    <ufinterface roottag='' billtype='' docid='' receiver='' sender='' proc=''  codeexchanged='' exportneedexch='' exportneedexch:= ''  timestamp='' check=''  paginate='' 
    `version='2.0' >
    `</ufinterface>

说明如下：
>roottag 单据模版名，如:客商档案:customer 客商分类:customerclass

>billtype  系统用 可填空 

>docid    唯一编号 可空

>receiver  接收方 可填U8

>sender  发送方编码 即注册的外部编码 (必填)

>proc操作码  导入时 Add /edit/delete 导出时:Query 该字段必填

>codeexchanged 编码是否已转换

#### OPENAPI 

- 添加入库单提示 在对应所需名称或序数的集合中，未找到项目
- ````
	CP-U8V13.0-100985-180828-ST.rar  打这个补丁
	参考链接:https://download.csdn.net/download/feed12/11828734
	````

- 入库单必须有收发类别(入库类别)

- sqlserver 调用dll 安全限制可执行

	 `ALTER DATABASE 你的U8数据库名称 SET TRUSTWORTHY ON`

### API资源管理器(坑)
- 插件开发完成之后，依赖的配置文件等，必须放到U8根目录，因为程序的运行目录是在U8根目录 
- 用到u8Login 需要替换debug版的dll到  %u8soft&/ufcomsql里面, 这是很大的坑(exe调用需要，dll貌似不用)(https://github.com/ChunSource/u8)
- `MSXML2.IXMLDOMDocument2 domHead = new MSXML2.DOMDocumentClass();`
- `string id = (String)head.selectSingleNode("//z:row").attributes.getNamedItem("csocode").nodeValue;`
- (销售订单中)  业务类型的数据类型为string 官方示例为int，会报错
- (销售订单中)  是否订单BOM  borderbom   这个类型官方是ENUM建议用string，因为返回是或否
- (发货单中) `fOutQuantity`  `fOutNum`  决定了已经入库的数量
- (发货单中)  获取单据前缀格式失败  原因是cvouchtype(单据类型编码)没有填，默认为"05"
- (其他出库单中) `broker.AssignNormalValue("cnnFrom", new ADODB.Connection());` \n 和`broker.AssignNormalValue("domPosition", new System.Object());` 不使用可以传null
- (库存管理API中) 所有类型单据，事件的head都是 `//rs:data/rs:update/rs:original/z:row`
- (库存管理API中) 提示类型不匹配很可能是需要打补丁，和上面那个一样
- (库存管理API中-目前遇到的是入库单) 审核接口最后三个可以不传，其他按实际情况
- (库存管理API) "审核、弃审、删除   该单据XXXXXXXX已经被其他人修改，请刷新后重新弃审，从视图中获取原来的ufts，填到参数中
- (销售管理) 目前知道的是销售和发货事件，body都是null
- **为了避免坑，BussinesObject一定要对应所有的字段，解决bug的时间够你填字段好多次了，实在不想手写，就写个Python脚本处理**
- 缺少根元素，参数没填对，API没填对
- (请购单中) `0非编辑 1修改  2新增`  **填错**  导致 `=附近语法错误`
- "editprop" //编辑属性：A表新增，M表修改，D表删除，string类型
- (销售订单) 表体<u>预测单号填不对</u>会提示:**存货编码已被其他人修改无法生成单据**
- 如果不会用XML可以new 一个broker获取BO，填好数据后转成XML的方式，有函数转的
- 形态转换单，审核后事件只有"转换后"的表体内容，审核前事件才是完整的
- EAI导入BOM ,如果含有自由项，务必开启结构自由项
