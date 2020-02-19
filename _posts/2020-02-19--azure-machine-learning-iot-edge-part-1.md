---
layout: post
published: true
title: 微软Azure Machine Learning 与IoT Edge实现智能红外体温检测 Part 1
tags:
  - azure
  - iot
  - ML
comments: true
---
抗击NCP新冠病毒这个事儿再往后面临的大问题就是实体经济的恢复生产问题。大量人员复工给企业带来的首要问题就是workplace safety。非接触式红外体温检测不仅应该用于机场码头，对企事业单位的入口也是必备手段。这个当然不仅是技术问题，还有整套管理流程问题。这个系列我先把技术探索分享一下，后面再把实践中的lesson learned分享给大家。

简化版的方案1，利用容器技术在边缘设备上直接部署azure function，通过对体温监测阈值的简单规则设置，触发报警和上传云端报告。[通过Azure function 连接Power BI的流式数据集](https://docs.microsoft.com/en-us/samples/azure-samples/functions-js-iot-hub-processing/processing-data-from-iot-hub-with-azure-functions/)可以做到近实时地展示警报。汇总数据，包括图片都存储在Azure Blob Storeage里留档备查。非实时报表可以通过Power BI的blob storage connector 直接读取azure blob storage里的累积数据。这是最佳成本方案。可以作为非常时期的快速部署的辅助报警手段。但准确性肯定不行，误报率会比较高。原因很多。比如人体在一天中的不同时期体温不同，男女体温不同，女性生理期体温不同，环境影响也没考虑进来等。所以有了通过机器学习的方式在线学习与改进边缘智能的方案2.

![架构图1]({{site.baseurl}}/img/figure1.png)


方案2就是加入了Azure Machine Learning Service。 Vs Code 开发、Azure Container Registry 发布训练好的模型 inference service image到边缘。每次报警提示现场人员二次检验并通过Azure 上的一个网页界面进行回报。这个数据会存在blob storage作为监督学习的正例，同时存于azure cosmo db是为了支持多人、近实时的报表。便于汇总多路边缘设备数据进行园区、甚至更大区域的动态趋势监测、预警。

![架构图2]({{site.baseurl}}/img/figure2.png)


这个方案里面还可以集成Microsoft Dynamics Field Service——一个自动客服流程管理的SaaS服务。从边缘设备报警可以直接生成一个alert，alert可以自动或手动转换为case，case经过review可以再转换为工单、下发最近的医疗单位进行现场处置。做到事事有落实，事后有追踪。Dynamics Field Service已经内置集成了Azure IoT服务，所以基本上没有二次开发，全是配置实现。

Azure Machine Learning 本身提供了大量内置算法，Azure ML Studio可以通过拖拽的方式建模训练、发布或下载训练好的模型。最新的[AutoML](https://docs.microsoft.com/en-us/azure/machine-learning/concept-automated-ml)功能甚至把你自己调参的工作都省了，指定数据源，它直接自动整理、清洗数据、选择算法、调参、性能比较之后呈现给你所有可能的模型供你下载。堪称傻瓜机里的战斗机。

再好的算法也有场景局限。所以最好的办法是把数据公开给大家，一起改进算法。启动数据集在Google Dataset Search上发现一个，**[The daily, weekly, and seasonal cycles of body temperature analyzed at large scale ](https://tandf.figshare.com/articles/The_daily_weekly_and_seasonal_cycles_of_body_temperature_analyzed_at_large_scale/9872681/1)**