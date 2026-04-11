# 关于 智能红绿灯 的简单探索

> 原文：https://www.cnblogs.com/tansm/p/16339622.html

# 现状

一直奇怪国内，甚至号称科技前沿的深圳，并没有太多使用智能红绿灯，查阅到的都是“秀”智能红绿灯的，基本上没有看见真正老百姓拍摄的智能红绿灯的好的反馈，感觉目前国内的技术还不靠谱，毕竟我在深圳科技园附近开了几年的车，周边都是固定时间的十字路口。

倒是找到几个美国的智能红绿灯实际视频，还是比较靠谱的、实用的。比如这个 《[美国智能红绿灯（1）](https://v.qq.com/x/page/o03175k7c5d.html)》，《[美国智能红绿灯（2）](https://v.qq.com/x/page/j0317iqjz9h.html)》。

# 概念

言归正传，既然智能红绿灯，那么我肯定首先想到的是 AI，我的理解是需要

1、数据源头，传感器的采集，比如摄像头或雷达；

2、态势感知，即分析数据来源，对当前的状况形成 模型；

3、决策，即AI最终做出红绿灯的决策；

4、评估，对最终的结果进行评估，以便对决策的结果评判好坏，这个非常重要，比如车辆的等待时间，车道的通行平均速度，车辆的刹车频率，车辆和行人过路口的危险程度等；

# 开源实现

我按照 "Intelligent Traffic Light" 关键字，简单搜索了一下，

 

https://github.com/suessmann/intelligent_traffic_lights

应该是个人爱好的python + CNN-DQN作品，已经很久没有更新了，但适合简单了解概念，比如他用到了 城市交通模拟器 Simulation of Urban MObility ([SUMO](https://www.eclipse.org/sumo/))。

 

https://github.com/wingsweihua/IntelliLight

这个是更久的作品，看说明还作为学术论坛发表过，技术和上面的差不多。

 

https://github.com/quantumiracle/Reinforcement_Learning_for_Traffic_Light_Control

深度学习。

 

https://github.com/sunilkumarmaurya786693/Intelligence-traffic-monitoring-system

这个项目不仅仅是红绿灯项目，还包含自动车辆牌照的识别。

 

我倒是觉得如果AI能精确的是哪个车牌的车辆，那是不是在车辆进入路口前就能预测他去哪个方向。甚至我作为一个老司机来说，一定要在系统中加入 车辆 过红绿灯起步过慢、堵塞路口等行为的惩罚行为。
