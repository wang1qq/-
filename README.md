随着科技不断发展，数据越来越丰富，数据分析的重要性也日益凸显。为了找到数据分析相关工作， 因此开始系统学习相关知识，并在这里记录自己的学习过程。

一.Power bi

1.介绍

Power BI 是微软推出的一款商业智能与数据可视化工具，能够连接多种数据源，对数据进行清洗、建模、分析，并通过交互式报表和仪表板直观展示分析结果。它广泛应用于企业数据分析、经营决策和业务监控等场景，是数据分析学习中非常重要的工具之一。

1）主要由几个部分组成：

Power BI Desktop：安装在电脑上的桌面工具，常用于导入数据、清洗数据、建立模型、制作报表。

Power BI Service：云端服务，用于发布、共享、查看仪表板和报表。

Power BI Mobile：移动端查看报表。

此外，还包括本地数据网关、嵌入式分析等组件，方便企业在不同场景中使用。
我这里因为个人学习安装了本地Power BI Desktop

2）核心优势

一是 可视化强，能快速做柱状图、折线图、地图、卡片图、矩阵等交互式图表；

二是 上手相对快，特别适合有 Excel 基础的人；

三是 与微软生态集成紧密，可以和 Excel、Teams、PowerPoint、Microsoft 365 等配合使用；

四是 适合企业共享与协作，支持工作区、权限管理、数据刷新和治理。

如果从学习角度来说，Power BI 常见的学习内容包括：
Power Query（数据清洗与转换）、数据建模（表关系、星型模型）、DAX（计算字段和度量值）、报表设计（可视化与交互）以及 发布与共享。

我的学习路径以项目驱动为为主导，围绕这些内容展开的。

2.项目实战

我这里主要是在小破站跟练学习。

URL地址为https://www.bilibili.com/video/BV1Hz4y1N7NH/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=730193a1fd84fa6069effc7dcaf71b52

其中用到的相关数据源，可自取：

链接: https://pan.baidu.com/s/1vPEtQ8EcjH3kQvSIZRMhtw 

提取码: dqjx

后续将进入实操阶段，并以图文结合的方式对学习过程进行记录

1）使用Power Query对数据进行清洗和转换

首先进行数据加载，如图所示，点击获取数据下excel工作簿找到已保存的 Excel 文件 Sales Analysis Report.xlsx，将其导入 Power BI，并利用 Power Query 对数据进行预处理。

<img width="1628" height="722" alt="image" src="https://github.com/user-attachments/assets/d1bb90af-6326-4584-95ee-686cb0d87d7f" />


接着，按照图示勾选矩形框内的 4 个工作表，然后选择右下角的“转换数据”，将数据加载到 Power Query 中。这四个表分别为客户表，产品表，区域表和销售订单表。
在自己实际使用中，选择自己需要处理的表即可。

<img width="1100" height="875" alt="image" src="https://github.com/user-attachments/assets/ab550d6b-df46-41bf-bfe7-41893a8bd33a" />

选择“销售订单”表后，找到箭头所指的两列，将数据类型从“小数”修改为“定点小数”，然后点击“添加新步骤”。这样做的原因是：小数属于浮点型，虽然取值范围较大，但可能存在一定的精度误差；而定点小数固定保留 4 位小数，更适合金额类数据和需要稳定精度的分析场景。点击“添加新步骤”则是为了利用 Power Query 记录操作过程，方便后续在处理其他类似表格时直接复用这些步骤，实现自动化处理。

<img width="1911" height="424" alt="image" src="https://github.com/user-attachments/assets/58e9c028-c1bb-4327-8ee8-ef2c77c39486" />

之后，在“添加列”功能区中选择“自定义列”，新建“销售收入”和“销售成本”两列。全部设置完成后，点击“文件”应用更改，并关闭窗口即可。数据清晰和处理到这里结束，后面进行开始建立数据模型。

<img width="846" height="170" alt="image" src="https://github.com/user-attachments/assets/daa97486-77c9-4344-b5bb-7ef066613f3a" />
<img width="876" height="555" alt="image" src="https://github.com/user-attachments/assets/fdd8c3f6-64d0-476b-af26-db09bbc56d32" />

2）数据建模

数据建模是根据分析需求，对原始数据表之间的关系、字段结构和计算逻辑进行设计与整理，从而形成适合数据分析和可视化展示的数据结构。简单来说，它的重点就是建立各个数据表之间的关系，这和在 Excel 中使用 VLOOKUP 函数进行数据匹配有些类似。

首先，需要建立日期表。那么，为什么需要日期表呢？

第一，便于按年、季度、月、周、日等时间维度进行筛选和分析。事实表中通常只有一列日期字段，而单独建立日期表后，可以进一步拆分出年份、季度、月份、周次、星期等属性，从而在设置筛选器和制作图表时更加方便。

第二，便于使用 DAX 时间智能函数。例如累计销售额、同比、环比、去年同期、年初至今等常见计算，通常都依赖一张连续且完整的日期表。若没有日期表，这类时间分析往往难以实现，或者计算结果不够准确。在 Power BI 中，常见的时间智能函数主要包括：TOTALYTD()（年初至今累计）、TOTALMTD()（月初至今累计）、TOTALQTD()（季度累计）、SAMEPERIODLASTYEAR()（去年同期）、DATEADD()（日期偏移）、DATESYTD()（返回年初至今的日期集合）、PREVIOUSMONTH()（上一个月）以及 PREVIOUSYEAR()（上一年）。这些函数常用于实现累计值、同比、环比等时间分析。

第三，能够使计算逻辑更加清晰。如果直接基于事实表中的日期列进行分析，往往会出现日期重复、日期缺失，或者同一天对应多条记录的情况。而日期表作为一张独立的维度表，每天只保留一个唯一日期，可以使数据模型更加规范，计算结果也更加稳定。

第四，有助于提升数据模型的规范性和可维护性。日期表相当于为整个数据模型建立了统一的时间维度。今后无论是销售表、订单表还是库存表，只要涉及日期字段，都可以与日期表建立关联，从而便于后续扩展和维护

使用以下DAX公式建立日期表dDate：
dDate = ADDCOLUMNS (

    CALENDARAUTO (),
    
    "Year", YEAR ( [Date] ),
    
    "Quartername", "Q" & QUARTER ( [Date] ),
    
    "Month", FORMAT ( [Date], "mmmm" ),
    
    "Month Number", MONTH ( [Date] ),
    
    "Quarternumber", ROUNDUP(MONTH ( [Date] )/3,0),
    
    "Yearquartersort", YEAR ( [Date] )*10+ROUNDUP (MONTH ([Date])/3,0),
    
    "Year-Quarter", YEAR ( [Date] )&"-"&"Q" & QUARTER ( [Date] ),
    
    "Year-Month",YEAR ( [Date] ) & "-"& FORMAT ( [Date], "mmm" ),
    
    "Yearmonthsort", YEAR ( [Date] )*100+MONTH ( [Date] )
    
)

该 DAX 公式用于创建日期表 dDate。首先通过 CALENDARAUTO() 自动生成完整的日期范围，再利用 ADDCOLUMNS() 为日期表添加年份、季度、月份及相应的排序字段。这样做的目的是为了便于后续按照年、季度、月份等维度进行筛选和展示，同时保证时间字段在图表中能够按照正确的时间顺序排列，并为后续的时间智能分析提供基础。


