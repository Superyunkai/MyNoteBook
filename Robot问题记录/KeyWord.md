| 来源              | 名称                                           | 描述                       |
| ----------------- | ---------------------------------------------- | -------------------------- |
| base.robot        | Parse Params From ${Reply}                     | 解析参数                   |
|                   | Get ExtData From ${Reply}                      | 得到ExData                 |
|                   | Get Record From ${Reply}                       |                            |
|                   | Get Value of ${Key} From ${Row}                | 选中某列                   |
|                   | Parse Info From ${Params}                      | 从登陆信息中解析Info字段   |
|                   | Get sid from ${Info}                           | 从登陆信息中获取sid        |
|                   | Get Flag From ${Info}                          | 从登陆信息中获取Flag       |
|                   | Parse errMsg From Reply                        | 从应答中或获取errMsg字段   |
|                   | ${Record} Should Exit Data                     | 判断返回的请求量表是否为空 |
|                   | ${Record} Exit Data Status                     | 返回的请求列表中是否有数据 |
|                   | Result of ${Reply} Should Be ${Expect}         | 检查返回值是否正确         |
|                   | ${Value} Should Range From ${Left} To ${Right} | 值范围判断                 |
|                   | ${Value} Should Between ${Left} And ${Right}   | 值范围判断                 |
|                   | add Count                                      | 计数功能                   |
|                   | ${datetime} Should in Today                    | 判断是否在今天             |
| hqInitCheck.robot | Check ${HQ_Stock} in ${HQ_Market} Inited       | 检查股票数据               |
|                   | ${TimeSpace} should With Today                 | 检查是否在今天             |
|                   | Get FenShi Data Of ${HQ_Stock} in {HQ_Market}  | 得到某支股票的分时数据     |
| hqselect.robot    | Get Index Shishi Data                          | 请求大盘数据               |
|                   | Get Stock Fenshi Data                          | 请求分时数据               |
|                   | Get US Shishi Data                             | 请求美股分时数据           |
|                   | Get Index Pankou Data                          | 请求大盘盘口数据           |
|                   | Get Stock Shishi Data                          | 请求实时数据               |
|                   | Get Stock Mingxi Data                          | 请求明细数据               |
|                   | Get Stock Mingxi Data by Time                  | 按照时间戳请求明细数据     |
|                   | Get Stock Capital Data                         | 请求资金数据               |
|                   | Get Stock 10 Trade Data                        | 请求10挡数据               |
|                   | Get Stock 500 Trade Data                       | 请求500档数据              |
|                   | Get Stock Deal Statistics Data                 | 请求IOS饼图数据            |
|                   | Get Stock Kline Data                           | 请求k线数据                |
|                   | Get Stock Belong Block                         | 请求股票所属板块           |
|                   | Parse RealPrice From ${Record}                 | 获取分时数据最新价         |
|                   | Parse Highest From ${Record}                   | 获取分时数据最高价         |
|                   | Parse Lowest From ${Record}                    | 获取分时数据最低价         |
|                   | Parse OpenPrice From ${Record}                 | 获取分时数据开盘价         |
|                   | Parse ClosePrice From ${Record}                | 获取分时数据昨收价         |
|                   | Parse UpperLimit From ${Record}                | 获取分时数据涨停价         |
|                   | Parse LowerLimit From ${Record}                | 获取分数数据跌停价         |
|                   | Parse Average From ${Record}                   | 获取分时数据均价           |
|                   | Parse VolumeRatio From ${Record}               | 获取分时数据量比           |
|                   | Parse ChangeRatio From ${Record}               | 获取分时数据换手           |
|                   | Parse CommissionRatio From ${Record}           | 获取分时数据委比           |
|                   | Parse PERWAVE From ${Record}                   | 获取分时数据振幅           |
|                   | Parse RiseScale4 From ${Record}                | 获取分时数据涨幅4          |
|                   | Parse FallPrice From ${Record}                 | 获取分时数据涨跌           |
|                   | Parse Money From ${Record}                     | 获取分时数据金额           |
|                   | Parse Volume From ${Record}                    | 获取分时数据总手           |
|                   | Parse MarketValue From ${Record}               | 获取分时数据总市值         |
|                   | Parse FreelyMarketValue From ${Record}         | 获取分时数据流通值         |
|                   | Parse PBRatio From ${Record}                   | 获取分时数据市净率         |
|                   | Parse DPERatio From ${Record}                  | 获取分时数据盈动           |
|                   | Parse SPERatio From ${Record}                  | 获取分时数据市盈静         |
|                   | Parse TTMRatio From${Record}                   | 获取分时数据市盈率TTM      |
|                   | Parse InnerDisc From ${Record}                 | 获取分时数据内盘           |
|                   | Parse OuterDisc From ${Record}                 | 获取分时数据外盘           |
|                   | Parse GeneralCapital From ${Record}            | 获取分时数据总股本         |
|                   | Parse CirculationStock From ${Record}          | 获取分时数据流通股         |
|                   | Parse RiseCount From ${Record}                 | 获取分时数据上涨           |
|                   | Parse FallCount From  ${Record}                | 获取分时数据下跌           |
|                   | Parse BuyCount From ${Record}                  | 获取分时数据委买           |
|                   | Parse SellCount From ${Record}                 | 获取分时数据委卖           |
|                   | Parse DP_CommissionRatio From ${Record}        | 获取分时数据委比           |
|                   | Parse  TotalStock From ${Record}               | 获取大盘分时的平盘         |
|                   | Parse DP_DPERatio From ${Record}               | 获取大盘分时的市盈动       |
|                   |                                                |                            |

