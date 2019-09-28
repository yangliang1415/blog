```
原文地址：
第一份工作总结
https://blog.csdn.net/yl1415/article/details/45157435

csdn好久不登录，账号密码都找不回了，于是copy一份备份。原文如下
```



部门：广告部门

时间：2014.06-2015.04

工作：DSP策略相关

 

# 业务背景相关

* RTB：实时竞价
* DSP：广告主服务平台，为广告主采买流量
* SSP：媒体服务平台，流量提供方
* AdExchange：连接SSP和DSP的中间平台
* DMP：数据管理平台，提供广告精准定向用户数据
* Cookie Mapping：cookie映射、匹配，表示adx和dsp用户
* DSP竞价流程：用户浏览网页，Adx向众多DSP发送竞价请求(reqlog)，DSP接收请求并参与竞价，发送竞价信息(send log)，如果出价最高，获得展示机会(pv log)，进一步用户会有点击(clk log)和转化行为(convert log)。



# DSP策略工作

* 概述：替广告主投放广告做决策，将广告在合适的时间展示给合适的人。替广告主衡量每次广告展示的价值，做出广告投放的最佳决策。
* 数据依赖：Req log；Send log；Pv log；Clk log；Convert log
* 策略组对外提供服务主要是这个接口：bidInfo Listidding（adList,bidFea），即根据bidFea从adList中挑选一个最合适的广告返回，并报价。这中间涉及到反作弊，点击率预估，竞价策略等模块。以下将重点介绍各个模块。
  * 反作弊：根据历史日志，统计各个维度(IP，adzone，url等)的一些指标（ctr，pv、clk times等）产生黑名单。竞价过程中放弃满足这些维度取值的竞价请求。
  * 点击率预估：这是DSP中最为核心的一环。根据历史日志建立LR模型，然后根据bidFea和adFea来预估广告被点击的可能性大小。其中涉及到的过程包括：日志清洗，拼接，解析，特征提取，模型训练。
  * 转化率预估： 根据clk、convert日志，自定义转化行为来调整正负样本比悬殊问题。类似点击率预估的流程，采用改进的NB模型建模。
  * 竞价策略：根据客户的不同要求(CPM、CPC、CPA)，有不同的竞价策略。
    重定向策略：及时为历史浏览用户展示历史行为相关广告。实时性要求高。



# DSP其他组相关工作

* 广告投放系统：广告主自助投放广告，设定一些定向。
* Cookie Mapping： Cookie映射、匹配。匹配Adx和DSP用户，为DSP提供用户数据做更好的决策。
* 广告引擎：位于策略组上游，接收Adx竞价请求，并从广告库中挑选合适广告（地域定向等）构成adList。



# 相关技术回顾及储备

* 语言方面： c++、java、python、shell
* 大数据处理： Hadoop、spark
* 实时：Storm、redis
* 机器学习： LR、SVM、NB等常用机器学习算法
* 最优化知识求解模型

