cptp case1 file log & run scripts

```
CCLMDIR=/HGST_SATA_8T_3/yycheng/CCLM_starter_test/cclm-sp-v3.1.1 #编译出的CCKN位置
CASEDIR=/HGST_SATA_8T_3/yycheng/CCLM_starter_test/cptp_test/ERA5_to_cclm_cptp #处理此次CASE的文件夹
DATADIR=/HGST_SATA_8T_3/yycheng/data_stage/cptp/ # 所有运行数据的位置
```

./ERA5 冷数据

./cptp_out 模式中间热数据

./cptp_ext_test EXTPAR参数，WEBpep生成后上传

---

2020.10.13 部署到.19服务器上    
之后主要用于进行不同版本调参的版本控制

10.13是第一次进行CASE1的模拟结果，相应的参数文件进行下冻结,为之后进行敏感性分析做准备