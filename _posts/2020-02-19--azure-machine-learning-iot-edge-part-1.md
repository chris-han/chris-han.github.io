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
抗击NCP新冠病毒这个事儿再往后面临就是实体经济的恢复生产、学校开学的常态化安全检查问题。非接触式红外体温检测不仅应该用于机场码头，对企事业单位的入口也是必备手段。这个当然不仅是技术问题，还有整套管理流程问题。这个系列我先把技术探索分享一下，以后再把实践中的lesson learned分享给大家。

简化版的方案1(图1)，利用容器技术在边缘设备上直接部署azure function，通过对体温监测阈值的简单规则设置，触发报警和上传云端报告。[通过Azure function 连接Power BI的流式数据集](https://docs.microsoft.com/en-us/samples/azure-samples/functions-js-iot-hub-processing/processing-data-from-iot-hub-with-azure-functions/)可以做到近实时地展示警报。汇总数据，包括图片都存储在Azure Blob Storeage里留档备查。非实时报表可以通过Power BI的blob storage connector 直接读取azure blob storage里的累积数据。现场人员通知可以通过Azure Logic App推送到手机上的Microsoft Teams。这是最少代码、快速低成本搭建的方案。可以作为非常时期的快速部署的辅助报警手段。但响应速度和准确性肯定不行。设备误报率会比较高。原因很多。比如人体在一天中的不同时期体温不同，男女体温不同，女性生理期体温不同，环境影响也没考虑进来等（参考**[The daily, weekly, and seasonal cycles of body temperature analyzed at large scale ](https://tandf.figshare.com/articles/The_daily_weekly_and_seasonal_cycles_of_body_temperature_analyzed_at_large_scale/9872681/1)**）。所以有了通过机器学习的方式在线学习与改进边缘智能的方案2.

![架构图1]({{site.baseurl}}/img/figure1-quick-n-dirty.png)


方案2（图2）是一个闭合运营系统。首先通过Microsoft Power Platform提供现场工作人员移动警报推送，提示体温异常人员通过检测点。Push notification是通过Microsoft Flow实现。推送的Notification又可以打开移动设备上的Power App（图3）。现场人员用手持测温设备进行二次测温后可以输入到Cosmo DB里作为追溯留档。同时也可以作为样本，通过Azure Machine Learning 来改进报警模型。这个方案里面还可以集成云主播控制台来实时汇总分布各地的实时视频画面。必要的话，甚至可以通过移动skye客户端实现广播级的多路现场-导播台视频连麦，对现场工作进行指导或作为即时新闻采访通道。所有服务资源都在云端动态扩展，实现了平时与战时/处突的经济性vs性能的平衡。

![架构图2]({{site.baseurl}}/img/figure2-full-architecture.png)

![Microsoft Power Platform构建无代码移动应用]({{site.baseurl}}/img/powerPlatform.png)

整个机器学习模型的DevOps可以是Vs Code 开发、Azure Repo/Github 做代码管理、Azure Container Registry 发布训练好的模型 inference service image到边缘。Azure Cosmo db同时也可以支持分布式多点、近实时的报表查询。便于汇总多路边缘设备数据进行园区、甚至更大区域的动态趋势监测、预警。Azure Machine Learning 本身提供了大量内置算法，Azure ML Studio可以通过拖拽的方式建模训练、发布或下载训练好的模型。最新的[AutoML](https://docs.microsoft.com/en-us/azure/machine-learning/concept-automated-ml)功能甚至把你自己调参的工作都省了，指定数据源，它直接自动整理、清洗数据、选择算法、调参、性能比较之后呈现给你所有可能的模型供你下载。堪称傻瓜机里的战斗机。

再好的算法也有场景局限。所以最好的办法是把数据公开给大家，一起改进算法。
