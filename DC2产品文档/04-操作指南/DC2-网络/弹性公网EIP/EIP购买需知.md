##可用区

EIP 作用于一个区域内，可与本区域内各个可用区的 DC2 绑定，购买 EIP 时，需要先选择区域

> 不同地域之间的云资源之间内网互不相通，选择靠近您客户的地域，可降低网络延迟，提高您客户的访问速度

##计费规则

EIP提供包年包月（预付费）和按时长（后付费）两种付费方式，您可以根据实际使用需要选择：

###1.包年包月

EIP提供包年包月（预付费）和按时长（后付费）两种付费方式，您可以根据实际使用需要选择：

包年包月的计费周期为：30天。费用相比按时长来说会便宜很多，如果长期使用建义选择包月的方式。包年包月资源到期前，您可以在官网进行续费继续包月使用。包月计费资源到期后如果不续费，将自动删除资源，资源到期删除后将立即被释放并不可恢复。您可以在系统中关闭此功能并修改包月到期的策略。


###2.按时长

按时长计费为用户提供了使用时长的灵活性，删除 EIP 后会立即停止计费。按时长计费也可以转换为包月计费方式。按时长付费EIP的公网带宽支持两种：按固定带宽计费，和按使用流量计费。



###3.按固定带宽

每个按固定带宽计费的EIP的总费用 = EIP资源占用费（EIP资源占用费 x 占用时长） + 带宽费（ 每天带宽费用 x 占用时长）

计费周期计费周期为1小时，账单周期也为1小时。在一个计费周期内，占用时长或绑定时间不足一天，按一小时计算。

您可以随时变更带宽上限。当一个计费周期（1小时）内，EIP发生过带宽变更（无论是变大还是变小），计费带宽值均取该计费周期内设置过的带宽值的最大值。

###4.按使用流量

每个按流量计费的EIP的总费用 = EIP资源占用费（EIP资源占用费 x 占用时长） + 流量费 （每小时流量单价 x 使用流量），使用流量指的是EIP的出方向流量累计值，入方向流量不参与计费（出方向指从滴滴数据中心流向互联网，反之是入方向）。

计费周期为1秒，账单周期为1小时，系统将精确到秒级收费。

修改带宽上限并不会影响单价。但是，我们建议您根据实际需求来设置带宽，以免因为程序错误或恶意访问导致产生大量计费流量。您也可以通过购买 DDoS高防 来防止EIP遭到恶意访问或攻击导致产生大量计费流量。

##带宽变配

EIP带宽支持升降配，升降配后的计费遵循以下逻辑：

###1.包年包月

付费方式为包年包月的EIP升降配时，当前的计费周期立即结束，变更后的配置作为新计费周期立即开始。当您在升配时，请确保账户中有充足的余额。如果需要降配，系统默认根据您降配的带宽价格，为您自动计算弥补相应时长，延长到期日。

###2.按量计费

付费方式为按量计费时，系统支持按秒级计费。原有配置计费会在变更成功后立即结束，新的配置计费在您变更成功后立即开始。


##欠费规则

###1.包年包月

到期预警：包年包月资源会在到期前7天、3天、1天的工作时间(8:00~22:00)以短信、邮件及站内信的方式向您推送到期预警，如果您的资源所属为团队，团队成员均会收到预警。

回收机制：包年包月资源到期前，您可以在官网进行续费继续包月使用。包月计费资源到期后如果不续费，将自动删除资源，资源到期删除后将立即被释放并不可恢复。您可以在系统中关闭此功能并修改包月到期的策略。


###2.按量计费

* 滴滴云会按照前一天的消费及账户余额去预测账户的欠费日期，在欠费前的3天、1天的工作时间(8:00~22:00)以短信、邮件及站内信的方式向您发送通知。

* 当您的账户发生欠费时，您的按时长的DC2将停机，同时EIP将被解除绑定同时滴滴云将向您发送停止服务的资源的详情通知，同时也会告知您具体的回收策略。

* 在您账户欠费72小时（3天）之内您的滴滴云的DC2可在充值后（充值后余额满足后付费资源创建条件）手动恢复。超过这个时间，滴滴云会将您的DC2数据以快照形式保存，同时DC2，EIP将被删除释放，且不可恢复。


##价格总览

滴滴云提供1~100M带宽规格的EIP资源，其中从1M带宽至6M带宽价格如下：

|1M带宽 |18元／月|0.03元/小时
| :--------| :-----| :---- |:----  |
|2M带宽 |36元／月|0.06元/小时
|3M带宽 |54元／月|0.09元/小时
|4M带宽 |72元／月|0.12元/小时
|5M带宽 |90元／月|0.15元/小时
|6M带宽 |170元／月|0.3元/小时

从6M~100M带宽价格计算方式如下:

|配置 |xM带宽（x>5）
| :--------| :-----| :---- 
|包月价格 |(x-5)*80+90 元/月 |
|按时长价格 |(x-5)*3.2+4 元/天|

##查看账单

1. 单机击控制台右上角账户名选择【账单】可以进入账单页面， 默认显示最近10条账单记录，单击【收支明细】可以查看所有的账单记录。

2. 单击账单后面的【详情】按钮可以进入账单的详情页面，如图所示。

