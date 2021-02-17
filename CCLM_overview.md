# CCLM 简易操作流程

yycheng 2021.02.14

## 整体流程简述

这一部分来自于[官方网站](https://wiki.coast.hzg.de/clmcom/clm-community-home-92864627.html) 对于CCLM的整体介绍，因为官方介绍较为完整周全，所以大都是直接翻译

### 模式介绍

气候模式下的COSMO模式（COSMO-CLM）是在德国Wetterdienst局地模式（LM）及其后续COSMO模式的基础上发展起来的一种非静力区域气候模式。LM由德国气象局（DWD）开发，用于业务天气预报。同时，小尺度模拟联合会（COSMO）组织的其他几个气象服务机构也在使用和进一步发展它，称为COSMO模型。
COSMO-CLM由CLM社区开发和维护。自2005年以来，COSMO-CLM是德国Cimate研究社区的社区模式。该模型已被用于长达数百年的时间尺度和1至50公里的空间分辨率的模拟。
COSMO模式系统是数值天气预报和区域气候模拟的统一模式系统。开发工作与COSMO密切合作。新开发和其他源代码更改必须遵循“源代码开发步骤”，并且必须与源代码管理员沟通。

### 前处理总览

preprocess一共分为两个部分，一部分为制作气候态上的定常部分(Exrternal Data)，一部分是使用资料制作初边值 (initial & boundary Data)两部分都制作完毕后需要输入给下一步的 INT2LM 从而插值到模式场，进行启动

#### External Data

来自外部数据集的有关气候常数数据的信息是为COSMO-CLM创建强迫数据所需的信息来源之一。EXTPAR（数值天气预报和气候应用的外部参数）是编制外部数据集的主程序。交互式web前端WebPEP（用于预处理外部数据参数的web接口）可用于使用EXTPAR生成外部参数文件。

[WebPEP](https://tools.clm-community.eu/web_pep/gui/web_pep.php) 需要先拥有RedC的账号才能使用

在上面填写好需要的外部数据区域后就可以 preview 进行预览，确认区域合适之后提交申请，能在邮箱中收到制作好的外部数据

*remark* 

外部数据域的大小必须大于实际COSMO-CLM域的大小。在域的四个边的每一个都需要至少两行额外的网格点。建议选择较大的外部数据集域，这样就可以改COSMO-CLM域的大小，改变起始长度和起始纬度等，而无需创建新的外部数据集。EXTPAR另请参阅RedC:EXTPAR中的文档



EXTPAR另见[RedC](https://redc.clm-community.eu/projects/extpar/documents)中的文件：

EXTPAR步骤的简要说明：

数值天气预报（NWP）模型和气候模型需要地理数据集，如地球表面的地形高度、植被、陆地和海洋的分布，以及根据使用的方案，各种其他外部参数。EXTPAR软件包能够为COSMO和ICON模型生成外部参数。该软件可以在存储原始数据的UNIX或Linux系统上运行。用户（有经验的用户）可以运行脚本，用自己的规范（例如模型域）创建新的外部参数。
生成外部参数必须执行以下步骤：

1.必须指定目标网格。

​	支持的目标网格是：

​	旋转和非旋转经纬度网格（COSMO）

​	二十面体三角形网格（ICON）

​	在选定区域中具有可选的更高分辨率（nesting）。

2.考虑到位于目标网格元素内的所有原始数据元素，不同的原始数据集聚合到目标网格。如果目标网格的分辨率高于原始数据可用的输入网格，则执行插值或用最近邻填充目标网格，但在这种情况下，子网格尺度统计计算（例如，地形高度的子网格尺度方差）被忽略。
3初始和边界数据.必须检查所有不同的外部参数集是否相互一致。如果发生冲突，将自动设置默认值。在NetCDF输出中，给出了输入数据和处理软件的信息。

#### 初始和边界条件

COSMO-CLM需要的初始和边界数据比需要模拟的域覆盖更大的区域。

数据可以从

	- 再分析全球气候模型（GCMs）
	- 区域气候模型（RCMs）
	- COSMO-CLM（在双重嵌套的情况下）中获取

需要大量的气象变量作为初始和边界数据。考虑这个[表格清单](https://wiki.coast.hzg.de/clmcom/list-of-variables-needed-for-int2lm-105349360.html) 

这些变量是插值程序INT2LM的输入，该程序将数据传输到COSMO-CLM网格。但是，这些数据不能立即使用（除了较粗的COSMO-CLM模拟的输出），因为它们通常以不同的格式提供，并且必须转换为INT2LM可以读取的标准（所谓的“INT2LM就绪”数据）。<br>强制数据必须是netCDF（标准CF-1.0格式）或GRIB格式。建议边界数据的最大时间分辨率为6小时。根据Denis et al.（2003），强迫数据和应执行的区域模拟分辨率之间的水平分辨率的系数为12是最大的。有关数据要求的更多信息，请参阅COSMO文件的第五部分。

INT2LM的数据准备是通过所谓的转换器程序完成的。此处提供已存在转换器程序的GCM列表。DKRZ提供了几个已经转换的（“INT2LM ready”）数据集（如NCEP、ERA和COSMO CLM数据）。

有关完整列表和如何访问数据的信息，请访问 https://redc.clm-community.eu/projects/cclmdkrz/wiki/INT2LM_input_data_sets

### 模式调参、运行 

INT2LM和COSMO的NWP配置以及气候模拟的可行配置（namelist）参数的简短描述一起提供。这些配置可用于COSMO的当前操作设置和CLM社区推荐的COSMO-CLM设置。有关详细信息，请访问[名称列表工具门户](https://tools.clm-community.eu/NLT/devel/portal/index.php)。
WG SUPTECH强烈建议仅使用推荐的COSMO-CLM配置来模拟EURO-CORDEX域（有关设置的详细信息，请访问上面给出的名称列表工具）。对于其他配置，即使与建议的设置（例如，区域大小、分辨率、大气高度、时间步长、调谐参数等）仅略有不同，也需要进行测试以验证新配置的可靠性。这些试验应包括至少两个不同时间步的试验，并应分析PMSL的结果（例如RMSE）。如果数值相差很大，则应将结果与观测值进行比较，或进行第三时间步的模拟，以确定先前的模拟中哪个更可靠。请注意，时间步长在任何情况下都必须满足CFL标准。
如果要在欧洲以外的其他域上运行模型，应按照以下步骤进行操作，以获得合理的模型设置：

​	1.尝试获取名称列表作为起点：

​		a.检查是否有人已经在此域或类似域上执行了模拟。如果是这种情况，请尝试直接从namelist工具或人员获取namelist，并使用此namelist作为起点。
​		b.如果没有针对目标域的模拟，或者无法获得名称列表，请检查是否已针对类似气候区（中纬度、热带、海洋、大陆、平坦、山区）的域进行了模拟，并将此名称列表用作起点。
​	2.采用推荐的COSMO-CLM版本，并合并步骤1的名称列表中的特定于域的更改。
​	3.进行ERA中期模拟（理想情况下为30年，但至少为10年），并分析结果是否合理。
​	4.测试尽可能多的不同配置，为您的域找到最好的设置。对于这一步，Bellprat等人提出的方法。可使用（Bellprat。O、 S.Kotlarski，D.Lüthi，C.Schär（2012）：区域气候模式的客观校准。地球物理研究杂志，117，D23115，DOI:10.1029/2012JD018262）。
​	5.使用类似的设置但不同的时间步执行测试，以避免时间步问题导致错误的结果。一个时间步可以给出非常不同的结果，例如压力，降水与所有其他时间步。此时间步长不应用于气候模拟。对于EURO-CORDEX域，建议的配置给出了不现实的结果，时间步长为120秒。由于问题很可能是解决方案和域相关的，因此不能给出一般性建议。）<br>6.根据测试为您的域选择最佳设置。
7.把你的名单寄给安德烈亚斯·威尔(安德烈亚斯·威尔[在]b-图德)用于包含在名称列表工具中。
8.写一份关于你的测试和结果的简短报告，并将其发送给[工作组评估的协调员](https://wiki.coast.hzg.de/clmcom/wg-eval-98599155.html)。

浏览[论坛](https://redc.clm-community.eu/projects/forum/boards/)，如果遇到任何问题。

### 后处理  

不同的工具可用于模型输出的后处理和数据分析。后处理方法的简短描述也可以在COSMO CLM教程中找到[tutorial 2013](https://redc.clm-community.eu/attachments/2191/cosmo\u tutorial\u 2013.pdf)。
如果使用starter包（subchain），post job中已经执行了一些步骤（例如，分离变量、创建时间序列……）。[ncdump](https://www.unidata.ucar.edu/software/netcdf/workshops/2011/utilities/Ncdump.html)和[Ncview](http://meteora.ucsd.edu/~pierce/ncview_home_page.htmlhttp://meteora.ucsd.edu/~pierce/ncview\u home\u page.html)是简单易用的工具，用于获取有关NetCDF文件内容的信息和数据的基本可视化。
[ETOOL](https://redc.clm-community.eu/projects/etool)是计算某些气候统计数据的基本工具，由CLM社区提供。
[CCLM2CMOR](https://github.com/C2SM-RCM/CCLM2CMOR)是将COSMO-CLM输出重写为CORDEX标准的气候模型输出工具（另请参阅RedC中的附加信息）。它是CLM社区几个成员机构的联合开发。
NetCDF操作符（[NCO](https://github.com/C2SM-RCM/CCLM2CMORhttp://nco.sourceforge.net/)）和气候数据操作符（[CDO](https://code.mpimet.mpg.de/projects/cdo/)）为NetCDF文件的操作（CDO也适用于Grib）以及数据的处理和分析提供了许多可能性。

“工具”部分提供了一些其他工具的信息。(工具的合集) [TOOLS](https://wiki.coast.hzg.de/clmcom/tools-98599080.html)

### 文档合集

#### COSMO模型的科学文献

COSMO-CLM是COSMO模式的气候模式。两种模式的源代码对于主模型版本（例如5.0）的每个版本都是同步的，并且在该时间点是相同的。
COSMO模型的科学文档可以在[COSMO网页](http://www.cosmo-model.org/content/model/documentation/core/default.htm)上找到。5.00版（2013年11月）和5.05版（2018年2月）的所有文档部分现在都有DOI，并永久存储。文档包括COSMO模型文档、预处理程序INT2LM和[用户指南](http://www.cosmo-model.org/content/model/documentation/core/cosmo_userguide_5.06a.pdf)。定期更新。然而，它从来不是“最新的”，因为文档需要很多时间。最近的发展记录在[COSMO技术报告](http://cosmo-model.org/content/model/documentation/techReports/default.htm)和[COSMO通讯的文章](http://cosmo-model.cscs.ch/content/model/documentation/newsLetters/default.htm)中。

#### 模型系统（独立）组件的文档

##### 直接耦合的组件

- TERRA（表面和土壤模型；请参阅[COSMO文档，第二部分，第10和11部分]（http://www.cosmo-model.org/content/model/documentation/core/cosmoPhysParamtr.pdf））
- TERRA-URB（COSMO模型的城市冠层地表方案）
- FLake（淡水湖模型）
- ART（气溶胶和反应性痕量气体；例程可从KIT Karlsruhe获得；请联系Bernhard Vogel）

##### 通过标准耦合器[OASIS](https://verc.enes.org/models/software-tools/oasis)耦合的组件

- [Community Land Model](http://www.cesm.ucar.edu/models/clm/) (SVAT with dynamical vegetation; alternative to TERRA)
- [Veg3D](https://redc.clm-community.eu/attachments/download/2189/veg3d_soil_vegeation_model.pdf) (SVAT; alternative to TERRA)
- [TRIMNP](https://redc.clm-community.eu/attachments/download/2188/trim_chapter2.pdf), [NEMO](http://www.nemo-ocean.eu/), and [ROMS](https://www.myroms.org/) (regional ocean models)
- [CICE](https://github.com/CICE-Consortium/About-Us/wiki) (sea ice model)
- [ECHAM](https://www.mpimet.mpg.de/en/science/models/mpi-esm/echam/) (global climate model; with OASIS a two-way nesting is possible)

#####  INT2LM & COSMO 编程标准

The COSMO model is written in FORTRAN. The code development is based on the [European standards for writing and documenting exchangeable FORTRAN 90 code](https://redc.clm-community.eu/attachments/download/2190/F90Style.pdf). Further regulations are specified in the “COSMO standards for source code development” (Appendix F of the CLM-Community Agreemnet; see [Terms & conditions](https://wiki.coast.hzg.de/clmcom/terms-conditions-98599061.html) for the latest version). 



# CCLM SP包简单小结

