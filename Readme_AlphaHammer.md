本说明文件默认位置在~/room下，会经常更新，用户可随时执行以下命令来更新

```
cd ~/room

./update_room.py
```



## 一、AlphaHammer是什么

AlphaHammer是依赖于Linux命令行环境的一组程序和数据的统称，它支持从回测研究到实盘交易；作为研究使用，用户可以使用历史数据进行回测，需要做的只是书写策略的核心逻辑和使用xml配置一些回测参数，便可以在较快的速度内得到回测结果；其他常见的使用场景包括下载数据、统计分析、实盘发单等，系统都提供了相应的工具支持。

## 二、如何开始

### 1.服务器配置

目前服务器地址为~~10.24.206.57，用户需要通知管理员开通账号，但链接服务器需要下载专门的vpn，不是很方便，*并且性能拉跨*，计划换一个*好一点的*服务器~~180.188.18.71，ssh端口为27921（注意不是默认的22），使用前需要通知管理员开通账号。

登陆服务器可以使用mobaxterm，xshell等工具。以mobaxterm为例，在下图的host username port分别填入ip地址，账号名和端口即可。

![image1](image1.png)

### 2.开始配置

美是第一生产力，程序也是这样，人会为了美不惜余力的完善改进，linux原始的命令行环境（bash）非常丑，因此推荐用户使用系统提供的zsh（oh-my-zsh）

执行以下命令：

`mkdir ~/.local/bin -p`

`cp /opt/hammer-stable/configs/zsh_config/antigen.zsh ~/.local/bin;`

`cp /opt/hammer-stable/configs/zsh_config/.antigen ~ -rf`

`cp /opt/hammer-stable/configs/zsh_config/.zsh* ~`

因为整体数据包都是从管理员用户直接拷贝过来的，所以要修改antigen的init文件，把管理员的用户名用自己的用户名替换：

`vim ~/.antigen/init.zsh`

在vim中进行全文搜索替换：

`:%s/renfei/usr/g`

其中usr用自己的用户名替换，保存退出：

`:wq`

即可，最后执行zsh启动oh-my-zsh：

`cd ~; zsh`

执行zsh后在.zshrc和.zshrc.customize上的配置才会生效，所以养成习惯进入系统后先zsh

除此之外系统还提供了便利化的vim配置和tmux配置，推荐拷贝：

`cp /opt/hammer-stable/configs/vim_config/vim-config ~ -rf`

`cp /opt/hammer-stable/configs/vim_config/.viminfo ~ -rf`

`cp /opt/hammer-stable/configs/tmux_config/.tmux* ~ -rf`

`cd`

`ln -s vim-config .vim`

`ln -s vim-config/vimrc .vimrc`

`ln -s vim-config/vimrc.bundles .vimrc.bundles`

其中~/.vimrc里写了一些快捷键配置（比如F5），可根据自己需要更改或延续使用

系统配置方面，管理员需要创建python软连接：

`cd /usr/bin`

`ln -s /usr/bin/python3.8 python`

### 3.环境变量配置

#### a.crontab配置

第一次输入crontab -e，打开定时任务配置时系统会让你选择编辑器，一般选择2（vim），如果选错了，可以输入以下命令重新选择：

`select-editor`

管理员需要执行以下命令配置定时任务环境：
`sudo cp /opt/hammer-stable/configs/environment /etc`

`sudo cp /opt/hammer-stable/configs/profile /etc`

用户则执行：

`crontab -e`

开始前在最下面加一行

`SHELL=/opt/hammer-stable/util/cron.bash`

除此之外，可能还需要获取该脚本的执行权限，运行

`ls /opt/hammer-stable/util/cron.bash -al`

如果显示的权限码大于等于如下则无需其他操作：

`-rwxr-xr-x`

如否则需要手动增加执行权限，运行

`chmod u+x /opt/hammer-stable/util/cron.bash`

这一步可能会被permission denied，需要通知管理员

#### b.PATH和PYTHONPATH

这一步的配置已经写在了~/.zshrc.customize，在输入zsh后会加载该文件的配置；如果遇到问题可执行:

`echo $PATH`

查看当前的环境变量配置与~/.zshrc.customize中是否一致

### 4.样例代码

在开始之前可以拷贝系统提供了的基本样例代码：

`cd ~; mkdir room; cd room`

`cp /opt/hammer-stable/configs/update_room.py .`

`chmod 755 update_room.py; python update_room.py`

以上命令会拷贝xmlsample和pysample到~/room下

### 5.服务器之间的免密登录

服务器之间传输文件每次都要输入密码，非常麻烦，免密登录的原理是：假设想从197免密登录57（即向57rsync或者scp时不需要输密码），那么需要在197生成一份密钥，并写在57的免密登录列表里，于是我们有如下方法：

`(197) cd ~`

`(197) ssh-keygen` 这一步之后一直回车使用默认选项即可

`(197) rsync .ssh/id_rsa.pub {your user name}@{57server address}:~/` 这一步要输57服务器的密码，之后去57操作

`(57) cd ~/.ssh`

`(57) cat ../id_rsa.pub | tee -a authorized_keys`

`(57) rm ../id_rsa.pub`

之后重新登陆一下197，就可以尝试免密登录57了，反过来也同理

## 三、从一个xml开始

### 1、xml基础配置

以下代码来自xmlsample/test_alpha.xml，是一个基本的xml配置，我们分别介绍其中的元素：

```xml
<context update_data="false" cache_path="/data/cache" lib_path="/opt/hammer-stable/lib" log="info" >
    <universe id="future" module="universe_simple" beg_date="2021" end_date="202107" >
        <dataloader auto_load="true" />

		<portfolio trade_price="settle" refer_price="settle" pnl_path="./pnl" >

			<alpha id="af1" module="alpha_formula" expr="func_ma(return,20)" universe="LIQPC" dump_file="true" >
            	<operation module="alphaop_powrank" exp="1" industry="f2" />
        	</alpha>

    	</portfolio>

	</universe>
</context>
```

### 2. context节点

定义了一些全局变量

| 属性               | 含义                                                         | 示例                   |
| ------------------ | ------------------------------------------------------------ | ---------------------- |
| update_data        | 是否更新公共数据缓存，回测时可设为false，只有当需要更新公共数据时才设为true；要注意，即使update_data被设为了false，仍会根据需要更新自建数据缓存，详见后面数据章节 | false                  |
| cache_path         | 公共数据缓存读写位置，一般使用系统公共缓存写为/data/cache即可 | /data/cache            |
| private_cache_path | 自建数据缓存读写位置，默认值与cache_path相同；当有自建数据缓存需求时使用，data与alpha节点的缓存设置依赖于该参数，详见后面数据章节 | {cache_path}           |
| lib_path           | 各个节点中module的位置，一般写为/opt/hammer-stable/lib即可   | /opt/hammer-stable/lib |
| log                | 屏幕打印log的级别，有debug, info, warning, error四个级别，log数量依次减少 | info                   |

### 3. universe节点

定义了全局股票池的细节，所谓全局股票池即一个定义了包含所有回测交易日和股票的二维矩阵，矩阵大小为di_num * ii_num，其中di_num为交易日天数，ii_num为股票个数

| 属性        | 含义                                                         | 示例                           |
| ----------- | ------------------------------------------------------------ | ------------------------------ |
| id          | 股票池名称，最终生成缓存的位置为{cache_path}/{id}            | stock、future                  |
| module      | 一般使用universe_simple，可以通过继承基类来自定义建立股票池的逻辑，详见建立股票池章节 | universe_simple                |
| beg_date    | 回测起始日，支持2009，200912，today，yesterday等写法         | 2009                           |
| end_date    | 回测终止日，支持2009，200912，today，yesterday等写法         | today                          |
| map_path    | 适用于期货场景，定义合约映射                                 |                                |
| map_info    | 格式“code:1;map_code:2”                                      | code:1;map_code:2              |
| tradingdays | 交易日列表                                                   | /data/stock/cn_tradingdays.txt |
| min_ii_num  | 预先定义一个ii_num，代码不足的用‘’填充                       | 5000                           |
| is_intra    | 在universe_from_parent使用，用于建立日内股票池               | true                           |
| interval    | 日内股票池的分钟频率                                         | 15                             |
| times       | 日内股票池的时间范围，多个时间段用分号隔开，格式为HHMMSS     | 93000-113000;130000-150000     |

universe节点下可以加入若干个instrument子节点，用于添加股票池的代码，如非自建股票池可忽略：

| 属性     | 含义                                                         | 示例                |
| -------- | ------------------------------------------------------------ | ------------------- |
| path     | 文件路径，程序将读入该路径文件的所有code，可以用{YYYYMMDD}来通配日期 |                     |
| code_col | 代码列，告诉程序要读文件的哪一列                             | 1                   |
| filter   | 可以筛选需要的代码，如需要正则表达式，需要通知管理员来支持   | 000001.SH;000050.SH |

### 4. dataloader节点

负责沟通数据缓存，程序与数据的交互逻辑：从csv、txt等文件中读出数据，以二进制文件的形式写入硬盘，在使用的时候直接读缓存好的二进制文件到内存，以作后续处理计算：

| 属性      | 含义                                                         | 示例                                 |
| --------- | ------------------------------------------------------------ | ------------------------------------ |
| auto_load | 指明读入缓存文件的路径，多个路径用空格隔开；可以只写一个true，等价于“{cache_path}/{uid};{private_cache_path}/{uid}“，即读入公共缓存和自建缓存 | true /data/cache/stock ./cache/stock |

如果涉及自建数据缓存，需要在dataloader下逐一加入data节点，将在后续数据章节说明，基本参数如下

| 属性             | 含义                                                         | 示例 |
| ---------------- | ------------------------------------------------------------ | ---- |
| increment_update | 在股票池增量更新后数据是否增量更新；不是所有的系统数据模块都支持该用法 | true |
| cache_path       | 缓存位置，默认为public，即缓存在cache_path；可选private，此时将缓存在private_cache_path |      |

### 5. portfolio节点

定义投资组合的一些基本信息

| 属性        | 含义                                            | 示例  |
| ----------- | ----------------------------------------------- | ----- |
| trade_price | 交易价格，close，vwap（股票）或settle（期货）   | close |
| refer_price | 持仓规模参考价格，close（股票）或settle（期货） | close |
| pnl_path    | pnl文件位置                                     | ./pnl |
| is_intra    | 如果是日内universe，需要设置is_intra为true      | true  |

### 6. alpha节点

一个alpha节点即定义了一个因子的计算逻辑和范围

| 属性           | 含义                                                         | 示例          |
| -------------- | ------------------------------------------------------------ | ------------- |
| id             | alpha的名称                                                  | alpha1, test  |
| module         | 计算alpha使用的模块                                          | alpha_formula |
| universe       | 计算alpha的选股股票池                                        | HS300，LIQPC  |
| delay          | 计算当天的信号不能使用**未来信息**，因此使用的数据要有至少1个bar的delay，若载入事先生成的csv则可根据需求调整 | ·             |
| alpha_is_money | 默认为false，即在每个切片计算完alpha和所有op操作之后会进行归一化；如果设为true，则不进行归一化 | true          |
| dump_file      | 是否将alpha缓存为二进制文件到硬盘                            | true          |
| interval       | alpha的计算频率，默认为每日计算，若设为“weekly”或“monthly”则将在每周/每月的最后一天计算 | weekly        |
| expr           | module设为alpha_formula时，可以通过公式表达式计算alpha，详见后续公式表达式章节 |               |
| cache_path     | 缓存位置，默认为private，可选public，注意与data节点默认值的不同 |               |

### 7. operation节点

operation是一类特殊的节点，它是alpha的子节点，用于处理alpha得到的原始信号向量，计算的逻辑是：在每个切片由alpha模块的run_daily（或者run_intra_daily，对应于日内模型）给出alpha向量后，会通过alphaop的run_daily/run_intra_daily进行处理，（如果有多个op则从上到下依次处理），最后给出的向量用于pnl计算。不同operation模块需设置不同属性，目前支持的operation如下

| 模块            | 含义                                                 | 参数示例                                   |
| --------------- | ---------------------------------------------------- | ------------------------------------------ |
| alphaop_decay   | 时序decay（加权移动平均）                            | days=“5”                                   |
| alphaop_ma      | 时序移动平均                                         | days="5"                                   |
| alphop_ema      | 时序ema                                              | days="10"                                  |
| alphaop_powrank | 截面排序                                             | exp="1" industry=“wind1” keep_sign=“false” |
| alphaop_neutral | 截面中性化（使alpha向量均值为零）                    |                                            |
| alphaop_scale   | 截面归一化（使alpha绝对值之和为1）                   |                                            |
| alphaop_vol     |                                                      |                                            |
| alphaop_slice   | 分层，通过level指定层级数量，通过pos指定多空哪些层级 | level="5",pos="-1,5"                       |

### 8.alphameta节点

将计算好的一系列alpha进行组合

| 模块                | 含义                                                 | 示例              |
| ------------------- | ---------------------------------------------------- | ----------------- |
| id                  | meta因子的名称                                       | alpha_meta        |
| module              | 计算meta时使用的模块，目前支持加权平均               | alphameta_simple  |
| meta                | 需要meta的alpha及其权重                              | af1:1;af2:1;af3:2 |
| delay               | 若meta前的alpha已计算过delay，则此时注意将delay设为0 | 0                 |
| dump_file，interval | 与alpha节点含义相同                                  |                   |

### 9.risk_manager节点

risk_manager实际上是另一种operation，其设计初衷是与计算alpha和op隔离开的一个风控模块，以daily模型为例，默认daily因子在用户编写的op处理之后都会再加一次alphaop_scale，以调整总规模到1，而risk_manager节点的设计是放在这些op计算之后的，因此允许用户自由调节如总规模、风险敞口等。

以alphaop_hedge模块为例，它可以模拟指数对冲，目前只用指数行情计算，并非使用股指期货对冲

| 模块       | 含义                       | 示例          |
| ---------- | -------------------------- | ------------- |
| module     | 计算risk_manager使用的模块 | alphaop_hedge |
| hedge_code | 对冲使用的指数代码         | 000300.SH     |
| hedge_name | 对冲使用的指数名称         | HS300         |
| scale      | 是否重新调整总规模到1      | 默认为true    |

其他alphaop也可以写在risk_manager节点中，在计算完所有op之后会依次计算risk_manager。

## 四、如何进行回测及绩效统计

### 1. 回测

系统提供auto.py来加载xml中的各个模块，得到最后的回测结果，运行：

`auto.py {xml位置}`

开始计算后即可看到中间过程，最后的pnl以文本文件的形式保存在pnl文件夹（portfolio节点定义）下。

### 2.绩效统计

系统提供summ.py来进行绩效统计，运行：

`summ.py -h`

即可看到该工具的说明，简单来说，基本的使用方法是：

`summ.py {pnl文件} --begdate=*** --enddate=***`

还有如下参数可供选择

| 参数      | 说明                                                         | 示例 |
| --------- | ------------------------------------------------------------ | ---- |
| compound  | 计算复利                                                     |      |
| book_size | 指定本金大小，默认情况下booksize=1，这时如果所有的权重绝对值加起来大于1就代表有杠杆，可以通过设置booksize来主动避免杠杆 | 1，2 |
| c, corr   | 在提供多个pnl文件时，可以计算相关之间的相关性                |      |
| P, plot   | 画净值图                                                     |      |
| csv_out   | 同时输出为csv                                                |      |

### 3.命令行工具

afstats.py可以用来做命令行回测及因子绩效统计，使用方法如下

`afstats.py --cache_path=stock -e "ma(return,120)" -o "rank;neu" -u TOP90P`

参数说明

| 参数         | 样例            | 说明                                                         |
| ------------ | --------------- | ------------------------------------------------------------ |
| --cache_path | stock, future   | 使用股票或期货缓存                                           |
| -e, --expr   | ma(return, 120) | 如果expr以负号开头，需要在负号之后加空格，如- ma(return,120) |
| -1 ~ -6      |                 | 替换公式中的占位符，比如-e “ma(return,%1)”  -1 10,20,30将会生成对应三个表达式，又比如-e “ma(%1, %2)” -1 close,open -2 10,20,30将会生成2 * 3 = 6个不同表达式，至多支持6个不同占位符 |
| -o           | “rank;neu”      | 用分号分隔，依次计算，比如-o “rank;neu”                      |
| -u           | TOP90P          |                                                              |
| --backtest   |                 |                                                              |
| --dump_id    |                 | 指定缓存的因子id，如果不加设置将会自动设置                   |

## 五、如何检查或单独使用缓存数据

### 1.命令行工具

由于系统数据的交互方式是二进制文件，不具有可读性，因此提供了stats.py来检查或单独使用系统缓存的二进制数据，运行：

`stats.py -h`

来查看使用说明，简单来说，你需要指定读取缓存的路径，读取的数据是什么，读取的代码以及时间范围

| 参数         | 说明                                                         | 示例                    |
| ------------ | ------------------------------------------------------------ | ----------------------- |
| e, expr      | 检查的数据，以分号隔开，支持**公式表达式**                   | close, func_ma(close,5) |
| u, universe  | 选股股票池；除了表达式的计算是以其为范围的，结果展示也只包括股票池内的标的 | HS300，LIQPC            |
| p, precision | 查看数据的精度，默认4位小数                                  |                         |
| P, plot      | 画图，未实现                                                 |                         |
| c, code      | 指定品种代码，用分号隔开，如果只加-c则输出截面均值           |                         |

如果想在日频股票池检查日内数据，可以这样做：

`stats.py -e "intra_1min_close; func_intra_offset(intra_1min_close, 1):intra"`

其中表达式的:intra后缀表示计算日内公式表达式，但在日内股票池则不需要这样做，默认全部计算日内公式。

stats.py支持占位符扩展，举例来说，以下表达式：

`stats.py -e "%1 - ma(%1, %2)" -1 close,open -2 5:10`

将展开为2*6=12个不同表达式。使用者可以最多使用%1到%6共六个占位符，分别用-1~-6的参数来表示。

### 2.脚本工具

在python脚本中，可以用以下方法来获得系统数据：

```python
from base.dataloader import dataloader
dl = dataloader.Dataloader()
dl.init("/data/cache/future")
# 通过系统股票池缓存路径载入缓存
print(dl.dates, dl.codes)

# 读取数据或计算表达式
data1 = dl.get_data("close")
data2 = dl.eval_expr("func_ma(close,5) - func_ma(close,60)")

# 获得具体合约某一日的数据
ii = dl.get_ii("RB.SHF")
di = dl.get_di("20210110")
print(data1[di, ii])
```

dl对象的init方法支持从多个缓存路径初始化，此时将依次导入各个路径下的所有数据缓存，但矩阵结构将以第一个路径的股票池为准，一般使用场景是除了系统缓存外，还想导入保存在私有缓存的数据，因为这些数据是依赖于系统缓存生成的，因此可以一并导入（不必担心缓存日期比系统缓存短，程序会自动补空值）。

另一种情形也涉及到多个缓存路径，比如想同时使用A股股票池和期货股票池的数据，这时就不能用上面的方法了，需要生成两个dl对象分别从两个公共缓存init初始化，因为两者的矩阵结构就不同，不能一并导入。

以上说明同样适用于命令行工具stats.py，它有一个参数--cache_path就是指定如何初始化dl对象的，

## 六、如何建立股票池

xml中的universe节点，定义了整体回测的系统股票池，以下是一个示例配置：

## 七、如何使用data节点

### 1.如何生成数据缓存

系统基本的数据生产流程是从1）数据库原始数据，到2）本地csv或其他格式的文本文件，到3）标准化格式的二进制缓存文件，我们先来说从1到2

这部分相对灵活，因为不同的数据库有不同的下载方式，但一定要找到一种自动化的每日下载方式（为了支持将来可能的实盘每日更新），系统提供一个脚本data.py，用户可以将自己的下载配置写在~/room/data_source/private_data.py中，以下示例中配置了两个示例数据

```python
datas = {
        "future_positions": {
            "database": "windfile",
            "type": "files",
            "sql": """
                select S_INFO_WINDCODE, FS_INFO_MEMBERNAME, FS_INFO_TYPE, FS_INFO_POSITIONSNUM, FS_INFO_RANK, S_OI_POSITIONSNUMC
                from CCOMMODITYFUTURESPOSITIONS
                where TRADE_DT = '{YYYYMMDD}'
                order by S_INFO_WINDCODE
            """,
            "data_path": "/data/future/future_positions/future_positions_${YYYY}${MM}${DD}.csv",
            "has_header": True,
            "date_format": "${YYYY}${MM}${DD}"
        },

        "future_position_stats": {
            "type": "run",
            "script": "python /data/future/py/get_position_stats.py",
            "check": "/data/future/position_stats/position_stats_${YYYY}${MM}${DD}.csv",
        },
}
```

第一个数据名为future_positions，其处理方法是用sql去wind数据库逐日下载，第二个数据名为future_position_stats，其处理方法是逐日运行一个python script；配置好之后在命令行运行data.py future_positions future_position_stats -s yyyymmdd -e yyyymmdd，就会从起始日到终止日逐日下载这两个数据，具体使用方法请data.py -h.

从2到3的步骤相对更为标准化，以xmlsample/test_data.xml为例，它定义了一个名为cal的data节点，module调用的程序是pysample/data_calendar.py，在初级阶段，使用者只需要实现init和reload_data两个函数，其中init是初始化函数，可以自定义需要的属性，比如我们在data_calendar里定义了需要缓存的数据矩阵，注意为了使用方便，通常数据矩阵的结构应该是(self.di_num, )[一维]，或者(self.di_num, self.ii_num)[二维]，或者(self.di_num, self.ti_num, self.ii_num)[三维]；reload_data中实现如何填充数据矩阵的逻辑，写好矩阵之后，调用系统提供的self.register_data(name, data)，即可将该矩阵缓存到文件系统，并且可供后续程序使用；比如在test_data.xml中，我们注册了year、month、day、weekday等数据，那么接下来在下面的alpha节点中就可以用这些数据进行因子计算了。

### 2.如何做实盘数据检查

系统提供data_test模块来完成数据检查功能，见下例：

```xml
<universe id="future" module="universe_simple" beg_date="2012" end_date="today" >
    <dataloader auto_load="true" >
        <data id="test_future" module="data_test" date="yesterday" send_to="522526392@qq.com" universe="LIQPC" >
            <test expr="close_prev" test="normal" tolerance="5" date="today" />
            <test expr="close" test="normal" tolerance="5" />
            <test expr="return" test="valid" tolerance="5" />
            <test expr="value_ss" test="positive" tolerance="3" />
            <test expr="volume_ss" test="positive" tolerance="3" />
        </data>
    </dataloader>

    <portfolio trade_price="settle" refer_price="settle" pnl_path="./pnl" >
    </portfolio>
</universe>
```

data节点的date表示希望检查的数据日，universe表示希望检查的范围，每个test子节点表示一个检查项，tolerance表示最大容许的个数，在test子节点可以重新设置date，会覆盖data父节点的date；如果一个data_test节点有任意一个test没有通过，程序就会终止，并且向send_to配置的邮箱发送提醒邮件。关于邮箱配置，见Q&A。

### 3.如何灵活使用二维、三维数据

系统目前支持二维、三维股票池进行回测，同时支持在不同的股票池灵活使用不同维度的数据，下面总结了不同场景下使用不同数据的best practice

#### a. 二维股票池使用二维数据

##### (1).data节点生成二维数据：data_fromfile

```xml
<data id="data_id" module="data_fromfile" increment_update="true" check_valid="10" data_path="data_path" data_info="code:1;data1:2;data2:3" />
```

注：要求原始数据以csv格式每天一个文件存放在data_path路径中，文件名能用YYYYMMDD辨识，数据格式体现在data_info中，必须有code列

##### (2).data节点生成二维数据：data_formula

```xml
 <data id="data_id" module="data_formula" expr="data1:expr1;data2:expr2" universe="universe_name" />
```

注：universe参数是expr中公式计算的应用范围，比如expr=“zzz:zs(return)”，如果不加universe，会生成一个名为zzz的数据，计算的是return在默认全市场的zscore，如果加universe，比如universe=“ZZ800”，则计算的是return在ZZ800中的zscore，且也只在ZZ800上有效

##### (3).alpha节点使用二维数据：alpha_formula

```xml
<alpha id="alpha_id" module="alpha_formula" expr="expr" universe="universe_name" />
```

##### (4).alpha节点使用二维数据：alpha_python

```xml
<alpha id="alpha_id" module="your_python_file.py" universe="universe_name" />
```

#### b.二维股票池使用三维数据

##### (1).data节点生成三维数据：data_intra_fromfile

```xml
 <data id="data_id" module="data_intra_fromfile" increment_update="true" check_valid="10" data_path="data_path" frequency_in_sec="60" times="90000-101500;103000-113000;133000-150000" bar_type="bob" data_info="code:0;time:1;data1:2;data2:3" />
```

注：二股股票池也可以导入三维分钟级别数据，类似于data_fromfile，额外需要的信息如上，其中bar_type可以取值bob (begin of bar)和eod (end of bar)，区别在于，当times写如90000-100000时，bob会采样90000,90100,...,95900这60个分钟数据，而eob会采样90100,90200,...100000这60个分钟数据；csv文件中除了code列之外，还需要提供time列

##### (2).data节点生成三维数据：data_formula

```xml
<data id="alpha_id" module="data_formula" expr="data:expr:intra" universe="universe_name" />
```

注：是的你没看错，我也没有写错，alpha_formula也可以生成三维数据，方法就是在expr后加上:intra后缀，系统会自动解析所有的用冒号分隔的元组，并判断有intra后缀的生成为三维数据，这时如果你仍然提供了一个二维公式，那么将会以repeat的方式自动扩充为三维

##### (3).data节点使用三维数据生成二维数据：data_formula

```xml
<data id="alpha_id" module="data_formula" expr="data:to2d(intra_expr,ti)" universe="universe_name" />
```

注：假设你有了一个三维数据，并且想用它每天固定某个ti的信息作为当日信息，从而导出一个二维数据，那么可以用to2d这个函数，如上所示，如果不写ti，默认取最后一个ti

##### (4).alpha节点使用三维数据计算pnl

```xml
<alpha id="alpha_id" module="alpha_formula" expr="to2d(intra_expr,ti)" universe="universe_name" />
```

注：原理与(3)一样

#### c.三维股票池使用三维数据

##### (1).data节点生成三维数据：data_intra_fromfile

```xml
 <data id="data_id" module="data_intra_fromfile" increment_update="true" check_valid="10" one_by_one="false" data_path="data_path" data_info="code:0,date:1,time:2,data1:3;data2:4" intra_expr="intra_data:intra_expr" />
```

注：与二维股票池生成三维数据不同，这里不需要加times和freq参数了，因为将以三位股票池的信息为准，除此之外，这里还可以用intra_expr直接利用data_info导入的数据生成日内数据；one_by_one如果在内存不够的情况下要设为true

##### (2).data节点生成三维数据：data_from_intv

```xml
 <data id="data_id" module="data_from_intv" to="n" data_info="to_data1:data1;to_data2:data2" />
```

注：目前只有我有用到，将1分钟的日内数据聚合成任意分钟频率，使用前咨询我

##### (3).alpha节点使用三维数据

只推荐用alpha_python，自己写py文件的方式使用，公式表达式换手率不好控制

#### d.三维股票池使用二维数据

##### (1).data节点生成二维数据

与二维股票池没有什么区别

##### (2).alpha节点使用二维数据：alpha_formula

```xml
<alpha id="alpha_id" module="alpha_formula" expr="expr" universe="universe_name" />
```

注：这时会默认在每天第一个bar调整到二维表达式当日的目标仓位

## 九、关于公式表达式

公式表达式提供了一个便捷的矩阵处理方法，在dataloader中的data节点和alpha节点，以及stats.py中均可以使用。

用户可以自由使用四则运算、逻辑运算，如close+high, open > close等；需要注意的是，**在xml中**由于一些特殊符号（如尖括号）无法被解析，需要用以下字符串替换，而在命令行中则不需要这样做：

| 运算符 | 代替符号 |
| ------ | -------- |
| <      | &lt；    |
| >      | &gt；    |
| &      | &amp；   |
| ‘      | &apos；  |
| “      | &quot；  |

除此之外，系统还提供一系列基本函数：

| 公式                 | 含义                            | 用法                                          | 说明                                                         |
| -------------------- | ------------------------------- | --------------------------------------------- | ------------------------------------------------------------ |
| func_ma              | 时序ma                          | func_ma(close, 20)                            | 第二个参数必须是正整数                                       |
| func_intra_ma        | 日内时序ma                      | func_intra_ma(intra_close, 20)                | 若is_reset_daily=True，则不使用跨日信息                      |
| func_decay           | 时序decay                       | func_decay(close, 20)                         | 过去n天加权平均，权重依次为n~1                               |
| func_ema             | 时序ema                         | func_ema(close, 20)                           | 第二个参数可以是正整数，也可以是0到1的实数，这时表示decay系数 |
| func_avg             | 截面均值                        | func_avg(return)                              |                                                              |
| func_ind_avg         | 行业截面均值                    | func_avg(return, wind1)                       | 表示在每个行业上分别取均值                                   |
| func_sum             | 时序或截面求和                  | func_sum(close [, 20])                        | 如果有第二个正整数参数，表示时序求和，否则截面               |
| func_std             | 时序std                         | func_std(return, 20)                          |                                                              |
| func_zs              | 时序或截面z-score               | func_zs(return [, 20])                        | 如果有第二个正整数参数，表示时序zs，否则截面                 |
| func_ind_zs          | 行业截面z-score                 | func_ind_zs(return, wind1)                    |                                                              |
| func_rank            | 时序或截面rank                  | func_rank(return [, 20])                      |                                                              |
| func_ind_rank        | 行业界面rank                    | func_ind_rank(return, wind1)                  |                                                              |
| func_iif             | 满足condition时取a，否则取b     | func_iif(close>20, 20, close)                 | 这个例子表示取close truncate到20                             |
| func_and             | 条件and                         | func_and(close>20, volume<1e9)                |                                                              |
| func_offset          | 时序位移，相当于shift           | func_offset(close, -1)                        | 只适用于二维数据                                             |
| func_at_ii           | 取某个股票的数值                | close - func_at_ii(close, ‘000001.SH’)        |                                                              |
| func_intra_offset    | 日内时序位移                    | func_intra_offset(intra_close, -1)            | 只适用于三维数据，当is_reset_daily=True时不使用跨日信息      |
| func_ifnan           | 替换nan                         | func_ifnan(close_prev, func_offset(close,-1)) | 第二个参数可以是实数也可是表达式                             |
| func_cumprod         | 连乘                            |                                               |                                                              |
| func_median          | 中位数                          |                                               |                                                              |
| func_sign            | 取sign                          |                                               |                                                              |
| func_log             | 取log                           |                                               |                                                              |
| func_pow             | 求幂                            | func_pow(return, 2)                           |                                                              |
| func_rpos            | 求一个表达式在过去n天的相对位置 | func_rpos(return, 20)                         | = (v - min) / (max - min)                                    |
| func_min             | 时序或截面min                   |                                               |                                                              |
| func_max             | 时序或截面max                   |                                               |                                                              |
| func_abs             | 取绝对值                        |                                               |                                                              |
| func_corr            | 取两个表达式的滚动相关系数      | func_corr(close, volume, 20)                  |                                                              |
| func_ffill           | 向前填充nan                     | func_ffill(roe [, 3])                         | 如果有第二个正整数参数，表示最多向前填充n天                  |
| func_intra_ffill     | 向前填充日内数据                | func_intra_ffill(intra_close, [, 3])          | 第二个参数，表示最多向前填充日内n个bar                       |
| func_multiply        | 两个表达式相乘                  | func_multiply(intra_close, adj)               | 一般情况下用*即可，但这个函数支持不同维度数据的相乘，比如intra_close和复权因子cumadj |
| func_replace_extreme | 在截面上根据中位数替换极值      | func_replace_extreme(return, 3)               | 第二个参数表示最大截取倍数                                   |
| func_dif             | 时序上做差分                    | func_dif(close, 10)                           | 第二个参数表示滞后项阶数                                     |
| func_wma             | 等价于func_decay                |                                               | 显著慢于func_decay，建议都用decay                            |
| func_sma             | 还没有测试                      |                                               |                                                              |
| func_beta            | 时序、横截面回归系数            | func_beta(a, b [, 20])                        | 求a和b的回归系数，有第三个正整数参数表示时序回归             |
| func_res             | 时序、横截面回归残差            | func_res(a, b [, 20])                         | 求a和b的回归残差，有第三个正整数参数表示时序回归             |
| func_ncbeta          | 时序、横截面回归系数（无常数）  | func_ncbeta(a, b [, 20])                      | 求a和b的回归系数，有第三个正整数参数表示时序回归             |
| func_ncres           | 时序、横截面回归残差（无常数）  | func_mcres(a, b [, 20])                       | 求a和b的回归残差，有第三个正整数参数表示时序回归             |
| func_mvbeta          | 多元回归系数                    | func_mvbeta(a, *b)                            | 求a对多个自变量b的回归系数                                   |
| func_winsorize       | 截面按百分位截取                | func_winsorize(return, .1, .1)                | 两个参数分别表示low_limit和high_limit                        |
| func_trunc           | 截尾                            | func_trunc(return, -0.1, 0.1)                 |                                                              |

nry补充的一些日内公式

|                        |      |      |      |
| ---------------------- | ---- | ---- | ---- |
| func_intra_ema         |      |      |      |
| func_intra_dif         |      |      |      |
| func_intra_avg         |      |      |      |
| func_intra_median      |      |      |      |
| func_intra_max         |      |      |      |
| func_intra_min         |      |      |      |
| func_intra_sign        |      |      |      |
| func_intra_log         |      |      |      |
| func_intra_pow         |      |      |      |
| func_intra_sqrt        |      |      |      |
| func_intra_abs         |      |      |      |
| func_intra_iif         |      |      |      |
| func_intra_and         |      |      |      |
| func_intra_or          |      |      |      |
| func_intra_decay       |      |      |      |
| func_intra_:wwinsorize |      |      |      |
| func_intra_trunc       |      |      |      |

## 十、日内alpha模型

日内alpha模型相对于日间alpha模型，最大的特点在于灵活性，具体来说，日间alpha中的interval和alpha_is_money参数不再生效，系统不再会自动为权重向量归一化，用户将自行决定每个日内时刻的仓位；目前系统模块alpha_formula同样支持日内模型，通过expr参数计算一个二维矩阵，回测时将在每天日内第一个bar调整到当日的目标权重，之后的日内其他时刻则不再换手。

一个例子是pysample/alpha_intra_test.py，配置文件是xmlsample/test_intra_alpha.xml。

日内股票池目前不支持afstats；stats的使用方式为（以期货cta15分钟股票池为例）

`stats.py --intra --cache_path=future_15m -e "intra_close" -c RB --begdate=2021 --enddate=202101 `

## 十一、可用数据

### 1.股票可用数据

A股日行情数据（包含股指）

| 数据名           | 代码          | 说明                                                         |
| ---------------- | ------------- | ------------------------------------------------------------ |
| 昨收盘价(元)     | close_prev    | 相对于close是复权的                                          |
| 开盘价(元)       | open          | 数据保留2位小数                                              |
| 最高价(元)       | high          | 数据保留2位小数                                              |
| 最低价(元)       | low           | 数据保留2位小数                                              |
| 收盘价(元)       | close         | 数据保留2位小数                                              |
| 涨跌幅           | return        | ROUND((close - close_prev) * 100 / close_prev, 2)            |
| 成交量(手)       | volume        |                                                              |
| 成交金额(万元)   | value         |                                                              |
| 复权昨收盘价(元) | adjclose_prev | 昨收盘价*复权因子                                            |
| 复权开盘价(元)   | adjopen       | 开盘价*复权因子                                              |
| 复权最高价(元)   | adjhigh       | 最高价*复权因子                                              |
| 复权最低价(元)   | adjlow        | 最低价*复权因子                                              |
| 复权收盘价(元)   | adjclose      | 收盘价*复权因子                                              |
| 复权因子         | cumadjfactor  | 初始值为1;当日复权因子=前一交易日收盘价/当日昨收盘价*前一交易日复权因子。 |
| 均价             | vwap          | 成交金额/成交量                                              |

A股日行情估值指标

| 数据名                    | 代码          | 说明                                                         |
| ------------------------- | ------------- | ------------------------------------------------------------ |
| 当日总股本                | shares        | 万股                                                         |
| 当日流通股本              | floats        | 万股;当日流通A股股本                                         |
| 当日自由流通股本          | frees         | 万股                                                         |
| 市盈率(PE,TTM)            | pe_ttm        | 倍;总市值/净利润(TTM)                                        |
| 市现率(PCF,经营现金流TTM) | ocf_ttm       |                                                              |
| 市现率(PCF,现金净流量TTM) | ncf_ttm       |                                                              |
| 市销率(PS,TTM)            | ps_ttm        |                                                              |
| 换手率                    | tvr           | %                                                            |
| 换手率(基准.自由流通股本) | free_tvr      | %                                                            |
| 营业收入(TTM)             | revenue_ttm   | 元                                                           |
| 当日总市值                | mv            | 万元;计算公式:A股股价*总股本(包含B股,H股等,除优先股)         |
| 当日流通市值              | circular_mv   | 万元                                                         |
| 当日净资产                | net_asset     | 元                                                           |
| 股价/每股派息             | price_div_dps |                                                              |
| 市净率(PB)                | pb            | 倍;总市值/净资产(LF)                                         |
| 涨跌停状态                | up_down       | 1表示涨停;0表示非涨停或跌停;-1表示跌停。                     |
| 最高最低价状态            | low_high      | 1表示是历史最高收盘价;0表示非历史最高价或最低价;-1表示是历史最低收盘价。 |

A股日收益率数据（beta和alpha）

| 数据名         | 代码         | 说明                                                         |
| -------------- | ------------ | ------------------------------------------------------------ |
| 日BETA(1年)    | beta_day_1y  | 以个股和标的指数最近1年日涨跌幅划线计算出的斜率;A股的标的指数是沪深300。 |
| 日BETA(2年)    | beta_day_1y  | 以个股和标的指数最近2年日涨跌幅划线计算出的斜率;A股的标的指数是沪深300。 |
| 日ALPHA(1年)   | alpha_day_1y | 以个股和标的指数最近1年日涨跌幅划线计算出的截距;A股的标的指数是沪深300。 |
| 日ALPHA(2年)   | alpha_day_2y | 以个股和标的指数最近2年日涨跌幅划线计算出的截距;A股的标的指数是沪深300。 |
| 周BETA(100周)  | beta_100w    | 以个股和标的指数最近100周的周涨跌幅划线计算出的斜率;A股的标的指数是沪深300。 |
| 周ALPHA(100周) | alpha_100w   | 以个股和标的指数最近100周的周涨跌幅划线计算出的截距;A股的标的指数是沪深300。 |
| 月BETA(24月)   | beta_24m     | 以个股和标的指数最近24月的月涨跌幅划线计算出的斜率;A股的标的指数是沪深300。 |
| 月BETA(60月)   | beta_60m     | 以个股和标的指数最近60月的月涨跌幅划线计算出的斜率;A股的标的指数是沪深300。 |
| 月ALPHA(24月)  | alpha_24m    | 以个股和标的指数最近24月的月涨跌幅划线计算出的截距;A股的标的指数是沪深300。 |
| 月ALPHA(60月)  | alpha_60m    | 以个股和标的指数最近60月的月涨跌幅划线计算出的截距;A股的标的指数是沪深300。 |

A股停牌数据

| 数据名           | 代码    | 说明                      |
| ---------------- | ------- | ------------------------- |
| 当天是否有过停牌 | suspend | 1表示停牌，空值表示未停牌 |

A股退市（警告）数据

| 数据名                                                       | 代码 | 说明                      |
| ------------------------------------------------------------ | ---- | ------------------------- |
| 当天是否处在：退市或特别处理(ST) 、暂停上市、退市整理、退市这4种情形当中 | stzl | 1表示处于，空值表示不处在 |

A股新股和次新股数据

| 数据名 | 代码          | 说明                                                        |
| ------ | ------------- | ----------------------------------------------------------- |
| 新股   | new_stock     | 当天是否处在，距IPO6个月以内；1表示处于，空值表示不处于     |
| 次新股 | sub_new_stock | 当天是否处在，距IPO6个月-1年以内；1表示处于，空值表示不处于 |

A股交易异动

| 数据名                                                       | 代码      | 说明                            |
| ------------------------------------------------------------ | --------- | ------------------------------- |
| 当天是否满足：连续三个交易日内涨跌幅偏离值累计达20%          | dev_20    | 1表示满足，空值表示不满足，下同 |
| 当天是否满足：连续三个交易日内日均换手率与前五个交易日的日均换手率的比值达到30倍且换手率累计达20%的证券 | dev_30_20 |                                 |
| 当天是否满足：连续3个交易日内收盘价格涨跌幅偏离值累计达30%   | dev_30    |                                 |

朝阳永续个股一致预期

| 数据名前缀 | 含义                                                         |
| :--------- | :----------------------------------------------------------- |
| fy0        | 基准年；每年5月1日进行调整，5月1日之前基准年为前年，5月1日后基准年为去年。例如：2017年4月30日，由于上市公司年报还未全部发布，则基准年为2015年；2017年5月1日起，基准年变为2016年。 |
| fy1        | 基准年的下一年                                               |
| fy2        | 基准年的下下年                                               |
| fy3        | 基准年的往后第三年                                           |

以下以fy0前缀举例，对朝阳永续个股一致预期字段说明：

| 数据名                       | 代码                               | 说明                                                         |
| ---------------------------- | ---------------------------------- | ------------------------------------------------------------ |
| 一致预期营业收入             | fy0_F2、fy1_F19、fy2_F36、fy3_F53  | 万元                                                         |
| 一致预期归母净利润           | fy0_F3、fy1_F20、fy2_F37、fy3_F54  | 万元                                                         |
| 一致预期每股收益             | fy0_F4、fy1_F21、fy2_F38、fy3_F55  | 元                                                           |
| 一致预期净资产               | fy0_F5、fy1_F22、fy2_F39、fy3_F56  | 万元                                                         |
| 一致预期市净率               | fy0_F6、fy1_F23、fy2_F40、fy3_F57  |                                                              |
| 一致预期市销率               | fy0_F7、fy1_F24、fy2_F41、fy3_F58  |                                                              |
| 一致预期市盈率               | fy0_F8、fy1_F25、fy2_F42、fy3_F59  |                                                              |
| 一致预期PEG                  | fy0_F9、fy1_F26、fy2_F43、fy3_F60  |                                                              |
| 一致预期净资产收益率         | fy0_F10、fy1_F27、fy2_F44、fy3_F61 |                                                              |
| 一致预期营业收入同比         | fy0_F11、fy1_F28、fy2_F45、fy3_F62 |                                                              |
| 一致预期净利润同比           | fy0_F12、fy1_F29、fy2_F46、fy3_F63 |                                                              |
| 一致预期净利润两年复合增长率 | fy0_F13、fy1_F30、fy2_F47、fy3_F64 | 100 * ( SQRT(t2期一致预期净利润 / t0期实际净利润) -1 )       |
| 1周一致预期净利润变化率      | fy0_F14、fy1_F31、fy2_F48、fy3_F65 | 100 * (统计日个股一致预期净利润－统计日1周前个股一致预期净利润)/ABS (统计日1周前个股一致预期净利润)，下类似 |
| 4周一致预期净利润变化率      | fy0_F15、fy1_F32、fy2_F49、fy3_F66 |                                                              |
| 13周一致预期净利润变化率     | fy0_F16、fy1_F33、fy2_F50、fy3_F67 |                                                              |
| 26周一致预期净利润变化率     | fy0_F17、fy1_F34、fy2_F51、fy3_F68 |                                                              |
| 52周一致预期净利润变化率     | fy0_F18、fy1_F35、fy2_F52、fy3_F69 |                                                              |

朝阳超预期因子

| 数据名     | 代码                | 说明                                                         |
| ---------- | ------------------- | ------------------------------------------------------------ |
| 超预期幅度 | excess_forcast_rate | 超预期幅度=(实际净利润或调整后一致预期净利润 - 一致预期净利润) / ABS(一致预期净利润) |

PIT财务因子

| 数据名                                  | 代码                | 说明                                                         |
| --------------------------------------- | ------------------- | ------------------------------------------------------------ |
| 营业收入(TTM)                           | or                  |                                                              |
| 净利润(TTM)                             | profit              |                                                              |
| 归属母公司股东的净利润(TTM)             | net_profit          |                                                              |
| 经营活动现金净流量(TTM)                 | op_cashflow         |                                                              |
| 总资产净利率(TTM)                       | roa1                |                                                              |
| 总资产报酬率(TTM)                       | roa2                |                                                              |
| 净资产收益率(TTM)                       | roe0                |                                                              |
| EBITDA(TTM反推法)                       | ebitda              |                                                              |
| EBIT(TTM)                               | ebit                |                                                              |
| 投入资本回报率ROIC(TTM)                 | roic2               |                                                              |
| 投入资本回报率ROIC(TTM)                 | roic0               |                                                              |
| 资产负债率(MRQ)                         | debt_to_asset       |                                                              |
| 增长率-经营活动产生的现金流量净额（TTM) | op_cf_grow          | (本期CFO(TTM)-上年同期CFO(TTM))/ABS(上年同期CFO(TTM))*100，下同 |
| 增长率-净利润(TTM)                      | profit_grow         |                                                              |
| 同比增长率-归属母公司股东的净利润(TTM)  | net_profit_grow     |                                                              |
| 净资产周转率(TTM)                       | nat                 | 营业总收入(TTM)*2/(本期股东权益+上年同期股东权益)            |
| 资产回报率(TTM)                         | roa0                | 净利润(TTM)×2/(本期资产总计(MRQ)+上年同期资产总计(MRQ))*100  |
| 权益回报率(TTM)                         | roe2                | 净利润(TTM)×2/(本期股东权益(MRQ)+上年同期股东权益(MRQ))*100  |
| 资本报酬率(TTM)                         | roc                 | 净利润(TTM)×2/(本期股东权益(MRQ)+本期长期负债(MRQ)+上年同期股东权益(MRQ)+上年同期长期负债(MRQ))*100 |
| 变化趋势-八季度净利润                   | earn_mom            | 统计前8个季度的净利润,如果同比(去年同期)增长记为+1,同比下滑记为-1,再将8个值相加 |
| 5年平均资产回报率                       | roa_avg5            | AVG(净利润×2/(本年资产总计(MRQ)+上年末资产总计(MRQ))*100,限定只计算过去五年的年报 |
| 5年平均权益回报率                       | roe_avg5            | AVG(净利润×2/(本年股东权益(MRQ)+上年末股东权益(MRQ))*100,限定只计算过去五年的年报 |
| 销售净利率(TTM)                         | net_profit_margin   |                                                              |
| 销售毛利率(TTM)                         | gross_profit_margin |                                                              |

Level2指标

| 数据名         | 代码       | 说明                                                         |
| -------------- | ---------- | ------------------------------------------------------------ |
| 主买比率(%)    | windl2_F2  | 主动性买入成交金额占总成交金额的比例                         |
| 主买总额(万元) | windl2_F3  | 主动买入的总成交金额 注:金额=成交价格*成交量(深市)           |
| 主买总量(手)   | windl2_F4  | 主动买入的总成交量                                           |
| 主卖比率(%)    | windl2_F5  | 主动性卖出成交金额占总成交金额的比例                         |
| 主卖总额(万元) | windl2_F6  | 主动卖出的总成交额                                           |
| 主卖总量(手)   | windl2_F7  | 主动卖出的总成交量                                           |
| 大买比率(%)    | windl2_F8  | 大单买入成交金额占总成交金额的比例 注:所谓大单=机构单+大户单(沪市) |
| 大买总额(万元) | windl2_F9  | 大单买入的总成交额                                           |
| 大买总量(手)   | windl2_F10 | 大单买入的总成交量                                           |
| 大卖比率(%)    | windl2_F11 | 大单卖出成交金额占总成交金额的比例                           |
| 大卖总额(万元) | windl2_F12 | 大单卖出的总成交额                                           |
| 大卖总量(手)   | windl2_F13 | 大单卖出的总成交量                                           |

资金流数据

| 数据名                              | 代码           | 说明                                                         |
| ----------------------------------- | -------------- | ------------------------------------------------------------ |
| 机构买入金额(万元)                  | windcf_F3      | 当日机构买入金额(机构前称:特大单,本表其他字段的“机构前称”同;单笔成交额大于100万元:机构) |
| 机构卖出金额(万元)                  | windcf_F4      | 当日机构卖出金额                                             |
| 大户买入金额(万元)                  | windcf_F5      | 当日大户买入金额(大户前称:大单,本表其他字段的“大户前称”同;单笔成交额20万元至100万元之间:大户) |
| 大户卖出金额(万元)                  | windcf_F6      | 当日大户卖出金额                                             |
| 中户买入金额(万元)                  | windcf_F7      | 当日中户买入金额(中户前称:中单,本表其他字段的“中户前称”同;单笔成交额 4万元到20万元之间:中户) |
| 中户卖出金额(万元)                  | windcf_F8      | 当日中户卖出金额                                             |
| 散户买入金额(万元)                  | windcf_F9      | 当日散户买入金额(散户前称:小单,本表其他字段的“小户前称”同;单笔成交额小于4万元:散户) |
| 散户卖出金额(万元)                  | windcf_F10     | 当日散户卖出金额                                             |
| 机构买入总量(手)                    | windcf_F11     | 当日机构买入总手数                                           |
| 机构卖出总量(手)                    | windcf_F12     | 当日机构卖出总手数                                           |
| 大户买入总量(手)                    | windcf_F13     | 当日大户买入总手数                                           |
| 大户卖出总量(手)                    | windcf_F14     | 当日大户卖出总手数                                           |
| 中户买入总量(手)                    | windcf_F15     | 当日中户买入总手数                                           |
| 中户卖出总量(手)                    | windcf_F16     | 当日中户卖出总手数                                           |
| 散户买入总量(手)                    | windcf_F17     | 当日散户买入总手数                                           |
| 散户卖出总量(手)                    | windcf_F18     | 当日散户卖出总手数                                           |
| 成交笔数(笔)                        | windcf_F19     | 当日个股成交总笔数                                           |
| 机构买入单数(单)                    | windcf_F20     | 当日买入特大单的单数                                         |
| 机构卖出单数(单)                    | windcf_F21     | 当日卖出特大单的单数                                         |
| 大户买入单数(单)                    | windcf_F22     | 当日买入大单的单数                                           |
| 大户卖出单数(单)                    | windcf_F23     | 当日卖出大单的单数                                           |
| 中户买入单数(单)                    | windcf_F24     | 当日买入中单的单数                                           |
| 中户卖出单数(单)                    | windcf_F25     | 当日卖出中单的单数                                           |
| 散户买入单数(单)                    | windcf_F26     | 当日买入小单的单数                                           |
| 散户卖出单数(单)                    | windcf_F27     | 当日卖出小单的单数                                           |
| 散户量差(含主动被动)(手)            | windcf_F28     | 小单买量-小单卖量(手)                                        |
| 散户量差(仅主动)(手)                | windcf_F29     | 小单主动买量-小单主动卖量(手)                                |
| 中户量差(含主动被动)(手)            | windcf_F30     | 中单买量-中单卖量(手)                                        |
| 中户量差(仅主动)(手)                | windcf_F31     | 中单主动买量-中单主动卖量(手)                                |
| 大户量差(含主动被动)(手)            | windcf_F32     | 大单买量-大单卖量(手)                                        |
| 大户量差(仅主动)(手)                | windcf_F33     | 大单主动买量-大单主动卖量(手)                                |
| 机构量差(含主动被动)(手)            | windcf_F34     | 特大单买量-特大单卖量(手)                                    |
| 机构量差(仅主动)(手)                | windcf_F35     | 特大单主动买量-特大单主动卖量(手)                            |
| 散户金额差(含主动被动)(万元)        | windcf_F36     | 小单买额-小单卖额                                            |
| 散户金额差(仅主动)(万元)            | windcf_F37     | 小单主动买额-小单主动卖额                                    |
| 中户金额差(含主动被动)(万元)        | windcf_F38     | 中单买额-中单卖额                                            |
| 中户金额差(仅主动)(万元)            | windcf_F39     | 中单主动买额-中单主动卖额                                    |
| 大户金额差(含主动被动)(万元)        | windcf_F40     | 大单买额-大单卖额                                            |
| 大户金额差(仅主动)(万元)            | windcf_F41     | 大单主动买额-大单主动卖额                                    |
| 机构金额差(含主动被动)(万元)        | windcf_F42     | 特大单买额-特大单卖额                                        |
| 机构金额差(仅主动)(万元)            | windcf_F43     | 特大单主动买额-特大单主动卖额                                |
| 净流入量(手)                        | windcf_F44     | 当日买入总量(仅主动)-当日卖出总量(仅主动)                    |
| 流入率(量)(%)                       | windcf_F45     | 当日净流入量/成交股数,仅主动                                 |
| 开盘资金流入量(手)                  | windcf_F46     | 10点前的资金净流入量,仅主动                                  |
| 开盘资金流入率(量)(%)               | windcf_F47     | 10点前的资金净流入量/10点前的成交股数,仅主动                 |
| 尾盘资金流入量(手)                  | windcf_F48     | 14:30后的资金净流入量,仅主动                                 |
| 尾盘资金流入率(量)(%)               | windcf_F49     | 14:30后的资金净流入量/14:30后的成交股数,仅主动               |
| 净流入金额(万元)                    | windcf_F50     | 当日买入总额(仅主动)-当日卖出总额(仅主动)                    |
| 流入率(金额)                        | windcf_F51     | 即净流入率,当日净流入/成交额,仅主动;是金额的比例             |
| 开盘资金流入金额(万元)              | windcf_F52     | 10点前的资金净流入金额,仅主动                                |
| 开盘资金流入率(金额)                | windcf_F53     | 10点前的资金净流入金额/10点前的成交额,仅主动;是金额的比例    |
| 尾盘资金流入金额(万元)              | windcf_F54     | 14:30后的资金净流入金额,仅主动                               |
| 尾盘资金流入率(金额)                | windcf_F55     | 14:30后的资金净流入金额/14:30后的成交额,仅主动;是金额的比例  |
| ~~委买总量(手)~~                    | ~~windcf_F56~~ | 当日委买总量,**已废弃**                                      |
| ~~委卖总量(手)~~                    | ~~windcf_F57~~ | 当日委卖总量,**已废弃**                                      |
| 资金流向占比(量)(%)                 | windcf_F58     | 当日资金净流入量/流通股本,仅主动                             |
| 开盘资金流向占比(量)(%)             | windcf_F59     | 10点前的资金净流入量/流通股本,仅主动                         |
| 尾盘资金流向占比(量)(%)             | windcf_F60     | 14:30后的资金净流入量/流通股本,仅主动                        |
| 资金流向占比(金额)                  | windcf_F61     | 当日资金净流入金额/流通市值,仅主动;是金额的比例              |
| 开盘资金流向占比(金额)              | windcf_F62     | 10点前的资金净流入金额/流通市值,仅主动;是金额的比例          |
| 尾盘资金流向占比(金额)              | windcf_F63     | 14:30后的资金净流入金额/流通市值,仅主动;是金额的比例         |
| 大单净流入量(手)                    | windcf_F64     | 机构买入总量+大户买入总量-(机构卖出总量+大户卖出总量),字段名中包含“大单”的,指机构与大户的合计,本表其他字段名中包含“大单”的同 |
| 大单流入率(量)(%)                   | windcf_F65     | 大单净流入量/成交股数                                        |
| 大单净流入金额(万元)                | windcf_F66     | 机构买入金额+大户买入金额-(机构卖出金额+大户卖出金额)        |
| [内部]大单流入率(金额)(%)           | windcf_F67     | 大单净流入金额/成交金额*100,单位%                            |
| 大单资金流向占比(量)(%)             | windcf_F68     | 大单净流入量/流通股数*100,单位%                              |
| [内部]大单资金流向占比(金额)(%)     | windcf_F69     | 大单净流入金额/流通市值*100;单位%                            |
| 大单开盘资金流入量(手)              | windcf_F70     | 10点前的大单净流入量                                         |
| [内部]大单开盘资金流入率(量)(%)     | windcf_F71     | 10点前的大单净流入量/10点前的成交股数*100,单位%              |
| 大单开盘资金流入金额(万元)          | windcf_F72     | 10点前的大单净流入金额                                       |
| [内部]大单开盘资金流入率(金额)(%)   | windcf_F73     | 10点前的大单净流入金额/10点前的成交金额*100,单位%            |
| [内部]大单开盘资金流向占比(量)(%)   | windcf_F74     | 10点前的大单净流入量/流通股数*100,单位%                      |
| 大单开盘资金流向占比(金额)(%)       | windcf_F75     | 10点前的大单净流入金额/流通市值*100,单位%                    |
| 大单尾盘资金流入量(手)              | windcf_F76     | 14:30后的大单净流入量                                        |
| [内部]大单尾盘资金流入率(量)(%)     | windcf_F77     | 14:30后的大单净流入量/14:30后的成交股数*100,单位%            |
| 大单尾盘资金流入金额(万元)          | windcf_F78     | 14:30后的大单净流入金额                                      |
| [内部]大单尾盘资金流入率(金额)(%)   | windcf_F79     | 14:30后的大单净流入金额/14:30后的成交金额*100,单位%          |
| 大单尾盘资金流向占比(量)(%)         | windcf_F80     | 14:30后的大单净流入量/流通股数*100,单位%                     |
| [内部]大单尾盘资金流向占比(金额)(%) | windcf_F81     | 14:30后的大单净流入金额/流通市值*100,单位%                   |
| 机构买入金额(仅主动)(万元)          | windcf_F82     |                                                              |
| 机构卖出金额(仅主动)(万元)          | windcf_F83     |                                                              |
| 大户买入金额(仅主动)(万元)          | windcf_F84     |                                                              |
| 大户卖出金额(仅主动)(万元)          | windcf_F85     |                                                              |
| 中户买入金额(仅主动)(万元)          | windcf_F86     |                                                              |
| 中户卖出金额(仅主动)(万元)          | windcf_F87     |                                                              |
| 散户买入金额(仅主动)(万元)          | windcf_F88     |                                                              |
| 散户卖出金额(仅主动)(万元)          | windcf_F89     |                                                              |
| 机构买入总量(仅主动)(万股)          | windcf_F90     |                                                              |
| 机构卖出总量(仅主动)(万股)          | windcf_F91     |                                                              |
| 大户买入总量(仅主动)(万股)          | windcf_F92     |                                                              |
| 大户卖出总量(仅主动)(万股)          | windcf_F93     |                                                              |
| 中户买入总量(仅主动)(万股)          | windcf_F94     |                                                              |
| 中户卖出总量(仅主动)(万股)          | windcf_F95     |                                                              |
| 散户买入总量(仅主动)(万股)          | windcf_F96     |                                                              |
| 散户卖出总量(仅主动)(万股)          | windcf_F97     |                                                              |

北上资金数据

| 数据名       | 代码        | 说明                              |
| ------------ | ----------- | --------------------------------- |
| 北上资金金额 | north_netin | 陆股通-当日买入成交净额（人民币） |

融资融券数据

| 数据名               | 代码  | 说明 |
| -------------------- | ----- | ---- |
| 沪交所融资余额（元） | tb_sh |      |
| 沪交所融券余额（元） | lb_sh |      |
| 深交所融资余额（元） | tb_sz |      |
| 深交所融券余额（元） | lb_sz |      |

### 2.指数可用数据

（以HS300为例，另有ZZ500、ZZ800、ZZ1000、SZ50）

| 数据名                       | 代码             | 说明                                                         |
| ---------------------------- | ---------------- | ------------------------------------------------------------ |
| 昨收盘价（元）               | HS300_close_prev |                                                              |
| 开盘价（元）                 | HS300_open       |                                                              |
| 最高价（元）                 | HS300_high       |                                                              |
| 最低价（元）                 | HS300_low        |                                                              |
| 收盘价（元）                 | HS300_close      |                                                              |
| 涨跌幅                       | HS300_return     | ROUND((close - close_prev) * 100 / close_prev, 2)            |
| 成交量（手）                 | HS300_volume     |                                                              |
| 成交金额（万元）             | HS300_value      |                                                              |
| 上涨的股票占指数成份股的比例 | HS300_uppct      | 上涨股票定义为涨跌幅>0的成分股                               |
| 下跌的股票占指数成份股的比例 | HS300_downpct    | 下跌股票定义为涨跌幅<0的成分股                               |
| 算术平均滚动市盈率           | HS300_pe2        | 计算每个指数成份股的的PE,去除小于0和大于1000的值,然后做简单平均 |
| 算术平均滚动市净率           | HS300_pb2        | 计算每个指数成份股的的PB,去除小于0和大于1000的值,然后做简单平均 |
| PE(中位数)                   | HS300_pe3        | 指数成份股市盈率中位数                                       |
| PB(中位数)                   | HS300_pb3        | 指数成份股市净率中位数                                       |

### 3.期货可用数据

| 名称                         | 代码                        | 注释                                             |
| ---------------------------- | --------------------------- | ------------------------------------------------ |
| 开盘价                       | open                        | 未复权                                           |
| 收盘价                       | close                       | 未复权                                           |
| 日最高价                     | high                        | 未复权                                           |
| 日最低价                     | low                         | 未复权                                           |
| 结算价                       | settle                      | 未复权                                           |
| ~~昨收盘价~~                 | ~~close_prev~~              | **已弃用，期货以settle价作为参考价格**           |
| 昨结算价                     | settle_prev                 | 相对于settle是复权的                             |
| 除权xxx价                    | adjxxx                      | 以上所有价格的除权版本                           |
| 除权因子                     | cumadj                      | adjxxx = xxx * cumadj                            |
| 收益率                       | return                      | settle / settle_prev，见表格下说明               |
| ~~收益率（收盘价）~~         | ~~return_ref_close~~        | ~~close / close_prev~~，**已弃用**               |
| 成交量（手）（单边）         | volume_ss                   | 不要用volume，下同                               |
| 成交额（单边）               | value_ss                    |                                                  |
| 持仓量（手）（单边）         | oi_ss                       |                                                  |
| 品种总成交量（单边）         | total_volume_ss             |                                                  |
| 品种总成交金额（单边）       | total_value_ss              |                                                  |
| 品种总持仓量（单边）         | total_oi_ss                 |                                                  |
| 库存仓单                     | instock                     |                                                  |
| 周库存合计                   | deliverable                 | 注意nan                                          |
| 入仓                         | warein                      |                                                  |
| 出仓                         | wareout                     |                                                  |
| 展期收益率                   | carry                       | 近月主力合约到远月次主力的展期收益率             |
| 主力合约收益率               | return1                     |                                                  |
| 次主力合约收益率             | return2                     |                                                  |
| 前5/10/15/20大户成交合计     | pos_vol_top5/10/15/20       |                                                  |
| 前5/10/15/20大户持仓多头合计 | pos_long_top5/10/15/20      |                                                  |
| 前5/10/15/20大户持仓空头合计 | pos_short_top5/10/15/20     |                                                  |
| 品种各合约大户xxx合计        | total_pos_yyy_top5/10/15/20 | xxx: 成交/持仓多头/持仓空头; yyy: vol/long/short |
| n日收益波动率                | vol{n}                      | n=10,20,60,120                                   |
| n日收益波动率（含上下阈值）  | capvol{n}                   | n=10,20,60,120，上下阈值分别为5%和0.1%           |

说明：为什么收益率计算可以用未复权的 settle / settle_prev - 1？这是因为 settle_prev 这个数据相对 settle 是经过复权处理的，事实上，期货的收益率 return 只有在换月当天不等于 settle / offset(settle, -1)，因为分子分母不是同一个合约的；settle_prev 对此进行了修复，保证 return = settle / settle_prev - 1 在同一个合约上计算。

### 4.BW离线数据股票池（bw_daily）

| 数据                | 名称                       | 说明                                                         |
| ------------------- | -------------------------- | ------------------------------------------------------------ |
| daily_closePrice    | 未复权日收盘价             | 类似的还有openPrice/highPrice/lowPrice/vwapPrice             |
| cumadjfactor        | 复权系数矩阵               | **注意与future股票池不同，这里是三维矩阵**                   |
| daily_close         | 复权日收盘价               | 类似的还有open/high/low/vwap                                 |
| daily_close_prev    | 复权昨收盘价               | 类似的还有open和vwap的prev复权价格                           |
| daily_return        | 日收益率                   | = daily_close / daily_close_prev - 1                         |
| daily_open_return   | 日开盘价的收益率           | = daily_open / daily_open_prev - 1                           |
| daily_vwap_return   | 日vwap价的收益率           | = daily_vwap / daily_vwap_prev - 1                           |
| daily_volume        | 日总成交量                 |                                                              |
| daily_oi            | 日持仓量                   |                                                              |
| intraday_std        | 日内收益率序列的波动率     |                                                              |
| intraday_price_std  | 日内价格序列的波动率       | 是未复权close价格序列的波动率                                |
| intraday_volume_std | 日内成交量序列的波动率     | 日内成交量是流量                                             |
| intraday_oi_std     | 日内持仓量序列的波动率     | 日内持仓量是存量                                             |
| daily_var20         | 日频方差                   | 类似还有daily_var10, 30, 60, 120                             |
| daily_capVar20      | 日频方差                   | 比daily_var20加了上下界处理，类似还有daily_capVar10, 30, 60, 120 |
| daily_vol20         | 日频波动率                 | 类似还有daily_vol10, 30, 60, 120                             |
| daily_capVol20      | 日频波动率                 | 比daily_vol20加了上下界处理，类似还有daily_capVol10, 30, 60, 120 |
| bw1                 | bw行业                     | 类似于f1、f2，一种静态行业分类，可以看 /data/cache/bw_daily/industry/bw1.csv |
| daily_closePrice1   | 主力合约收盘价（未复权）   | 类似的，daily_close1是主力复权收盘价                         |
| daily_closePrice2   | 远月次主力收盘价（未复权） | 类似的，daily_close2是远月次主力复权收盘价                   |
| daily_carry         | 展期收益率                 | 计算方法是 (daily_close2 / daily_close1 - 1) / #(相差月份) * 12 |
| daily_return1       | 主力合约收益率             |                                                              |
| daily_return2       | 远月次主力合约收益率       |                                                              |

### 5.BW离线数据股票池（bw_intv1, bw_intv15)

| 数据               | 名称           | 说明                                                         |
| ------------------ | -------------- | ------------------------------------------------------------ |
| intra_closePrice   | 未复权价格     | 类似还有openPrice,highPrice,lowPrice,vwapPrice               |
| intra_cumadjfactor | 复权系数       | 是个三维矩阵                                                 |
| intra_close        | 复权价格       | 类似还有open,high,low,vwap                                   |
| intra_close_prev   | 复权前收盘价格 |                                                              |
| intra_return       | 日内收益率     | = intra_close / intra_close_prev -1；注意每天的第一个 intra_return 包含了隔夜收益率 |
| intra_volume       | 日内成交量     |                                                              |
| intra_oi           | 日内持仓量     |                                                              |
| intra_var20        | 方差           | 类似还有intra_var10, 30, 60, 120                             |
| intra_carVar20     | 方差           | 比intra_var20加了上下界处理，类似还有intra_capVar10, 30, 60, 120 |
| intra_vol20        | 波动率         | 类似还有intra_vol10, 30, 60, 120                             |
| intra_capVol20     | 波动率         | 比intra_vol20加了上下界处理，类似还有intra_capVol10, 30, 60, 120 |

### 6.【重要】数据复权如何影响回测，以及在什么情况下要使用复权数据

在回测的 xml 中，portfolio 节点的 trade_price 和 refer_price 非常重要，直接影响 pnl 的计算精确与否，其中 trade_price 代表每天模拟交易的价格，一般可以假设用全天均价 vwap 交易，而 refer_price 是持仓资产规模的计算价格，对股票一般用收盘价 close，期货则用 settle 结算价（intra 股票池则为 intra_close）。以 refer_price = close 为例，系统要求必须存在 _prev 后缀的数据，也就是必须有 close_prev 数据，并以此计算相邻两个区间的收益率 close / close_prev - 1，这就要求 close_prev 相对于 close 必须是复权的，从上面的表格可以看出，这点恰好是满足的。另一方面，trade_price 可以是任何价格，只要和 refer_price 是可比的（比如你不能用复权的 refer_price 和未复权的 trade_price）。

除此之外，我们在策略中可能以多种形式用到价格数据，必须要清楚何时需要用复权数据，当价格因为（股票的）派息、分股、分红、（期货的）换月等原因产生了非连续的价格变动，就要对其复权以还原真实的价格变动，举个例子，CU 今天发生了换月，从昨天的2204主力换到了今天的2205主力合约，价格如下：

|                  | CU2204 | CU2205 | CU    |
| ---------------- | ------ | ------ | ----- |
| 昨天，settle[-1] | 10000  | 12000  | 10000 |
| 今天，settle     | 12000  | 15000  | 15000 |

对CU合约来说，今天的收益率好像是15000/10000-1=0.5，但事实上应该是15000/12000-1=0.25，因此我们需要对价格做一下调整，这就叫复权（除权）调整。具体方法为：假设今天的复权因子是x，使得 15000*x / 10000 = 15000 / 12000，那么显然 x=10000 / 12000 = settle[-1] / settle_prev，其中 settle[-1] 表示连续合约的昨日价格，settle_prev 表示真实的昨日价格（也就是表格中所谓的相对于settle是复权的）。

是否用复权价格有以下几个原则：

1. 连续信号计算都用复权价格
2. 日内价格比较可以用非复权价格
3. 复权价格和非复权价格不可比
4. 计算真实开仓手数用非复权价格

### 7.（有兴趣可以看）如何计算组合 pnl

![image2](image2.png)

## 十二、Q&A

### A. 系统相关

#### 1. 如何dump因子为csv格式

```xml
<alpha module="alpha_formula" id="af1" universe="LIQPC" expr="return">
	<operation module="alphaop_dump_alpha" days="3"/>
</alpha>
```

其中参数days表示dump最近几天的因子值，如果不写将会dump全部数据，dump下来的文件保存在~/room/history_alphas/{alpha_id}下

#### 2. 如何从csv中读取因子

```xml
<alpha module="alpha_from_csv" id="af2" universe="LIQPC" delay="0" seperator=" " csv="./history_alphas/af1/af1_20210730.csv"/>
```

可以与alphaop_dump_alpha搭配使用，seperator支持空格或者分号，需要显示定义delay=0，或者采用实际信号的delay

#### 3. 如何处理因子中的nan

因子返回nan表示告诉程序，该标的在该日因子缺失，无法交易，从而后续的因子op也就会保留其nan，切忌贸然将nan转化为0，因为一些截面因子op很有可能将其变换为非0的实数值，从而产生不准确的结果。

但还有一种情况，某些标的仅仅是数据缺失，并非不可交易；如果希望避免这种情况引发的因子异常变化，可以从两个角度考虑，一是在数据处理阶段，一些data module（比如data_fromfile）默认处理缺失值的方法是填充nan，这时可以选择向前填充，即继承前日值（比如data_formula的use_before参数），二是在因子计算阶段，可以在计算alpha时判断该标的是否仍在股票池中，如果确实也在股票池中，但因子缺失，可以考虑继承前日因子值。系统的公式表达式支持func_ffill和func_intra_ffill来向前填充nan

#### 4. 如何继承前日因子

使用alpha的一个属性prev_alpha_value，该值考虑了当日市值变动，直接返回即表示当日无换手

#### 5. 如何使用公共数据的同时更新自定义的数据

公共目录存放一些基础数据，由管理员维护，并且向所有用户开放使用（不能更改）；用户可以在自己的xml中更新自用数据，方法是设置update_data=“false”，及private_cache_path=“~/room/cache”（可任意，只要是在自己的目录里），私有数据data节点下设置cache_path=“private”即可。可参考xmlsample/update_private.xml，参见一、4.样例代码

#### 6.如何发邮件

首先你需要在~/room/private/email_config.py中配置邮箱信息，目前仅支持qq邮箱，示例：

```python
# ./private/email_config.py
# -*- coding: utf-8 -*-
email_config = {
    "qq": {
        "user": "522526392", "password": "********",
        "from": "522526392@qq.com",
        "signature": ""
    },
}
```

其中password不是邮箱账户的登陆密码，而是第三方登录邮箱的授权码，以QQ邮箱为例，你要在开启邮箱设置中的POP3服务，并“获取授权码”，然后填在password处。

在系统的某些python模块中，只要你配置好了config文件，就可以正常使用了，比如使用data_test模块，在send_to参数处写522526392@qq,com即可，如果想主动使用程序发送邮件，参考下例：

```python
from util import send_email
email = send_email.Email("qq")
email.send(to="522526392@qq.com", subject="test", text="test")
```



### B. Linux相关

#### 1. 为什么crontab定时任务执行失败

查看mail，crontab执行结果会展示在mail里，一般是调用脚本权限不足，参考二、3.a.crontab配置

#### 2. zsh弹出insecure directories怎么办

因为某些文件夹的执行权限过高，输入

`compaudit`

可以看到哪些文件夹需要调整权限，比如输出为：

```shell
% compaudit 
There are insecure directories:
/usr/share/zsh/site-functions
/usr/share/zsh/4.3.4/functions
/usr/share/zsh
/usr/share/zsh/4.3.4
```

那么执行

`sudo chmod -R 755 /usr/share/zsh`

即可

#### 3. 如何将标准输入转化为命令行参数

在linux系统中，某些命令是可以接受标准化输入的，比如echo，因此可以直接使用管道符号“|”，来将前一个命令的标准输出转化为标准输入传给它，比如：

`which stats.py | echo`

此时屏幕会输出

`/opt/hammer-stable/util/stats.py`

但也有很多命令不支持标准输入，需要接收命令行参数，此时就需要使用命令，将标准输入转化为命令行参数，详见https://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html

#### 4.如何将远程目录挂载到系统下

假如想在系统中将另一个服务器x的某个目录A挂载到本地目录B，首先要保证服务器安装了nfs服务：

`sudo apt-get install nfs-kernel-server`

然后编辑/etc/exports，在最后加上一行

`A *(rw,sync,no_subtree_check)`

启动相应服务

`sudo /etc/init.d/rpcbind restart`

`sudo /etc/init.d/nfs-kernel-server restart`

接下来在目标服务器中运行以下命令即可

`sudo mount -t nfs x:A B`

#### 5.Git push/close遇到”fatal: unable to access“怎么办

问题还没有解决

#### 6.如何正确升级系统python

从源码安装，http://www.zhushiyao.com/?p=3484

如果遇到openssl错误，尝试从源码安装它：https://stackoverflow.com/questions/53543477/building-python-3-7-1-ssl-module-failed

#### 7.Ubuntu一些软件/包的安装

oracle: https://stackoverflow.com/questions/49542537/ubuntu-16-04-oracle-module-for-python-how-to-install-cx-oracle-module-ea

mail: apt-get install mailutils

pip: apt install python3-pip; python -m pip install --upgrade pip

tkinter: apt install python3-tk

Cryptodome: python -m pip install pycryptodomex

#### 8.如何合并多个硬盘并自动挂载

https://blog.csdn.net/d1240673769/artcle/details/113999873

#### 9.Cuda和Pytorch安装、升级与卸载

参考https://blog.csdn.net/qxqxqzzz/article/details/112005275

首先升级gcc到9.x（9.3）

然后卸载其他版本的nvidia驱动

```bash
apt-get remove --purge nvidia*
apt-get autoremove
```

卸载已经安装的cuda

```bash
/usr/local/cuda-11.x/bin/cuda-uninstaller
```

安装新的驱动前要停掉lightdm，gdm等服务，并停掉nouveau，参见上面的文章，然后安装dkms

```bash
apt-get install dkms
```

然后在以下网址下载对应版本的安装包并按照说明安装cuda：

https://developer.nvidia.cn/cuda-downloads

更新PPA repo，并查看可用的驱动版本

```bash
add-apt-repository ppa:graphics-drivers/ppa
apt update
ubuntu-drivers devices
```

安装适合GPU设备的驱动

```bash
apt-get install nvidia-driver-xxx
```

cuda在安装前应该卸载其他版本

```bash
sudo apt-get autoremove --purge cuda
dpkg -l | grep cuda
dpkg -P cuda...
```

卸载其他版本的驱动（dpkg -l | grep nvidia）

然后在以下网址下载对应版本的安装包并按照说明安装cuda：

https://developer.nvidia.cn/cuda-downloads

接下来要安装对应版本的驱动，在以下地址寻找合适版本的驱动并下载：

https://www.nvidia.cn/Download/index.aspx?lang=cn

最后安装驱动

`sh NVIDIA-LINUX-x86_64_***.run`

pytorch的安装应该根据cuda版本选择，离线安装：https://zhuanlan.zhihu.com/p/162724771

#### 10. 如何在Window和Linux间互传文件

win->linux: scp -r -P 12609 public_cache.tgz renfei@43.249.193.55:~/room (12609是55的ssh服务端口，一般linux的ssh服务都是22)

linux->win rsync -av --port=873 ./hammer-stable.tgz 10.24.22.92::data/ （873是windows上安装rsync开启的端口）

#### 11.如何安装支持boost_python

首先在安装python的时候要支持动态链接库(--enable-share)：https://cloud.tencent.com/developer/article/1740752

#### 12.如何通过sudo执行alias

如果定义了一个alias，并通过sudo以su身份执行这个alias，会得到command not found，即使将这个alias配置给su也不行；正确的做法是定义alias sudo=“sudo ”（并非在su，而是在用户账户下）

#### 13.tmux窗口的历史记录太短怎么办

在tmux中运行 ctrl + b + [ 可以查看历史屏幕输出，但初始tmux保留的屏幕输出记录太短，这时可以在命令行运行以下命令

`echo 'set-option -g history_limit 60000' >> ~/.tmux.conf`

### C.Vim相关

#### 1.Vim好难用，能不能用pycharm远程链接到服务器进行远程开发？

可以，pycharm支持远程开发，但需要专业版，如果不想购买请自行破解。

### D.Git相关

#### 1.如何关联本地工程到远程仓库

假设想将本地代码关联到远程仓库xxx/xxx.git，首先在git上建立项目xxx.git，然后在本地git目录运行

`git remote add origin http://gitlab.com/xxx/yyy.git`

这里一定要http而不是https（跟代理有关），如果关联错误，可以以下代码重置

`git remote rm origin`

然后上传

`git push -u origin master`
