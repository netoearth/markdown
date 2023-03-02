发表于 2021-05-14 | 分类于 [CPU](https://plantegg.github.io/categories/CPU/) | 次

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA%EF%BC%9F "十年后数据库还是不敢拥抱NUMA？")十年后数据库还是不敢拥抱NUMA？

在2010年前后MySQL、PG、Oracle数据库在使用NUMA的时候碰到了性能问题，流传最广的这篇 [MySQL – The MySQL “swap insanity” problem and the effects of the NUMA architecture](http://blog.jcole.us/2010/09/28/mysql-swap-insanity-and-the-numa-architecture/) 描述了性能问题的原因(文章中把原因找错了)以及解决方案：关闭NUMA。 实际这个原因是kernel实现的一个低级bug，这个Bug在[2014年修复了](https://github.com/torvalds/linux/commit/4f9b16a64753d0bb607454347036dc997fd03b82)，但是修复这么多年后仍然以讹传讹，这篇文章希望正本清源、扭转错误的认识。

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E8%83%8C%E6%99%AF "背景")背景

最近在做一次性能测试的时候发现MySQL实例有一个奇怪现象，在128core的物理机上运行三个MySQL实例，每个实例分别绑定32个物理core，绑定顺序就是第一个0-31、第二个32-63、第三个64-95，实际运行结果让人大跌眼镜，如下图

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620953504602-30988926-85d8-4af1-996d-f35aa5fede00.png)](https://plantegg.github.io/images/951413iMgBlog/1620953504602-30988926-85d8-4af1-996d-f35aa5fede00.png)

从CPU消耗来看差异巨大，高的实例CPU用到了2500%，低的才488%，差了5倍。但是神奇的是他们的QPS一样，执行的SQL也是一样

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620953709047-cbe4b59c-aa2b-4845-8b59-9ed6d07e3916.png)](https://plantegg.github.io/images/951413iMgBlog/1620953709047-cbe4b59c-aa2b-4845-8b59-9ed6d07e3916.png)  
所有MySQL实例流量一样

那么问题来了为什么在同样的机器上、同样的流量下CPU使用率差了这么多？ 换句话来问就是CPU使用率高就有效率吗？

这台物理机CPU 信息  

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p></pre></td><td><pre><p>#lscpu</p><p>Architecture:          aarch64</p><p>Byte Order:            Little Endian</p><p>CPU(s):                128</p><p>On-line CPU(s) list:   0-127</p><p>Thread(s) per core:    1</p><p>Core(s) per socket:    64</p><p>Socket(s):             2</p><p>NUMA node(s):          1</p><p>Model:                 3</p><p>BogoMIPS:              100.00</p><p>L1d cache:             32K</p><p>L1i cache:             32K</p><p>L2 cache:              2048K</p><p>L3 cache:              65536K</p><p>NUMA node0 CPU(s):     0-127</p><p>Flags:                 fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid</p></pre></td></tr></tbody></table>

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E5%8E%9F%E5%9B%A0%E5%88%86%E6%9E%90 "原因分析")原因分析

先来看这两个MySQL 进程的Perf数据

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p><p>39</p><p>40</p><p>41</p><p>42</p><p>43</p><p>44</p><p>45</p><p>46</p><p>47</p></pre></td><td><pre><p>#第二个 MySQL IPC只有第三个的30%多点，这就是为什么CPU高这么多，但是QPS差不多</p><p>perf stat -e branch-misses,bus-cycles,cache-misses,cache-references,cpu-cycles,instructions,L1-dcache-load-misses,L1-dcache-loads,L1-dcache-store-misses,L1-dcache-stores,L1-icache-load-misses,L1-icache-loads,branch-load-misses,branch-loads,dTLB-load-misses,iTLB-load-misses  -a -p 61238</p><p>^C</p><p> Performance counter stats for process id '61238':</p><p>        86,491,052      branch-misses                                                 (58.55%)</p><p>    98,481,418,793      bus-cycles                                                    (55.64%)</p><p>       113,095,618      cache-misses              #    6.169 % of all cache refs      (53.20%)</p><p>     1,833,344,484      cache-references                                              (52.00%)</p><p>   101,516,165,898      cpu-cycles                                                    (57.09%)</p><p>     4,229,190,014      instructions              #    0.04  insns per cycle          (55.91%)</p><p>       111,780,025      L1-dcache-load-misses     #    6.34% of all L1-dcache hits    (55.40%)</p><p>     1,764,421,570      L1-dcache-loads                                               (52.62%)</p><p>       112,261,128      L1-dcache-store-misses                                        (49.34%)</p><p>     1,814,998,338      L1-dcache-stores                                              (48.51%)</p><p>       219,372,119      L1-icache-load-misses                                         (49.56%)</p><p>     2,816,279,627      L1-icache-loads                                               (49.15%)</p><p>        85,321,093      branch-load-misses                                            (50.38%)</p><p>     1,038,572,653      branch-loads                                                  (50.65%)</p><p>        45,166,831      dTLB-load-misses                                              (51.98%)</p><p>        29,892,473      iTLB-load-misses                                              (52.56%)</p><p>       1.163750756 seconds time elapsed</p><p>#第三个 MySQL</p><p>perf stat -e branch-misses,bus-cycles,cache-misses,cache-references,cpu-cycles,instructions,L1-dcache-load-misses,L1-dcache-loads,L1-dcache-store-misses,L1-dcache-stores,L1-icache-load-misses,L1-icache-loads,branch-load-misses,branch-loads,dTLB-load-misses,iTLB-load-misses  -a -p 53400</p><p>^C</p><p> Performance counter stats for process id '53400':</p><p>       295,575,513      branch-misses                                                 (40.51%)</p><p>   110,934,600,206      bus-cycles                                                    (39.30%)</p><p>       537,938,496      cache-misses              #    8.310 % of all cache refs      (38.99%)</p><p>     6,473,688,885      cache-references                                              (39.80%)</p><p>   110,540,950,757      cpu-cycles                                                    (46.10%)</p><p>    14,766,013,708      instructions              #    0.14  insns per cycle          (46.85%)</p><p>       538,521,226      L1-dcache-load-misses     #    8.36% of all L1-dcache hits    (48.00%)</p><p>     6,440,728,959      L1-dcache-loads                                               (46.69%)</p><p>       533,693,357      L1-dcache-store-misses                                        (45.91%)</p><p>     6,413,111,024      L1-dcache-stores                                              (44.92%)</p><p>       673,725,952      L1-icache-load-misses                                         (42.76%)</p><p>     9,216,663,639      L1-icache-loads                                               (38.27%)</p><p>       299,202,001      branch-load-misses                                            (37.62%)</p><p>     3,285,957,082      branch-loads                                                  (36.10%)</p><p>       149,348,740      dTLB-load-misses                                              (35.20%)</p><p>       102,444,469      iTLB-load-misses                                              (34.78%)</p><p>       8.080841166 seconds time elapsed</p></pre></td></tr></tbody></table>

从上面可以看到 IPC 差异巨大0.04 VS 0.14 ，也就是第一个MySQL的CPU效率很低，我们看到的CPU running实际是CPU在等待(stall)。

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#CPU%E7%9A%84%E5%AE%9E%E9%99%85%E4%BF%A1%E6%81%AF "CPU的实际信息")CPU的实际信息

找到同一个机型，但是NUMA开着的查了一下：

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p></pre></td><td><pre><p>#lscpu</p><p>Architecture:          aarch64</p><p>Byte Order:            Little Endian</p><p>CPU(s):                128</p><p>On-line CPU(s) list:   0-127</p><p>Thread(s) per core:    1</p><p>Core(s) per socket:    64</p><p>Socket(s):             2</p><p>NUMA node(s):          16</p><p>Model:                 3</p><p>BogoMIPS:              100.00</p><p>L1d cache:             32K</p><p>L1i cache:             32K</p><p>L2 cache:              2048K</p><p>L3 cache:              65536K</p><p>NUMA node0 CPU(s):     0-7</p><p>NUMA node1 CPU(s):     8-15</p><p>NUMA node2 CPU(s):     16-23</p><p>NUMA node3 CPU(s):     24-31</p><p>NUMA node4 CPU(s):     32-39</p><p>NUMA node5 CPU(s):     40-47</p><p>NUMA node6 CPU(s):     48-55</p><p>NUMA node7 CPU(s):     56-63</p><p>NUMA node8 CPU(s):     64-71</p><p>NUMA node9 CPU(s):     72-79</p><p>NUMA node10 CPU(s):    80-87</p><p>NUMA node11 CPU(s):    88-95</p><p>NUMA node12 CPU(s):    96-103</p><p>NUMA node13 CPU(s):    104-111</p><p>NUMA node14 CPU(s):    112-119</p><p>NUMA node15 CPU(s):    120-127</p><p>Flags:                 fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid</p></pre></td></tr></tbody></table>

这告诉我们实际上这个机器有16个NUMA，跨NUMA访问内存肯定比访问本NUMA内的要慢几倍。

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E5%85%B3%E4%BA%8ENUMA "关于NUMA")关于NUMA

如下图，是一个Intel Xeon E5 CPU的架构信息，左右两边的大红框分别是两个NUMA，每个NUMA的core访问直接插在自己红环上的内存必然很快，如果访问插在其它NUMA上的内存还要走两个红环之间上下的黑色箭头线路，所以要慢很多。

[![img](https://plantegg.github.io/images/951413iMgBlog/1623830161880-c4c74f4d-785e-4274-a579-5d1aa8b5e990.png)](https://plantegg.github.io/images/951413iMgBlog/1623830161880-c4c74f4d-785e-4274-a579-5d1aa8b5e990.png)

实际测试Intel的E5-2682（对应V42机型）和8269（对应V62机型） 的CPU跨Socket（这两块CPU内部不再是上图的红环Bus,而是改用了Mesh Bus一个Die就是一个NUMA，服务器有两路，也就是一个Socket就是一个NUMA），也就是跨NUMA访问内存的延迟是本Node延迟的将近2倍。[测试工具从这里下载](https://software.intel.com/content/www/us/en/develop/articles/intelr-memory-latency-checker.html)

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p></pre></td><td><pre><p>//E5-2682</p><p>Intel(R) Memory Latency Checker - v3.9</p><p>Measuring idle latencies (in ns)...</p><p>Numa node</p><p>Numa node     0     1</p><p>       0  85.0 136.3</p><p>       1 137.2  84.2</p><p>//8269</p><p>Intel(R) Memory Latency Checker - v3.9  </p><p>Measuring idle latencies (in ns)...</p><p>    Numa node</p><p>Numa node      0       1</p><p>       0    78.6   144.1</p><p>       1   144.7    78.5</p></pre></td></tr></tbody></table>

开启NUMA会优先就近使用内存，在本NUMA上的内存不够的时候可以选择回收本地的PageCache还是到其它NUMA 上分配内存，这是可以通过Linux参数 zone\_reclaim\_mode 来配置的，默认是到其它NUMA上分配内存，也就是跟关闭NUMA是一样的。

**这个架构距离是物理上就存在的不是你在BIOS里关闭了NUMA差异就消除了，我更愿意认为在BIOS里关掉NUMA只是掩耳盗铃。**

以上理论告诉我们：**也就是在开启NUMA和 zone\_reclaim\_mode 默认在内存不够的如果去其它NUMA上分配内存，比关闭NUMA要快很多而没有任何害处。**

#### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#UMA%E5%92%8CNUMA%E5%AF%B9%E6%AF%94 "UMA和NUMA对比")UMA和NUMA对比

The SMP/UMA architecture

[![img](https://plantegg.github.io/images/951413iMgBlog/uma-architecture.png)](https://plantegg.github.io/images/951413iMgBlog/uma-architecture.png)

The NUMA architecture

[![img](https://plantegg.github.io/images/951413iMgBlog/numa-architecture.png)](https://plantegg.github.io/images/951413iMgBlog/numa-architecture.png)

Modern multiprocessor systems mix these basic architectures as seen in the following diagram:

[![img](https://plantegg.github.io/images/951413iMgBlog/39354-figure-3-184398.jpg)](https://plantegg.github.io/images/951413iMgBlog/39354-figure-3-184398.jpg)

In this complex hierarchical scheme, processors are grouped by their physical location on one or the other multi-core CPU package or “node.” Processors within a node share access to memory modules as per the UMA shared memory architecture. At the same time, they may also access memory from the remote node using a shared interconnect, but with slower performance as per the NUMA shared memory architecture.

[![03-05-Broadwell_HCC_Architecture](https://plantegg.github.io/images/951413iMgBlog/03-05-Broadwell_HCC_Architecture.svg)](https://plantegg.github.io/images/951413iMgBlog/03-05-Broadwell_HCC_Architecture.svg)

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E5%AF%B9%E6%AF%94%E6%B5%8B%E8%AF%95Intel-NUMA-%E6%80%A7%E8%83%BD "对比测试Intel NUMA 性能")对比测试Intel NUMA 性能

对如下Intel CPU进行一些测试，在开启NUMA的情况下

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p></pre></td><td><pre><p>#lscpu</p><p>Architecture:          x86_64</p><p>CPU op-mode(s):        32-bit, 64-bit</p><p>Byte Order:            Little Endian</p><p>CPU(s):                64</p><p>On-line CPU(s) list:   0-63</p><p>Thread(s) per core:    2</p><p>Core(s) per socket:    16</p><p>Socket(s):             2</p><p>NUMA node(s):          2</p><p>Vendor ID:             GenuineIntel</p><p>CPU family:            6</p><p>Model:                 79</p><p>Model name:            Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz</p><p>Stepping:              1</p><p>CPU MHz:               2500.000</p><p>CPU max MHz:           3000.0000</p><p>CPU min MHz:           1200.0000</p><p>BogoMIPS:              5000.06</p><p>Virtualization:        VT-x</p><p>L1d cache:             32K</p><p>L1i cache:             32K</p><p>L2 cache:              256K</p><p>L3 cache:              40960K</p><p>NUMA node0 CPU(s):     0-15,32-47</p><p>NUMA node1 CPU(s):     16-31,48-63</p><p>Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 ds_cpl vmx smx est tm2 ssse3 fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch ida arat epb invpcid_single pln pts dtherm spec_ctrl ibpb_support tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm cqm rdt rdseed adx smap xsaveopt cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local cat_l3</p><p>#numastat</p><p>                           node0           node1</p><p>numa_hit               129600200        60501102</p><p>numa_miss                      0               0</p><p>numa_foreign                   0               0</p><p>interleave_hit            108648          108429</p><p>local_node             129576548        60395061</p><p>other_node                 23652          106041</p></pre></td></tr></tbody></table>

我在这个64core的物理机上运行一个MySQL 实例，先将MySQL进程绑定在0-63core，0-31core，以及0-15,32-47上

用sysbench对一亿条记录跑点查，数据都加载到内存中了：

-   绑0-63core qps 不到8万，总cpu跑到5000%，降低并发的话qps能到11万；
-   如果绑0-31core qps 12万，总cpu跑到3200%，IPC 0.29；
-   如果绑同一个numa下的32core，qps飙到27万，总CPU跑到3200% IPC: 0.42；
-   绑0-15个物理core，qps能到17万，绑32-47也是一样的效果；

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620954918277-c669bd74-df58-4d69-8185-a93f37046972.png)](https://plantegg.github.io/images/951413iMgBlog/1620954918277-c669bd74-df58-4d69-8185-a93f37046972.png)

从这个数据看起来**即使Intel在只有两个NUMA的情况下跨性能差异也有2倍，可见正确的绑核方法收益巨大，尤其是在刷榜的情况下**， NUMA更多性能差异应该会更大。

说明前面的理论是正确的。

来看看不通绑核情况下node之间的带宽利用情况：

[![image-20210525151537507](https://plantegg.github.io/images/951413iMgBlog/image-20210525151537507.png)](https://plantegg.github.io/images/951413iMgBlog/image-20210525151537507.png)

[![image-20210525151622425](https://plantegg.github.io/images/951413iMgBlog/image-20210525151622425.png)](https://plantegg.github.io/images/951413iMgBlog/image-20210525151622425.png)

实际在不开NUMA的同样CPU上，进行以上各种绑核测试，测试结果也完全一样。

如果比较读写混合场景的话肯定会因为写锁导致CPU跑起来，最终的性能差异也不会这么大，但是绑在同一个NUMA下的性能肯定要好，IPC也会高一些。具体好多少取决于锁的竞争程度。

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9B%86%E5%9B%A2%E5%86%85%E5%A4%96%E6%89%80%E6%9C%89%E7%89%A9%E7%90%86%E6%9C%BA%E9%83%BD%E6%8A%8ANUMA%E5%85%B3%E6%8E%89%E4%BA%86%E5%91%A2%EF%BC%9F "为什么集团内外所有物理机都把NUMA关掉了呢？")为什么集团内外所有物理机都把NUMA关掉了呢？

10年前几乎所有的运维都会多多少少被NUMA坑害过，让我们看看究竟有多少种在NUMA上栽的方式：

-   [MySQL – The MySQL “swap insanity” problem and the effects of the NUMA architecture](http://blog.jcole.us/2010/09/28/mysql-swap-insanity-and-the-numa-architecture/)
-   [PostgreSQL – PostgreSQL, NUMA and zone reclaim mode on linux](http://frosty-postgres.blogspot.com/2012/08/postgresql-numa-and-zone-reclaim-mode.html)
-   [Oracle – Non-Uniform Memory Access (NUMA) architecture with Oracle database by examples](http://blog.yannickjaquier.com/hpux/non-uniform-memory-access-numa-architecture-with-oracle-database-by-examples.html)
-   [Java – Optimizing Linux Memory Management for Low-latency / High-throughput Databases](http://engineering.linkedin.com/performance/optimizing-linux-memory-management-low-latency-high-throughput-databases)

最有名的是这篇 [MySQL – The MySQL “swap insanity” problem and the effects of the NUMA architecture](http://blog.jcole.us/2010/09/28/mysql-swap-insanity-and-the-numa-architecture/)

我总结下这篇2010年的文章说的是啥：

-   如果本NUMA内存不够的时候，Linux会优先回收PageCache内存，即使其它NUMA还有内存
-   回收PageCache经常会造成系统卡顿，这个卡顿不能接受

所以文章给出的解决方案就是（三选一）：

-   关掉NUMA
-   或者启动MySQL的时候指定不分NUMA,比如：/usr/bin/numactl –interleave all $cmd
-   或者启动MySQL的时候先回收所有PageCache

我想这就是这么多人在上面栽了跟头，所以干脆一不做二不休干脆关了NUMA 一了百了。

但真的NUMA有这么糟糕？或者说Linux Kernel有这么笨，默认优先去回收PageCache吗？

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#Linux-Kernel%E5%AF%B9NUMA%E5%86%85%E5%AD%98%E7%9A%84%E4%BD%BF%E7%94%A8 "Linux Kernel对NUMA内存的使用")Linux Kernel对NUMA内存的使用

实际我们使用NUMA的时候期望是：优先使用本NUMA上的内存，如果本NUMA不够了不要优先回收PageCache而是优先使用其它NUMA上的内存。

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#zone-reclaim-mode "zone_reclaim_mode")zone\_reclaim\_mode

事实上Linux识别到NUMA架构后，默认的内存分配方案就是：优先尝试在请求线程当前所处的CPU的Local内存上分配空间。**如果local内存不足，优先淘汰local内存中无用的Page（Inactive，Unmapped）**。然后才到其它NUMA上分配内存。

intel 芯片跨node延迟远低于其他家，所以跨node性能损耗不大

zone\_reclaim\_mode，它用来管理当一个内存区域(zone)内部的内存耗尽时，是从其内部进行内存回收还是可以从其他zone进行回收的选项：

zone\_reclaim\_mode:

> Zone\_reclaim\_mode allows someone to set more or less aggressive approaches to  
> reclaim memory when a zone runs out of memory. If it is set to zero then no  
> zone reclaim occurs. Allocations will be satisfied from other zones / nodes  
> in the system.

zone\_reclaim\_mode的四个参数值的意义分别是：

0 = Allocate from all nodes before reclaiming memory  
1 = Reclaim memory from local node vs allocating from next node  
2 = Zone reclaim writes dirty pages out  
4 = Zone reclaim swaps pages

<table><tbody><tr><td><pre><p>1</p><p>2</p></pre></td><td><pre><p># cat /proc/sys/vm/zone_reclaim_mode</p><p>0</p></pre></td></tr></tbody></table>

我查了2.6.32以及4.19.91内核的机器 zone\_reclaim\_mode 都是默认0 ，也就是kernel会：优先使用本NUMA上的内存，如果本NUMA不够了不要优先回收PageCache而是优先使用其它NUMA上的内存。这也是我们想要的

Kernel文档也告诉大家默认就是0，但是为什么会出现优先回收了PageCache呢？

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E6%9F%A5%E7%9C%8Bkernel%E6%8F%90%E4%BA%A4%E8%AE%B0%E5%BD%95 "查看kernel提交记录")查看kernel提交记录

[github kernel commit](https://github.com/torvalds/linux/commit/4f9b16a64753d0bb607454347036dc997fd03b82)

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620956491058-09a1ebc6-c248-41db-9def-67b4f489c4f4.png)](https://plantegg.github.io/images/951413iMgBlog/1620956491058-09a1ebc6-c248-41db-9def-67b4f489c4f4.png)

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620956524069-85ec2c06-ff55-48e9-8c26-96e738456ed4.png)](https://plantegg.github.io/images/951413iMgBlog/1620956524069-85ec2c06-ff55-48e9-8c26-96e738456ed4.png)

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620956551990-6e376a3d-de40-4180-a05b-b21a9cbf33bc.png)](https://plantegg.github.io/images/951413iMgBlog/1620956551990-6e376a3d-de40-4180-a05b-b21a9cbf33bc.png)

关键是上图红框中的代码，node distance比较大（也就是开启了NUMA的话），强制将 zone\_reclaim\_mode设为1，这是2014年提交的代码，将这个强制设为1的逻辑去掉了。

这也就是为什么之前大佬们碰到NUMA问题后尝试修改 zone\_reclaim\_mode 没有效果，**也就是2014年前只要开启了NUMA就强制线回收PageCache，即使设置zone\_reclaim\_mode也没有意义，真是个可怕的Bug。**

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E9%AA%8C%E8%AF%81%E4%B8%80%E4%B8%8Bzone-reclaim-mode-0%E6%98%AF%E7%94%9F%E6%95%88%E7%9A%84 "验证一下zone_reclaim_mode 0是生效的")验证一下zone\_reclaim\_mode 0是生效的

内核版本：3.10.0-327.ali2017.alios7.x86\_64

#### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E6%B5%8B%E8%AF%95%E6%96%B9%E6%B3%95 "测试方法")[测试方法](https://github.com/torvalds/linux/commit/e02dc017c3032dcdce1b993af0db135462e1b4b7)

先将一个160G的文件加载到内存里，然后再用代码分配64G的内存出来使用。  
单个NUMA node的内存为256G，本身用掉了60G，加上这次的160G的PageCache，和之前的一些其他PageCache,总的 PageCache用了179G，那么这个node总内存还剩256G-60G-179G，

如果这个时候再分配64G内存的话，本node肯定不够了，我们来看在 zone\_reclaim\_mode=0 的时候是优先回收PageCache还是分配了到另外一个NUMA node(这个NUMA node 有240G以上的内存空闲）

#### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E6%B5%8B%E8%AF%95%E8%BF%87%E7%A8%8B "测试过程")测试过程

分配64G内存

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p></pre></td><td><pre><p>#taskset -c 0 ./alloc 64</p><p>To allocate 64GB memory</p><p>Used time: 39 seconds</p></pre></td></tr></tbody></table>

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620966121309-a264fd7f-fe50-4fc6-940f-4cb603ec7874.png)](https://plantegg.github.io/images/951413iMgBlog/1620966121309-a264fd7f-fe50-4fc6-940f-4cb603ec7874.png)

从如上截图来看，再分配64G内存的时候即使node0不够了也没有回收node0上的PageCache，而是将内存跨NUMA分配到了node1上，符合预期！

释放这64G内存后，如下图可以看到node0回收了25G，剩下的39G都是在node1上：  
[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620967573650-b8400c2f-7b48-4502-b7d5-6c050e557126.png)](https://plantegg.github.io/images/951413iMgBlog/1620967573650-b8400c2f-7b48-4502-b7d5-6c050e557126.png)

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E5%B0%86-proc-sys-vm-zone-reclaim-mode-%E6%94%B9%E6%88%90-1-%E7%BB%A7%E7%BB%AD%E5%90%8C%E6%A0%B7%E7%9A%84%E6%B5%8B%E8%AF%95 "将 /proc/sys/vm/zone_reclaim_mode 改成 1 继续同样的测试")将 /proc/sys/vm/zone\_reclaim\_mode 改成 1 继续同样的测试

可以看到zone\_reclaim\_mode 改成 1，node0内存不够了也没有分配node1上的内存，而是从PageCache回收了40G内存，整个分配64G内存的过程也比不回收PageCache慢了12秒，这12秒就是额外的卡顿

[![undefined](https://plantegg.github.io/images/951413iMgBlog/1620977108922-a2f67827-cf00-43a0-bba1-4ba105a33201.png)](https://plantegg.github.io/images/951413iMgBlog/1620977108922-a2f67827-cf00-43a0-bba1-4ba105a33201.png)

测试结论：**从这个测试可以看到NUMA 在内存使用上不会优先回收 PageCache 了**

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#innodb-numa-interleave "innodb_numa_interleave")innodb\_numa\_interleave

从5.7开始，mysql增加了对NUMA的无感知：[innodb\_numa\_interleave](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_numa_interleave)，也就是在开了NUMA的机器上，使用内错交错来分配内存，相当于使用上关掉 NUMA

> For the [`innodb_numa_interleave`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_numa_interleave) option to be available, MySQL must be compiled on a NUMA-enabled Linux system.

当开启了 innodb\_numa\_interleave 的话在为innodb buffer pool分配内存的时候将 [NUMA memory policy](https://linux.die.net/man/2/set_mempolicy) 设置为 MPOL\_INTERLEAVE 分配完后再设置回 MPOL\_DEFAULT（OS默认内存分配行为，也就是zone\_reclaim\_mode指定的行为)。

innodb\_numa\_interleave参数是为innodb更精细化地分配innodb buffer pool 而增加的。很典型地innodb\_numa\_interleave为on只是更好地规避了前面所说的zone\_reclaim\_mode的kernel bug，**修复后这个参数没有意义了**。

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#AUTOMATIC-NUMA-BALANCING "AUTOMATIC NUMA BALANCING")[AUTOMATIC NUMA BALANCING](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_tuning_and_optimization_guide/sect-virtualization_tuning_optimization_guide-numa-auto_numa_balancing)

RedHat 7默认会自动让内存或者进程就近迁移，让内存和CPU距离更近以达到最好的效果

> Automatic NUMA balancing improves the performance of applications running on NUMA hardware systems. It is enabled by default on Red Hat Enterprise Linux 7 systems.
> 
> An application will generally perform best when the threads of its processes are accessing memory on the same NUMA node as the threads are scheduled. Automatic NUMA balancing moves tasks (which can be threads or processes) closer to the memory they are accessing. It also moves application data to memory closer to the tasks that reference it. This is all done automatically by the kernel when automatic NUMA balancing is active.

对应参数

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>cat /proc/sys/kernel/numa_balancing shows 1</p></pre></td></tr></tbody></table>

### [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E7%9B%91%E6%8E%A7 "监控")监控

查找相应的内存和调度器事件

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p></pre></td><td><pre><p><span>#</span><span>perf <span>stat</span> -e <span>sched</span>:sched_stick_numa,<span>sched</span>:sched_move_numa,<span>sched</span>:sched_swap_numa,migrate:mm_migrate_pages,minor-faults -p 7191</span></p><p> Performance counter stats for process id '7191':</p><p>                 0      sched:sched_stick_numa                                        (100.00%)</p><p>                 1      sched:sched_move_numa                                         (100.00%)</p><p>                 0      sched:sched_swap_numa</p><p>                 0      migrate:mm_migrate_pages</p><p>               286      minor-faults</p><p><span>#</span><span> perf <span>stat</span> -e <span>sched</span>:sched_stick_numa,<span>sched</span>:sched_move_numa,<span>sched</span>:sched_swap_numa,migrate:mm_migrate_pages,minor-faults -p PID</span></p><p>...</p><p>                 1      sched:sched_stick_numa</p><p>                 3      sched:sched_move_numa</p><p>                41      sched:sched_swap_numa</p><p>             5,239      migrate:mm_migrate_pages</p><p>            50,161      minor-faults</p><p>#<span>perf <span>stat</span> -e <span>sched</span>:sched_stick_numa,<span>sched</span>:sched_move_numa,<span>sched</span>:sched_swap_numa,migrate:mm_migrate_pages,minor-faults -p 676322</span></p><p> Performance counter stats for process id '676322':</p><p>                 0      sched:sched_stick_numa</p><p>                16      sched:sched_move_numa</p><p>                 0      sched:sched_swap_numa</p><p>                24      migrate:mm_migrate_pages</p><p>             2,079      minor-faults</p></pre></td></tr></tbody></table>

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E6%80%BB%E7%BB%93 "总结")总结

-   放弃对NUMA的偏见吧，优先回收 PageCache 这个Bug早已修复了
-   按NUMA绑定core收益巨大，即使只有两个NUMA的intel芯片，也有一倍以上的性能提升，在飞腾等其他芯片上收益更大
-   没必要自欺欺人关掉NUMA了
-   RDS这样独占物理机的服务可以做到按NUMA来绑定core，收益可观
-   ECS售卖如果能够精确地按NUMA绑核的话性能，超卖比能高很多
-   在刷tpcc数据的时候更应该开NUMA和正确绑核

我个人一直对集团所有机器默认关闭NUMA耿耿于怀，因为定制的物理机（BIOS也是定制的）BIOS默认就是关闭NUMA的，装机还得一台台手工打开（跪了，几十万台啊），算是理清了来龙去脉。因为一个kernel的bug让大家对NUMA一直有偏见，即使14年已经修复了，大家还是以讹传讹，没必要。

关于cpu为什么高但是没有产出的原因是因为CPU流水线长期stall，导致很低的IPC，所以性能自然上不去，可以看[这篇文章](http://www.brendangregg.com/blog/2017-05-09/cpu-utilization-is-wrong.html)

其他同学测试的结论：

-   Hadoop离线作业在 Intel(R) Xeon(R) Platinum 8163 CPU @ 2.50GHz 24 cores/socket \* 2, Turbo Off 下打开NUMA后性能提升8%

一些其它不好解释的现象：

1.  增加少量跨NUMA 的core进来时能增加QPS的，但是随着跨NUMA core越来越多（总core也越来越多）QPS反而会达到一个峰值后下降—效率低的core多了，抢走任务，执行得慢
2.  压12-19和8-15同样8core，不跨NUMA的8-15性能只好5%左右(87873 VS 92801) — 难以解释
3.  由1、2所知在测试少量core的时候跨NUMA性能下降体现不出来
4.  在压0-31core的时候，如果运行 perf这个时候QPS反而会增加（13万上升到15万）— 抢走了一些CPU资源，让某个地方竞争反而减小了
5.  综上在我个人理解是core越多的时候UPI压力到了瓶颈，才会出现加core性能反而下降

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0 "系列文章")系列文章

[CPU的制造和概念](https://plantegg.github.io/2021/06/01/CPU%E7%9A%84%E5%88%B6%E9%80%A0%E5%92%8C%E6%A6%82%E5%BF%B5/)

[CPU 性能和Cache Line](https://plantegg.github.io/2021/05/16/CPU%20Cache%20Line%20%E5%92%8C%E6%80%A7%E8%83%BD/)

[Perf IPC以及CPU性能](https://plantegg.github.io/2021/05/16/Perf%20IPC%E4%BB%A5%E5%8F%8ACPU%E5%88%A9%E7%94%A8%E7%8E%87/)

[Intel、海光、鲲鹏920、飞腾2500 CPU性能对比](https://plantegg.github.io/2021/06/18/%E5%87%A0%E6%AC%BECPU%E6%80%A7%E8%83%BD%E5%AF%B9%E6%AF%94/)

[飞腾ARM芯片(FT2500)的性能测试](https://plantegg.github.io/2021/05/15/%E9%A3%9E%E8%85%BEARM%E8%8A%AF%E7%89%87(FT2500)的性能测试/)

[十年后数据库还是不敢拥抱NUMA？](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/)

[一次海光物理机资源竞争压测的记录](https://plantegg.github.io/2021/03/07/%E4%B8%80%E6%AC%A1%E6%B5%B7%E5%85%89%E7%89%A9%E7%90%86%E6%9C%BA%E8%B5%84%E6%BA%90%E7%AB%9E%E4%BA%89%E5%8E%8B%E6%B5%8B%E7%9A%84%E8%AE%B0%E5%BD%95/)

[Intel PAUSE指令变化是如何影响自旋锁以及MySQL的性能的](https://plantegg.github.io/2019/12/16/Intel%20PAUSE%E6%8C%87%E4%BB%A4%E5%8F%98%E5%8C%96%E6%98%AF%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8D%E8%87%AA%E6%97%8B%E9%94%81%E4%BB%A5%E5%8F%8AMySQL%E7%9A%84%E6%80%A7%E8%83%BD%E7%9A%84/)

## [](https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99 "参考资料")参考资料

[https://www.redhat.com/files/summit/session-assets/2018/Performance-analysis-and-tuning-of-Red-Hat-Enterprise-Linux-Part-1.pdf](https://www.redhat.com/files/summit/session-assets/2018/Performance-analysis-and-tuning-of-Red-Hat-Enterprise-Linux-Part-1.pdf)

[https://informixdba.wordpress.com/2015/10/16/zone-reclaim-mode/](https://informixdba.wordpress.com/2015/10/16/zone-reclaim-mode/)

[https://queue.acm.org/detail.cfm?id=2513149](https://queue.acm.org/detail.cfm?id=2513149)

[NUMA DEEP DIVE PART 1: FROM UMA TO NUMA](https://frankdenneman.nl/2016/07/07/numa-deep-dive-part-1-uma-numa/) 这是一个系列，都很干货，值得推荐

[https://15721.courses.cs.cmu.edu/spring2016/papers/p743-leis.pdf](https://15721.courses.cs.cmu.edu/spring2016/papers/p743-leis.pdf) Morsel-Driven Parallelism: A NUMA-Aware Query Evaluation Framework for the Many-Core Age 论文给出了很多numa-aware下的bandwidth、latency数据，以及对THC-H的影响