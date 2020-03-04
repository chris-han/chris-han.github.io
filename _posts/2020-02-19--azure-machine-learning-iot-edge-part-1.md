---
layout: post
published: true
title: 微软Azure Machine Learning 与IoT Edge实现智能红外体温检测
tags:
  - azure
  - iot
  - machine learning
  - power platform
  - flow
comments: true
subtitle: Part 1
---
抗击NCP新冠病毒这个事儿再往后面临就是实体经济的恢复生产、学校开学的常态化安全检查问题。非接触式红外体温检测不仅应该用于机场码头，对企事业单位的入口也是必备手段。这个当然不仅是技术问题，还有整套管理流程问题。Azure云平台上的服务太多了，跟IoT相关的[参考这里](https://azurecharts.com/stories/?s=7,52)。中国Mooncake和其他国家的略有不同，在选型的时候就有很多选择，对架构师来说选什么不选什么为什么这么选就要看实际情况了。在加上[IoT Edge](https://github.com/Azure/iotedge)本身就是很新的技术，配套工具正在完善中。我就是Visutal Stduio和VS Code混用，而且我发现VS code在IoT这块的支持居然相对更完善！微软真是快Open first了。这个系列我先把技术坑儿分享一下，以后再把运营实践中的lesson learned分享给大家。

![架构图1]({{site.baseurl}}/img/figure1-quick-n-dirty.png)
简化版的方案1(图1)，利用容器技术在边缘设备上直接部署azure function，通过对体温监测阈值的简单规则设置，触发报警和上传云端报告。[通过Azure function 连接Power BI的流式数据集](https://docs.microsoft.com/en-us/samples/azure-samples/functions-js-iot-hub-processing/processing-data-from-iot-hub-with-azure-functions/)可以做到近实时地展示警报(图2)。汇总数据，包括图片都存储在Azure Blob Storeage里留档备查。非实时报表可以通过Power BI的blob storage connector 直接读取azure blob storage里的累积数据。
![PowerBI 实时报表]({{site.baseurl}}/img/PBI-rt.png)

现场人员通知可以通过Azure Logic App推送到手机上的Microsoft Teams。这是最少代码、快速低成本搭建的方案。可以作为非常时期的快速部署的辅助报警手段。但响应速度和准确性肯定不行。设备误报率会比较高。原因很多。比如人体在一天中的不同时期体温不同，男女体温不同，女性生理期体温不同，环境影响也没考虑进来等（参考**[The daily, weekly, and seasonal cycles of body temperature analyzed at large scale ](https://tandf.figshare.com/articles/The_daily_weekly_and_seasonal_cycles_of_body_temperature_analyzed_at_large_scale/9872681/1)**）。所以有了通过机器学习的方式在线学习与改进边缘智能的方案2.


![架构图2]({{site.baseurl}}/img/figure2-full-solution-architecture.png)

方案2(图3)是一个闭合运营系统。首先通过Microsoft Power Platform提供现场工作人员移动警报推送，提示体温异常人员通过检测点。Push notification是通过[Microsoft Flow](https://flow.microsoft.com/zh-cn/)(现在改名叫Power Automate)实现。推送的Notification又可以打开移动设备上的[Power App](https://powerapps.microsoft.com/zh-cn/build-powerapps/)（补一张图4）。
![Microsoft Power Platform构建无代码移动应用]({{site.baseurl}}/img/powerPlatform.jpg)


现场人员用手持测温设备进行二次测温后可以输入到Cosmo DB里作为追溯留档。同时也可以作为样本，通过Azure Machine Learning 来改进报警模型。这个方案里面还可以集成云主播控制台来实时汇总分布各地的实时视频画面。必要的话，甚至可以通过移动skye客户端、无人机等实现广播级的多路现场-导播台视频连麦，对现场工作进行指导或作为即时新闻采访通道。这部分由[微软技术中心MTC](https://www.microsoft.com/en-us/mtc)开发的云导播方案可以实现单机32路视频会议直播，并可以通过Azure VMSet线性扩展，我再另文展开。所有服务资源都在云端，根据工作量弹性缩放，实现了平时与战时/处突的经济性vs性能的平衡。

整个机器学习模型的DevOps可以是Vs Code 开发、Azure Repo/Github 做代码管理、Azure Container Registry 发布训练好的模型 inference service image到边缘。Azure Cosmo db同时也可以支持分布式多点、近实时的报表查询。便于汇总多路边缘设备数据进行园区、甚至更大区域的动态趋势监测、预警。Azure Machine Learning 本身提供了大量内置算法，Azure ML Studio可以通过拖拽的方式建模训练、发布或下载训练好的模型。最新的[AutoML](https://docs.microsoft.com/en-us/azure/machine-learning/concept-automated-ml)功能甚至把你自己调参的工作都省了，指定数据源，它直接自动整理、清洗数据、选择算法、调参、性能比较之后呈现给你所有可能的模型供你下载。堪称傻瓜机里的战斗机。

再好的算法也有场景局限。所以最好的办法是把数据公开给大家，一起改进算法。

关于硬件，理论上任何测体温的红外摄像头只要有SDK都可以集成。我实验用的双光镜头,测温30~45度。图5是我对市售镜头的统计，售价都是6万以上。我的目标是可以用更便宜的硬件配合算法的提升来让更多的单位用起来。
![市售红外镜头]({{site.baseurl}}/img/ir cams.png)

Edge PC用的Intel NUC 7i5DNH6EL 跑Windows 10 IoT Enterprise LTSC. Azure IoT Edge的架构是可以实现不同模型的串联或并联处理。所以跑多少模型完全由硬件能力决定。以后有时间再聊一下硬件加速edge inference。另外这个方案基本上是工业互联网的架构。装上OPC UA模块就可以和其他工业设备互通互联。福兮祸兮！制造业受疫情影响，也许正是加速技术升级的机会。微软在[开源工业互联网](https://github.com/Azure/Industrial-IoT)的这部分我也另篇分享吧。
