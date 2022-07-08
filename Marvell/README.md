# Marvell 物理层笔记

## 代码文档svn

DocSVN:   https://192.168.5.44:8443/svn/BaiBS_NRPHY_SoC

CodeSVN:  https://192.168.5.5/svn/5G-4G-MPHY

代码编译服务器：5.13

## 主要的参考资料

1、BaiBS_MPHY_1.0\09参考资料\CNF95XXO\CNF95XXO-HM-A-0.96E.pdf的Section 2 ~ 2.3；

2、BaiBS_MPHY_1.0\09参考资料\Marvell Training，这里面包含几个MHAB的培训文档；

3、BaiBS_MPHY_1.0\09参考资料\CNF95XXO SDK  这里面包含SDK的编译、运行log，以及简单的说明文档

4、192.168.5.44  ： 51-eNB/00_Document/Marvell

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





会议纪要：
大家对DPDK线程机制基本清楚了，按以下方案修改：
\1. 取消fapi内部的rte_eal_init()，恢复使用main()主流程里的rte_eal_init()。dpdk是整个工程的底层，不能在fapi内部初始化，保持架构清晰合理。
\2. 设置dpdk lcore_mask=0x7，core0作为main_lcore，core1/2分别用于fapi，lte_app。lte_app的多小区支持不清晰，暂时只起单核。
\3. 取消rte_eal_mp_remote_launch()，改用rte_eal_remote_launch()，分别launch fapi、lte_app。
\4. main()流程加入while(1){ sleep() }，避免进程退出，同时避免阻塞os，未来可加入维护和命令行响应。原始设计main()也跑lte_app，并依靠lte_app阻塞，所以不可放置core0。





打开NPA宏内存耗尽的问题，Marvell那边也不清楚，这是他们的回复：We have not seen this before, when the NPA Buffer is freed, this is available immediately for SW to allocate
    It looks like you may have exhausted the number of outstanding buffers configured.



李正明:

打开NPA宏内存耗尽的问题，Marvell那边也不清楚，这是他们的回复：We have not seen this before, when the NPA Buffer is freed, this is available immediately for SW to allocate It looks like you may have exhausted the number of outstanding buffers configured.

[@李正明](app://desktop.dingtalk.com/web_content/chatbox.html?isFourColumnMode=false#) ，[@苗沛](app://desktop.dingtalk.com/web_content/chatbox.html?isFourColumnMode=false#) ，苗沛整理完DPDK，再来看看NPA吧。原则上减少动态内存分配，后面不一定用，但机制上我们要掌握。PSM FREE CMD只是指示了地址，还有个aura参数应该是指示pool id，实际的释放操作应该是其他地方完成的。

软件上得能看到这个NPA模块，而且注册了pool管理的操作，以及对PSM FREE CMD的响应支持。



后续顺着这些线索整理吧。有‘pf’，应该是physical function，有硬件支持的。



接下来看看NPA吧，看看怎么参与到pool管理的，怎么响应PSM FREE CMD指令的。





```shell
# phyc 设备启动

$ ulimit -q unlimited
$ ulimit -c unlimited

$ dpdk-devbind.py -b vfio-pci eth0
$ cd /userdata/dpdk_pkg/dpdk_bphy_phyc
$ ./dpdk-env-setup.sh
$ ./dpdk_bphy_phyc  0 0 0

# 950刚启动，依次输入这4条命令； 以后运行PHY只输命令：  
$ ./dpdk_bphy_phyc  0 0 0
```



```shell
# marvell BSP sdk history
19  dpdk-devbind.py -b vfio-pci eth0
20  cd /userdata/dpdk_pkg/dpdk_bphy_phyc
21  ./dpdk-env-setup.sh
22  cd /userdata/dpdk_pkg/dpdk_bphy_phyc
23  ./dpdk-env-setup.sh
24  vim /etc/profile
25  pwd
26  cd  /userdata/dpdk_pkg/dpdk_bphy_sdk/
27  ll
28  chmod +x ./*
29  vim /etc/profile
30  mkdir -p /dev/huge
31  mount -t hugetlbfs none /dev/huge
32  echo 24 > /proc/sys/vm/nr_hugepages
33  ./dpdk-env-setup.sh
34  vi /etc/profile
35  ll
36  vi /etc/profile
37  vi /etc/profile
38  source /etc/profile
39  cd /userdata/dpdk_pkg/dpdk_bphy_sdk/
40  ll
41  top
42  cat /dev/tty7
43  tty 
44  cd /dev/pts/
45  ls
46  cat 1
47  vim  bphy_cmn_cfg.txt
48  ./dpdk_bphy_phyc0705_delEvent_delSpin 0 0 0 > beijin0705cpuIsolt.txt
88  tail -f beijin0705_2cpuIsolt.txt 

93  dpdk-devbind.py -s
94  ./dpdk_bphy_phyc0705_delEvent_delSpin 0 0 0 > beijin0705_3cpuIsolt.txt


145  reboot 
146  dpdk-devbind.py -s
147  dpdk-devbind.py -b vfio-pci eth0
148  dpdk-l2fwd -c f -- -p1
149  ping 172.17.3.1
150  dpdk-devbind.py -b vfio-pci eth0
151  cd /userdata/dpdk_pkg/dpdk_bphy_phyc
152  ./dpdk-env-setup.sh
153  ll
154  top
155  source /etc/profile
156  cd /userdata/xzq
157  ./dpdk_bphy_phyc0705_delEvent_delSpin 0 0 0 > beij0705_4.txt
158  ll
159  ./dpdk_bphy_phyc070201core012DelSpinlock 0 0 0


236  reboot 
237  dpdk-devbind.py -b vfio-pci eth0
238  cd /userdata/xzq/dpdk_pkg/dpdk_bphy_phyc
239  ./dpdk-env-setup.sh
240  ./dpdk_bphy_phyc 0 0 0
241  ls
242  ulimit -c unlimited
243  ./dpdk_bphy_phyc 0 0 0

263  cd /userdata/dpdk_pkg/dpdk_bphy_phyc/
264  ll
265  ./dpdk_bphy_phyc 0 0 0
266  phyinit
267  cd  /userdata/dpdk_pkg/dpdk_bphy_phyc/
268  ll
269  vim bphy_cmn_cfg.txt
270  ps -a
271  who
272  ./dpdk_bphy_phyc 0 0 0
273  ps -a
274  cd /userdata/xzq/dpdk_pkg/
275  ls
276  cd dpdk_bphy_phyc/
277  ls
278  ./dpdk_bphy_phyc 0 0 0
279  ps -a
280  kill -9 736
281  top
282  taskset -cp 5 168
283  top
284  dpdk-devbind.py -s
285  dpdk-devbind.py -u 0000:01:10.1
286  dpdk-devbind.py -u 0000:01:10.2
287  dpdk-devbind.py -u 0000:01:10.3
288  dpdk-devbind.py -u 0002:02:00.0
289  dpdk-devbind.py -b vfio-pci  0002:02:00.0
290  cd /userdata/dpdk_pkg/dpdk_bphy_phyc/
291  sh dpdk-env-setup.sh 
292  ps -a
293  ps -a
294  chmod +x ./*
295  ll
296  top
297  sh /etc/profile
298  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
299  ll
300  top
301  kill -9 499
302  vim bphy_cmn_cfg.txt
303  kill -9 501
304  kill -9 498
305  ls
306  ls
307  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
308  top
309  cat /tmp/messages 
310  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
311  ./pdump  -l 4 --file-prefix=mv_dpdk -- --pdump 'port=0,queue=*,rx-dev=/tmp/mvphy.pcap,tx-dev=/tmp/mvphy.pcap'
312  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
313  cd /root
314  ll
315  chmod +x ./pdump 
316  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
317  ./pdump  -l 4 --file-prefix=mv_dpdk -- --pdump 'port=0,queue=*,rx-dev=/tmp/mvphy.pcap,tx-dev=/tmp/mvphy.pcap'
318  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
319  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
320  ll
321  pwd
322  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
323  ./pdump  -l 4 --file-prefix=mv_dpdk -- --pdump 'port=0,queue=*,rx-dev=/tmp/mvphy.pcap,tx-dev=/tmp/mvphy.pcap'
324  top
325  ls
326  cd /
327  cd usr/
328  find -name facility.h
329  ./dpdk_bphy_phyc0706DelEventSpinMemPrint 0 0 0
```

