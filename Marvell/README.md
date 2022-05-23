# Marvell 物理层笔记

## 代码文档svn

DocSVN:   https://192.168.5.44:8443/svn/BaiBS_NRPHY_SoC

CodeSVN:  https://192.168.5.5/svn/5G-4G-MPHY

代码编译服务器：5.13

## 主要的参考资料

1、BaiBS_MPHY_1.0\09参考资料\CNF95XXO\CNF95XXO-HM-A-0.96E.pdf的Section 2 ~ 2.3；

2、BaiBS_MPHY_1.0\09参考资料\Marvell Training，这里面包含几个MHAB的培训文档；

3、BaiBS_MPHY_1.0\09参考资料\CNF95XXO SDK  这里面包含SDK的编译、运行log，以及简单的说明文档

## 主要的代码
1、75上的LTE demo代码：5G-4G-MPHY/F95_BPHY_SW；

2、95O的sdk：5G-4G-MPHY/cnf95O_dpdk_sdk/bphy_app/bphy_sdk和bphy_examples；

3、95O上适配过的LTE代码：5G-4G-MPHY/cnf95O_dpdk_sdk/bphy_app/bphy_phyc

## 群讨论

5.12：

1. 物理层团队保留少量维护x86物理层（分析解决问题，不再做新特性开发），其余重点投入Marvell
2. 5月30日开始4G端到端联调，硬件采用Demo板+Gamma452 RRU，各网元做好版本准备和自测验证
3. 自研硬件设计加速，投板生产重点保障
4. FAPI接口重点考虑走PCIe，适时启动CNF95O Full DU PCIe加速卡开发



进程退出的时候执行一下rte_eal_cleanup

bphysdk退出的时候加一下大页内存释放，现在sdk退出的时候不释放内存

5.13：

1、DLFE加速器的配置，与前级串联（正明）；

2、RFOE的代码适配，与前级串联（晓献）；

3、抓数功能的开发，跟踪上行DSP工程编译问题（杜托）；

6、PHY->MAC 各消息的地址偏移开发（老于）。

