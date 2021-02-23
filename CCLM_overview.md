# CCLM 整体流程简述

yycheng 2021.02.14

## 从官网进行翻译的部分

这一部分来自于[官方网站](https://wiki.coast.hzg.de/clmcom/clm-community-home-92864627.html) 对于CCLM的整体介绍，因为官方介绍较为完整周全，所以大都是直接翻译；部分网页需要RedC的账号，这部分需要先在CCLM官网进行申请之后获得，才能浏览部分官方文档，以及使用namelist工具，外部数据工具等。

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



# CCLM 简易操作流程

下面简单的从两个实际例子说明一下CCLM的使用步骤，简单的给出了step by step 方式进行运行的流程；

## 获取外部数据

从此前提到的获取外部数据的[外部数据](https://tools.clm-community.eu/web_pep/gui/web_pep.php)中就能够获取，网页中的部分参数和namelist中的&LMGRID部分相似，在下表给出 

简单地说，CCLM使用了rotated coordinates，并通过**原点 + 格点数目 + 格距 + 模式区域左下角起点 **来确定模式区域（当然也包括输入数据的区域，在未经旋转的情况下，例如**未经旋转**的经纬度坐标数据，只需填写 pollat = 90. pollon = -180）

对于旋转坐标，需要先确定区域模式的中心经纬度（作为旋转坐标系的坐标原点） origin lon，之后就能获得旋转坐标下的北极点经纬度 pollat / pollon，并填写到**int2lm**的namelist中的**&LMGRID**下的 pollat / pollon 中

同样，在确定了 pollat / pollon ,可将模式区域右下方的格点的经纬度转换到rotated coordinates 下（转换计算器在上面的外部数据中有提供），并且填写到 下表中的 **startlon / lat** 部分，以及 &LMGRID 下的 startlon_tot 等参数中。

| origin lon / lat     | 旋转系统原点（0,0）的地理经度（填写原本需要的模式中心的经纬度） |
| -------------------- | ------------------------------------------------------------ |
| **ie_tot / je_tot ** | **X / Y 方向的格点数量**                                     |
| **startlon**         | **STARTLON是旋转坐标中域左下角的经度。<br/>STARTLON的值通过在原点lon处以零开始进行计数。（填写经过origin lon/lat 转换之后的模式左下格点的经纬度，上面网页中有简单的尖酸部分）** |
| **startlat**         | **和上述相似**                                               |
| **dlon / dlat**      | **格点间距大小**                                             |

 勿忘外部数据需要准备的较模式区域稍大（至少两个格点）经纬度不需要完全一致

## 模式运行脚本的结构

下面介绍模式的step-by-step的运行，模式编译出来分为两部分，即是 int2lm 作为制作初边值的部分 和 cclm 作为运行的部分，脚本也分为两块进行执行

模式脚本是一个shell文件，需要在开头用export设置好环境变量，把路径也填写到变量中，以及NPX NPY 等传入一些并行拆分的数目，使用 cat 将控制运行的namelist输入到 INPUT* 文档中，最后在同一个环境中使用 mpirun 运行脚本， 下面是一个示例：

```
#!/bin/bash
#
#################################################
# batch settings for INT2LM run on mistral at DKRZ
#################################################
#SBATCH --job-name=sp_int2lm
# ...

# ulimit: the setting of the stack size instead of using "ulimit -s unlimited",
#         together with the --propagate=STACK in the "srun" command (see below),
#         speeds up the job considerably; in my test cases between 10% and 15%;
#         the value 102400 is the default maximum stack size on MISTRAL
#
# ulimit -s 102400

#
export FORT_BUFFERED=true
export decfort_dump_flag=true
#
export MXM_LOG_LEVEL=ERROR

set -e

#### !!!! Adjust this directory !!!!
INT2LMDIR=/m2data2/yycheng/CCLM/src/INT2LM-v2.05_clm2 #编译出的CCKN位置
CASEDIR=/m2data2/yycheng/cptp_test/ERA5_to_cclm_cptp #处理此次CASE的文件夹
DATADIR=/m2data2/yycheng/data_stage/cptp # 所有运行数据的位置
# change to new work path
cd ${CASEDIR}/log_int2lm

# clean the directory
rm -rf I* O* Y*

NPX=12
NPY=8
NPIO=0
NP1=`expr $NPX \* $NPY`
NP=`expr $NP1 + $NPIO`

#################################################
# cat together the INPUT*-files
#################################################

cat > INPUT <<**
 &CONTRL
  ...
/END
 &GRID_IN
  ...
 /END
 &LMGRID
  ...
 /END
 &DATABASE
 /
 &DATA
  ...
 /END
**

#################################################
# run the program
#################################################

# you have to adapt the way how to invoke the binary
echo ----- start INT2CLM
mpirun ${INT2LMDIR}/bin/int2lm.exe
# ${SPDIR}/src/int2lm/bin/int2lm.exe
# srun -l --propagate=STACK --cpu_bind=verbose,cores --distribution=block:cyclic  ${SPDIR}/src/int2lm/bin/int2lm.exe
echo ----- INT2CLM finished, cheer up.
```

## 制作初始、边界条件 INT2LM

这部分由  INT2LM 来控制，nanelist 包含下面六个部分

| CONTRL  | 控制模式运行的参数                   |
| ------- | ------------------------------------ |
| GRID_IN | 确定驱动场格点的区域以及大小         |
| LMGRID  | 确定LM格点（模式区域）的区域以及大小 |
| DATA    | 控制输入以及输出的格点               |
| PRICTR  | 控制某个 point 的输出                |

所有NAMELIST组必须按照上面给出的顺序出现在输入文件INPUT中。 为所有参数设置了默认值，您只需要指定必须与默认值不同的值即可。 用户可以在INT2LM的运行脚本中指定NAMELIST变量，然后该INT2LM将创建INPUT文件。 有关运行脚本的详细信息，请参见官方文档中的教程部分 tutorial2018的附录A.3（NWP）和/或B.3（RCM）。

### 模式区域控制 LMGRID

本节说明&LMGRID的变量，要指定您选择的域，您必须选择以下参数：

- 1.旋转坐标中域的左下网格点。
- 2.纵向和横向的水平分辨率，以度为单位。
- 3.网格点中区域的大小。（格点数）
- 4.旋转的北极的地理坐标，以指定旋转（或域的旋转角度）。
- 5.通过指定垂直坐标参数来获得垂直分辨率。

所有这些参数都可以通过名称列表组LMGRID的名称列表变量来确定。 在下文中解释了变量及其含义。
注意：要描述外部参数的更大范围，您必须为上面的1）-4）指定值。 不能为外部参数提供垂直分辨率，因为它们是纯水平的。

#### 水平格点

COSMO模型使用旋转的球面坐标系，极点在该坐标系中移动并且可以定位为使得赤道穿过模型域的中心。 因此，对于全球范围内的任何有限区域模型域，可以将因经线辐合而产生的问题最小化。
在所选外部参数数据集的区域内，您可以为测试指定任何较小的区域。 请注意，由于插值的原因，您必须在外部参数域的每一侧至少少选择一个网格点。通过设置以下名称列表变量来指定水平网格。 在这里，我们展示了一个中欧地区的示例。

```'
startlat_tot = -17.0, ! lower left grid point in rotated coordinates 
startlon_tot = -12.5, dlon=0.0625, dlat=0.0625, ! horizontal resolution of the model 
ielm_tot=225, jelm_tot=269, ! horizontal size of the model domain in grid points 
pollat = 40.0, pollon = -170.0, ! geographical coordinates of rotated north pole (or: angles of rotation for the domain)
```

#### 选取垂直格点

要指定垂直网格，必须给出垂直层的数量及其在大气中的位置。 这可以通过Namelist变量 kelm_tot 并通过定义垂直坐标参数列表来完成

这是设置模型域中最困难的部分，因为需要有关垂直网格的一定知识。  COSMO模型方程式是使用具有广义垂直坐标的地形跟踪坐标系来表示的。 请参考科学文档（第一部分：动力学和数值）以获取有关垂直坐标系的完整说明。
对于实际应用，INT2LM提供了三种不同的选项来指定垂直坐标。 这些选项由Namelist变量ivctype选择(具体查询CCLM使用手册)

一个使用例

```
ivctype = 2, 选择垂直坐标的类型，这里是标准的基于高度的坐标
vcoord_d = sigma1, sigma2, ..., sigman 垂直坐标的参数，来控制垂直层的结构
vcflat = 11430.0 levels变换回z坐标的高度
kelm_tot = 40 垂直层数
```

### 确定驱动场格点 GRID_IN

驱动场由namelist GRID_IN 来控制

#### 可行的驱动场模式

使用INT2LM，可以将来自多个驾驶模型的数据插值到COSMO网格。 到目前为止，NWP模式支持以下输入模型：

- ICON：DWD的新非静液压整体模型（具有二十面体模型网格）
- GME：DWD的旧静液压整体模型和二十面体模型网格
- IFS  ：ECMWF的全球光谱模型； 数据在高斯网格上传递
- COSMO：来自较粗的COSMO模型运行的数据可用于驱动更高分辨率的运行

RCM模式中，下面的模式是支持的：

- NCEP：美国国家环境预测中心的数据
- ERA40，ERA5I，ERA20C：ECMWF的再分析数据
- JRA55：日本气象厅全球再分析数据
- ECHAM5 / 6：来自MPI汉堡全球模型的数据
- ECHAM5 / 6：来自MPI汉堡全球模型的数据
- REMO：来自MPI汉堡的区域气候模型REMO的数据。
- CMIP5 GCMs: CGCM3, HadGEM, CNRM, EC-EARTH, Miroc5, CanESM2等

这些模型的数据经过预处理（pre-pre-processed），以使数据具有通用格式，可以由INT2LM处理。（在int2lm之前已经被官方准备好）

#### 确定格点：标准格点

名称列表组GRID_IN指定所选驾驶模型的网格的特征。
大多数输入模型使用矩形网格，可以通过以下名称列表变量指定该网格：

```
startlat_in_tot   总输入域的左下网格点的纬度
startlon_in_tot   总输入域的左下网格点的经度
dlon_in           输入网格的纵向分辨率
dlat_in           输入网格的纵向分辨率
ie_in_tot         纵向网格点大小（数目）
je_in_tot         纬度网格点大小
ke_in_tot         垂直网格点大小
pollat_in         北极的地理纬度（如果适用；或旋转角度）
pollon_in         北极的地理经度（如果适用；或旋转角度）
```

**未经旋转**的经纬度坐标数据，只需填写 pollat = 90. pollon = -180

#### 确定格点：GME模式输入

这部分暂时没有使用经验，省略，具体可以参考原本的模式手册，后面没经说明均是如此。

#### 确定格点：ICON模式输入

### 确定数据特征 DATA

本节说明名称列表组DATA的一些变量。 在此“名称列表”组中，必须指定目录和数据的某些其他规范。
对于每个数据集，您可以通过将适当的“名称列表”变量（如下所示）设置为“ grb1”以使用DWD GRIB1库读取或写入GRIB1数据，来指定用于I / O的格式和库。

- “ apix”可通过ECMWF grib_api读取GRIB1或GRIB2数据。

- “ api2”以使用ECMWF grib_api写入GRIB2数据。

- “ ncdf”以使用NetCDF库读取或写入NetCDF数据。

  请注意，Climate community 偏爱NetCDF，而NWP community则使用GRIB格式。

#### COSMO_Model 的外部参数

必须为COSMO模型域的外部参数的数据集指定以下变量：

```
ie_ext = 965，je_ext = 773	外部参数的大小、格点数目
ylmext_lfn ='cosmo_d5_07000_965x773.g1_2013111400'	  外部参数文件的名称
ylmext_cat ='/tmp/data/ext1/'					  	目录
ylmext_form_read ='grb1'	指定数据格式（或：'ncdf'，'apix'）
```

#### COSMO_Model 驱动场的外部参数

INT2LM还需要用于粗网格驱动模型的外部参数。 必须为此数据集指定以下变量：

```
 yinext_lfn ='invar.i384a'，GME的外部参数文件的名称
 yinext_lfn ='icon_extpar_0026_R03B07_G_20141202.nc'，全球ICON域的外部参数文件的名称
 yinext_lfn ='icon_extpar_ <name-of-region>_R03B07_20141202.nc'，区域外部参数的名称
 yinext_cat ='/ tmp / data / ext2 /'	此文件的目录
 yinext_form_read ='grb1'，用于指定数据格式（或'apix'或 'ncdf'）
```

这部分是用于GME\ICON模式作为粗分辨率的驱动场的时候，需要补充外部参数。（在前面省略）

#### 驱动场的输入数据

下面的参数需要指定驱动场的输入数据的一些信息：

```
yin_cat ='/ tmp / data / input /'，输入数据的目录
yin_hhl ='/ tmp / data / input /'，垂直网格HHL文件的名称（仅COSMO和ICON GRIB2必需）
ybitmap_cat ='/ tmp  / data / btmp /'，bitmap文件的目录（仅适用于GME）
ybitmap_lfn ='bitmp384'	bitmap文件的名称（仅适用于GME）
yin_form_read ='grb1'，用于指定数据格式（或'apix'或'  ncdf'）
ytunit_in ='f'，指定时间单位，由此指定输入数据的文件名
yinput_type ='forecast'，指定时间单位，由此指定输入数据的文件名
```

一些注意事项：

驱动场的外部参数：ICON外部参数文件的文件名包含日期。 该日期指定外部参数的生成日期。 这样可以确保使用正确的ICON文件。
HHL文件：ICON和COSMO-Model都使用广义的垂直坐标。 这意味着三维网格的构建是一个更复杂的过程，这不容易完成。 因此，完整的3D网格存储在所谓的HHL文件中。 在GRIB1中，必要的元数据可以存储在GRIB记录中，以重建三维垂直网格。  GRIB2中不再是这种情况。 因此，如果使用GRIB2，则INT2LM必须为COSMO模型网格以及驱动场网格读取HHL文件。  HHL文件始终以GRIB2编写。

#### COSMO模式格点输出

下面是INT2LM部分的输出数据，用以接下来的COSMO的模拟

必须为COSMO模型的输出数据集指定以下变量：

```
lm_cat ='/ tmp / data / out /'，输出结果的目录
ylm_form_write ='grb1'，输出数据的数据格式（或'api2'  ，'ncdf'）
ytunit_out ='f'，以指定时间单位，从而指定输出数据的文件名。有不同的变体来指定输入和输出数据集的文件名。
```

同样，NWP和CLM的应用在此处使用不同的设置，分别由变量 ytunit_in 和 ytunit_out 指定。 对于NWP应用程序，主要使用默认值“ f”，而CLM-Community则采用设置“ d”。 有关这些设置的更详细说明，请参见《 INT2LM用户指南》（第五部分：预处理，第6.5节：文件名约定）。

### 控制INT2LM的参数 CONTRL

本节仅说明名称列表组CONTRL的几个基本变量。 该组中的大多数参数用于指定INT2LM的特定行为，具体取决于对COSMO模型的特殊要求。 请参阅《 INT2LM用户指南》（第五部分：预处理，第7.1节）以获取所有变量的完整文档。

#### 预报时间，区域分解，驱动场

在任何情况下都必须指定以下参数，不再赘述。
指定预测的日期和时间长度

```
ydate_ini	预报开始的日期
hstart	起始时间，相对于预测的起始时间
hstop	停止时间，相对于预测的起始时间
hincbound	以小时为单位的边界更新增量
```

指定需要计算的数据

```
linitial	计算初始数据
lboundaries	计算边界数据
```

指定驱动场

```
yinput_model	驱动模型的名称指定为字符串。 可以使用以下名称：“ ICON”，“ GME”，“ IFS”，“ COSMO”，“ CM”。
```

指定处理器数量

```
nprocx	在X方向（纵向）上的处理器数
nprocy	在Y方向（纵向）上的处理器数
nprocio	用于异步IO的处理器数（通常为0）
```

这部分在运行脚本的前面通常以变量的形式写入

#### 基本控制参数

根据COSMO模型的配置，必须提供其他初始和/或边界数据。 要控制它，可以使用以下名称列表参数。 请注意，其中某些开关可能需要附加的外部参数和/或驱动模型的附加输出字段。

```
lforest	提供常绿和落叶林分布的其他外部参数
lso	提供额外的外部参数以运行亚网格规模的地形方案
lprog_qi	内插驱动模型中的云冰
lprog_qr_qs	内插驱动模型中的雨雪
lmulti_layer_in	使用多层输入土壤层
lmulti_layer_lm	计算多层输出土壤水平
llake	为FLake模型提供附加的外部参数
lprog_rho_snow	从行驶模型中插入预测的雪密度
lfilter_oro	为地形应用过滤器
lfilter_pp为压力偏差应用过滤器（垂直插值后）
```

#### NWP 和 RCM 之间切换的开关

在NWP或RCM应用程序之间进行选择的主要开关是（在两个模型中均为INT2LM和COSMO模型），是Namelist变量**lbdclim**。 对于NWP模式，此变量为**.FALSE.**。
如果将其设置为**.TRUE.**。，则将为每个输出集的COSMO模型提供其他边界数据。 这些是缓慢变化的变量，例如海面温度，植物覆盖率，叶面积指数等。在仅运行几天的典型NWP应用中，这些变量保持恒定。 当然，这在气候应用中是不合理的。

```
lbdclim	运行气候模式
```

#### RCM模式运行的其他有趣选项

```
itype_w_so_rel	选择相对土壤水分输入的类型
itype_t_cl	选择气候温度的来源
itype_rootdp	选择根深的外部参数的处理
itype_ndvi	选择植物覆盖和叶面积的处理
itype_calendar	指定公历 或每年有360天的日历或每年有365天的日历
luse_t_skin	将skin teamperature用于surface
```

再次注意，对于这些名称列表变量的某些设置，可能需要其他外部参数。
同样，有关这些参数的完整文档，请参阅《 INT2LM用户指南》（第五部分：预处理，第7.1节）。

## 模式运行部分 CCLM

使用之前int2lm部分产生的初边值条件，下面可以开始模式的运行。运行部分的namelist有下面的内容来控制

```
LMGRID	指定模型运行的网格的域和大小
RUNCTL	参数用于模型运行
TUNING	参数用于调整物理和动力学
DYNCTL	参数用于绝热模型
PHYCTL	参数用于非绝热模型
DIACTL	参数用于诊断计算
NUDGING	控制数据同化
INICTL	参数模型变量的初始化
IOCTL	控制环境
GRIBIN	控制GRIB输入
GRIBOUT	控制GRIB输出
SATCT	控制合成卫星图像的计算
EPSCTL	控制集合运行
```

所有NAMELIST组必须按照上面给出的顺序出现在相应的输入文件INPUT_ \**中。 为所有参数设置了默认值，您只需要指定必须与默认值不同的值即可。 用户可以在COSMO模型的运行 shell 脚本中指定NAMELIST变量，然后该脚本将创建INPUT_ **文件。 有关运行脚本的详细信息，请参见附录A.3（NWP）和/或B.3（RCM）。
我们在这里仅描述一些基本变量。 有关详细说明，请参见《 COSMO模型用户指南》（第七部分：用户指南，第7节）。

#### 模式区域 LMGRID

模型域由“名称列表”组LMGRID中的变量指定。 该组与INT2LM中的相应组相似，但变量较少。 例如，在COSMOModel中，您不能指定垂直坐标参数。 这些通过初始数据（通过Grib或NetCDF标头或hhl文件）提供给COSMO模型。
这些是LMGRID中的变量，带有一些示例设置：

&LMGRID 

| Variable | COSMO_EU | COSMO_DE | for RCM-Mode |
| -------- | -------- | -------- | ------------ |
|ie_tot |665 |421 |101|
|je_tot |657 |461 |111|
| ke_tot| 40 |50 |32|
|startlat_tot| -17.0| -5.0| -24.09|
|startlon_tot| -12.5| -5.0| -25.13|
|pollat| 40.0 |40.0| 39.25|
|pollon| -170.0| -170.0| -162.00|
#### 基本控制变量 RUNCTL

在该组中，设置了基本控制变量。 其中包括用于指定模拟日期和范围的变量，以及用于域分解的处理器数量。
这些变量与INT2LM组**&CONTRL**中的相应变量具有相同的名称和含义，在此不再赘述。
其他变量用于指定时间步长，用于域分解的边界线数量以及在COSMO模型运行期间应使用哪些组件

&RUNCTL

| Variable    | COSMO_EU | COSMO_DE | for RCM-Mode | function           |
| ----------- | -------- | -------- | ------------ | ------------------ |
| dt          | 66.0     | 25.0     | 240.0        | 时间步长           |
| nboundlines | 3        | 3        | 3            | 边界线的个数       |
| lphys       | .TRUE.   | .TRUE.   | .TRUE.       | 打开参数化方案     |
| luseobs     | .TRUE.   | .TRUE.   | .FALSE.      | 与观测过程一起运行 |
| ldignos     | .TRUE.   | .TRUE.   | .TRUE.       | 生成诊断文件       |
| luse_rttov  | .TRUE.   | .TRUE.   | .FALSE.      | 计算模拟卫星图像   |
| lartif_data | .FALSE.  | .FALSE.  | .FALSE.      | 运行理想化案例开关 |

#### 动力学设置 DYNCTL

在DYNCTL组中，必须选择Leapfrog动力核心（3时间级别方案）或Runge-Kutta动力核心（2时间级别方案）。 这可以通过指定变量l2tls = .FALSE来完成。 对于Leapfrog或= .TRUE。 对于Runge-Kutta。 对于这两种方案，都可以指定其他参数。
在版本4.24中，针对Runge-Kutta方案引入了新的快速波求解器，其具有以下功能：

- 提高垂直导数和平均值的准确性。
- 以强守恒形式使用散度算子。
- 人工发散阻尼的各向同性处理。

这些特性现在可以在坡度更大的山区地形中实现更稳定的整合。 通过名称列表开关itype_fast_waves = 1（旧求解器）或= 2（新求解器）来选择不同的快速波求解器。
不再建议使用Leapfrog动态核心。 现在，大多数COSMO应用程序已移植到Runge-Kutta动态内核。  COSMO_EU（分辨率约为7公里）和COSMO_DE（分辨率约为2.8公里）之间存在一些差异。
下表列出了不同应用程序的一些值。

&DYNCTL

| Variable | COSMO_EU | COSMO_DE | for RCM-Mode | function |
| -------- | -------- | -------- | ------------ | -------- |
| l2tls |.TRUE.| .TRUE.| .TRUE.| 选择2-tl或3-tl方案    |
| lcond| .TRUE.| .TRUE. |.TRUE. |云水凝结|
|rlwidth| 85000.0| 50000.0| 500000.0| 松弛层的宽度（*） |
|y_scalar_advect |’BOTT2_STRANG’ |’BOTT2_STRANG’ |’BOTT2_STRANG’|Bott advection|
|irunge_kutta| 1 | 1 |  1 | 选择Runge-Kutta方案 |
| irk_order | 3 | 3 | 3 | Runge-Kutta方案的阶数 |
| iadv_order |  3 | 5  | 5 | 对流方案的顺序 |
|itype_fast_waves| 2| 2| 1| 快速波解算器类型 |
|ldyn_bbc| .FALSE.| .FALSE.| .FALSE.| 动力底部边界          |
|lspubc | .TRUE.  | .TRUE. |  .TRUE. | 阻尼海绵层 |
|nrdtau  | 5 | 5  | 10 | 模型顶部的阻尼 |
| itype_hdiff | 2 | 2 | 2 | 水平扩散的类型 |
|hd_corr_u_in | 0.25 | 0.1 | - |风的扩散因子（内部域） |
|hd_corr_u_bd  | 0.25 | 0.75 | - |风的扩散因子（边界带） |
| hd_corr_t_in | 0.0 | 0.0 | - |温度的扩散因子（内部域） |
| hd_corr_t_bd | 0.0 | 0.75 | - |温度的扩散因子（边界带） |
| hd_corr_p_in | 0.0 | 0.0  | - |压力的扩散因子（内部域） |
| hd_corr_p_bd | 0.0 | 0.75 | - |压力的扩散因子（边界区域） |
| hd_corr_trcr_in | 0.0 | 0.0 | - |示踪剂的扩散因子（内部域）|
|hd_corr_trcr_bd  | 0.0  | 0.0  | - | 示踪剂的扩散因子（边界带） |

（\*）松弛层的宽度应延伸约15个网格点。 因此，分辨率应为米* 15。
对于水平扩散，可以为边界和域的内部区域指定特殊的扩散因子。 可以为风速，压力，温度和示踪剂成分指定它们。

#### 设置物理过程 PHYSICS

在PHYCTL组中，可以打开或关闭不同的参数。 大多数参数设置软件包都具有用于特殊设置的附加参数。 在＆PHYCTL中，COSMO_EU和COSMO_DE之间最重要的区别在于对流方案的设置。
虽然COSMO_EU使用Tiedtke对流（itype_conv = 0），但COSMO_DE使用浅对流方案（itype_conv = 3）。

&PHYCTL、

| Variable | COSMO_EU | COSMO_DE | for RCM-Mode | function |
| -------- | -------- | -------- | ------------ | ----- |
| lgsp| .TRUE. |  .TRUE.  | .TRUE. | 打开微物理过程 |
| itype_gscp | 3 | 4 | 3 | 选择特定的方案 |
| lrad | .TRUE.  |  .TRUE. |  .TRUE. | 打开辐射过程 |
| ltur | .TRUE.  | .TRUE.  | .TRUE. | 打开湍流过程 |
| lconv |  .TRUE. |  .TRUE.  | .TRUE.  | 打开对流过程 |
| itype_conv|  0 | 3 | 0 | 选取特定对流过程方案 |
| nincconv | 4 | 10 | 4 | 调用对流过程的时间间隔 |
|lsoil |  .TRUE. |  .TRUE.  | .TRUE.  | 使用土壤模型 |
| ke_soil |  7 | 7 |  9 | 选取土壤模型的特殊方案 |
| lsso |  .TRUE. | .FALSE. |  .TRUE. | 使用SSO方案 |

#### 设置 I/O &IOCTL &GRIBIN &GRIBOUT

有3个“名称列表”组可为“输入和输出”指定各种设置，例如选择“ Grib”或“ NetCDF”作为数据格式。 在IOCTL中，指定了对所有I / O有效的基本设置。
还请注意特殊的名称列表变量lbdclim，通过该变量激活COSMO-Model的气候模式。

&IOCTL 

| Variable | COSMO_EU | COSMO_DE | for RCM-Mode | function |
| -------- | -------- | -------- | ------------ | -------- |
| lbdclim | .FALSE. |  .FALSE. |  .TRUE.  | 运行气候模式的开关 |
| yform_read |  ’grb1’ |  ’grb1’ |  ’ncdf’ | 选择数据格式 |
| yform_write |  ’grb1’ |  ’grb1’ |  ’ncdf’| 选择数据格式 |
| ngribout | 1 | 3 | 3/7 | 设置输出文件夹数目 |

请注意，也可以指定“ apix”（用于使用ECMWF grib_api读取GRIB数据）和“ api2”（用于使用grib_api写入GRIB2数据）来选择数据格式。
以下组的设置取决于特殊的工作，在这里无需给出不同的规格。

下面是一个GRIBIN的示例：

```
&GRIBIN
ydirini=’/gtmp/uschaett/gme2lm/’, directory of the initial fields 
ydirbd=’/gtmp/uschaett/gme2lm/’, directory of the boundary fields 
hincbound=1.0, increment for lateral boundaries 
lchkini=.TRUE., lchkbd =.TRUE., additional control output 
lana_qi=.TRUE., llb_qi=.TRUE., cloud ice is provided for analyses and lateral boundaries
```

下面是一个GRIBOUT的示例，注意，需要根据ngribout的数目来写gribout的数目，因为此选项设置了需要输出到多少个文件夹（所给的sp包示例是三个文件夹）

```
&GRIBOUT hcomb=0.0,48.0,1.0, start, stop and incr. of output 
lcheck=.TRUE.,
yvarml=’default’,
yvarpl=’default’,
yvarzl=’default’,
lwrite_const=.TRUE.,
ydir=’/gtmp/uschaett/lm_test/with_out/’, directory of the result data
```

#### 设置诊断量计算 DIACTL

COSMO模型可以产生一些诊断输出，以快速监控预测。可以在DIACTL中指定生成的输出。下面是一个示例

```
&DIACTL
n0meanval=0, nincmeanval=1,
lgplong=.TRUE., lgpshort=.FALSE., lgpspec=.FALSE., n0gp=0, hincgp=1.0,
igp_tot = 3, 13, 25, 46,
jgp_tot = 31, 29, 96, 12,
stationlist_tot= 0, 0, 46.700, 6.200, ’RODGAU’,
0, 0, 50.050, 8.600, ’Frankfurt-Flughafen’,
53, 68, 0.0 , 0.0 , ’My-Own-Station’,
```



