---
layout: page
title: 博主简介
description: 生命不息，编程不止
keywords: Jianjian Du, 杜健健
comments: true
menu: 关于
permalink: /about/
---

Every journey begins with the first step.

Cease to struggle and you cease to live. 

### 简介:

# 杜健健:

- 南京航空航天大学硕士研究生
- 研究方向：计算机仿真
- skill：java、python、c

### 科技竞赛获奖（部分）:

- 中国研究生数学建模竞赛全国二等奖
- 挑战杯科技作品竞赛江苏省三等奖
- 中国大学生物联网大赛华东赛区一等奖，全国三等奖
- 中国机器人大赛暨Robocup公开赛I型机器人探险游全国特等奖
- “中兴捧月”算法竞赛75分
- “华为杯”网络技术精英挑战赛进入复赛，获得绿卡




### 爱好

编程，NBA。

相关技术问题可一起讨论。邮箱：jianrobot@126.com



## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

邮箱：jianrobot@126.com

## Skill Keywords

{% for category in site.data.skills %}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
