# 导出STATA结果

**描述性统计、相关系数矩阵、组间均值差异检验和实证结果**是实证论文中最常见、同时也是最重要的四张基本表格。

Stata 官方命令 `est table` 仅能输出回归结果表格，效果差强人意，与多数期刊的格式要求相去甚远。

外部命令则主要使用 `logout` 命令输出**描述性统计、相关系数矩阵**，用 `ttable2` 或 `ttable3` 输出**组间均值检验表格**，`esttab` 或 `outreg2` **输出回归结果。**

对于 Stata 15 以前的用户而言，这些外部命令已经基本上可以满足我们的需求。**缺点是，每张表格需要单独存放于一个 Word 或 Excel 文档，事后尚需手动合并之。**

若想锦上添花，在 Stata 15 版本下，我们可以使用今天介绍的四个命令，**将论文中的所有表格自动汇总到一个 Word 文档中，大幅提高表格处理效率。**



## 基本表格的计算

### 描述性统计

```stata
* tabstat命令
webuse pig,clear
tabstat id week weight,s(N mean sd) f(%12.2f) c(s) // s()代表统计量（样本容量，均值，标准差等···；f()——format，设置格式; c()——column，以c(s)以统计量作为列

*sum命令
webuse pig,clear
sum id week weight,
```



### 相关系数表

``` stata
*官方命令pwcorr,不标注显著性
sysuse auto,clear
pwccorr make price mpg rep78 headroom trunk weight length turn displacement

*pwcorr_a,标注显著性;https://github.com/arlionn/pwcorr_a
sysuse auto,clear
pwccorr_a make price mpg rep78 headroom trunk weight length turn displacement
```



### T检验表格

```stata
sysuse auto,clear
ttable2 price wei len mpg, by(foreign)
ttable2 price wei len mpg, by(foreign) f(%6.2f)
```

### 回归结果表格

``` stata
cd "D:\课程作业\stata\导出STATA结果\reg2docx"
webuse pig,clear
reg weight id i.week
est sto m1
reg weight week i.id
est sto m2
reg weight i.id i.week
est sto m3

esttab m1 m2 m3 using estab.rtf,replace b(%12.3f) se(%12.3f) nogap compress s(N r2 ar2) ///
							star(* 0.1 ** 0.05 *** 0.01) // 系数格式，标准误格式 行间无空行压缩，显示N R2 AR2，显著性

--------------优化固定效应回归的导出结果-----------
cd "D:\课程作业\stata\导出STATA结果\reg2docx"
webuse pig,clear
reg weight id i.week
estadd local week"YES" //为固定效应打标签
estadd local id"NO"
est sto m1
reg weight week i.id
estadd local week"NO"
estadd local id"YES"
est sto m2
reg weight i.id i.week
estadd local week"YES"
estadd local id"YES"
est sto m3

esttab m1 m2 m3 using estab-promote.rtf,replace  b(%12.3f) se(%12.3f) nogap compress s(N r2 ar2) ///
									drop(*.week *.id) ///  //将生成的虚拟变量丢弃
									star(* 0.1 ** 0.05 *** 0.01) 
```



## Logout[^1]

将**单个**结果导出为**单个**文件保存

``` stata
webuse pig, clear

*导出描述性统计结果
logout, save(myfile) excel replace: tabstat id week weight,s(N mean sd) f(%12.2f) c(s) //导出为excel格式，我的电脑用不用了不知原因
logout,save(filename) word replace:tabstat id week weight,s(N mean sd) f(%12.2f) c(s) //导出为word格式
logout,save (filename) word replace: sum id week weight

*导出相关系数表
logout,save(corr) word replace:pwccorr price mpg rep78 headroom trunk weight length turn displacement
logout,save(corr) exel replace:pwccorr price mpg rep78 headroom trunk weight length turn displacement

*导出T检验表格
logout,save(ttable2) word replace:ttable2 price wei len mpg,by(foreign)
logout,save(ttable2) excel replace:ttable2 price wei len mpg,by(foreign)
```



## Sum2docx

将描述性统计结果导出Word文档

```stata
cd "D:\课程作业\stata\导出STATA结果\sum2docx"
sysuse auto,clear

sum2docx price-foreign using sum2docx, ///
		replace stats(N mean(%9.2f) sd min(%9.0g) median(%9.0g) max(%9.0g)) ///
		title("表：描述性统计") note("Data source:auto.dta")
```



## Corr2docx[^2]

将相关系数矩阵导出Word文档

```stata
sysuse auto,clear
corr2docx price mpg rep78 headroom trunk weight length turn displacement using corr2  //基础版，默认格式导出
corr2docx price mpg rep78 headroom trunk weight length turn displacement using corr2  ,star(*** 0.01 ** 0.05 * 0.1) fmt(%4.2f) title("表1：相关系数矩阵")  //设定显著性表示，格式，表名
```



## T2docx

将组件均值差异检验导出Word文档

```stata
sysuse auto,clear
t2docx price wei len mpg using t2docx ,by(foreign) star(*** 0.01 ** 0.05 *0.1) fmt(%4.2f) title("表：T检验")
```



## Reg2out

将回归结果导出Word文档

```stata
cd "D:\课程作业\stata\导出STATA结果\reg2docx"
sysuse auto,clear

*做两个线性回归
reg price mpg weight length
est store m1
reg price mpg weight lengrth foreign
est store m2

*做一个Probit回归
probit foreign price weight length
est store m3

*将结果输出至Word文档
```



## Putdocx [^10]

将STATA生成的结果导入word文档



### 创建Word文档

默认一次仅能创建一个文档

```stata
cd 路径 //使用cd命令设置文件保存路径

putdocx begin //在STAT中创建空白Word文档

putdocx save chuangjian.docx // 将文档保存
```

重新创建另一个文档

```stata
putdocx clear // 将原先创建的文件清除

putdocx save chuangjian.docx //报错，此时路径中已有同名文件

putdocx save chuangjian.docx,repalce //以新建的chuangjian代替原有路径中的同名文件

putdocx save chuangjian.docx,append // 将新创建的chuangjian 整合入原有路径中的同名文件
```



##  常用命令



### 在创建的Word文档中添加文本

```stata
putdocx paragraph //在WOrd中创建段落，相当于文本的容器

putdocx text("1+1")

putdocx text("=2") //默认在同一个段落中，会接续在前一个文本后方
```

呈现的结果为$1+1=2$



### 文本效果

```stata
putdocx text("1+1"),bold // 加粗
putdocx text("=2"),italic // 斜体
putdocx text("1+1"),underline(single) // 加单下划线
putdocx text("=2"),strikeout //加删除线

putdocx text("1+1=2"),font("宋体","20","red") // 设置字体格式，20号红色宋体

putdocx text("1+1=2"),linebreak //在文本后方进行段落内换行
```



## 在创建的Word文档中添加图像

```stata
putdocx clear
putdocx begin
putdocx paragraph

putdocx image 111.jpg // 若所需添加图片在当前文件目录中，直接输入文件名即可；若不在则需使用绝对路径
putdocx image 111.jpg // 再次添加图片于同意段落，两个图片呈现并排格式

putdocx image 111.jpg,linebreak 
putdocx image 111.jpg // 新添加的图片于下一行显示
```



## 

[^1]:[Stata入门——导出结果(1)描述性统计_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1E7411N7Vi/?spm_id_from=333.999.0.0&vd_source=ad8704328d18372fc83eff36c660ecb6)
[^2]:[Stata入门——导出结果(2)相关系数表_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV13E411H7td/?spm_id_from=333.999.0.0)
[^10]:[stata putdocx text 选项_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1sw411h7cb/?spm_id_from=pageDriver&vd_source=ad8704328d18372fc83eff36c660ecb6)











