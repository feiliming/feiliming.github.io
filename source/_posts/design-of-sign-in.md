---
title: 签到实现方案设计
date: 2016-12-09 10:32:01
tags: 
- 签到
- 实现方案设计
categories: 实现方案设计
description: 签到实现方案设计。签到功能的目的是什么？怎么达到目的？目的是增加用户黏性，激励用户成就感，同时会使用积分、抽奖等方式奖励用户成就感。
---

> 元问题
> 签到功能的目的是什么？怎么达到目的？
> 目的是增加用户黏性，激励用户成就感，同时会使用积分、抽奖等方式奖励用户成就感。

## 需求

1.用户每天签到一次，不能重复
2.记录连续签到次数，可以有一个阈值，达到阈值后从0开始
3.签到没有连续，则从0开始
4.可以查看签到明细

ps：不用考虑积分奖励，那是积分系统的事

## 表结构

1.签到表
id int(11) 主键
person_id int(11) 用户ID
last_time datetime 上次签到日期
continuous_count int(11) 已经连续签到次数
total_count int(11) 签到总次数

2.签到明细表
id int(11) 主键
person_id int(11) 用户ID
sign_time datetime 签到日期

## 算法
1.接收用户ID
2.获取服务器当前日期
3.根据用户ID和服务器当前日期判断该用户今天是否签到过
```
if(数据存在){
    if(当前日期 > 上次签到日期){
        可签到
    }else if(当前日期 == 上次签到日期){
        return 今天已签，禁止再次签到
    }
}else{
    可签到
}
```
4.根据阈值进行签到
last_time = 当前日期
total_count++ 签到总次数加1
```
//数据存在update
if(已经连续签到次数 >= 阈值){
    continuous_count = 0 连续签到次数清零
}else if(连续){
    continuous_count++ 连续签到次数加1
}else{
    continuous_count = 0 连续签到次数清零
}
//数据不存在insert
continuous_count = 1 连续签到次数为1
```
5.判断是否连续
```
if(当前日期 == 上次签到日期 + 1){
    连续
}else{
    不连续
}
```
6.如果签到则插入明细

## 接口设计

1.根据用户ID获取当前日期签到情况
参数：`person_id`
返回值：
`is_sign: 1表示当前日期已签，0表示未签`
`continuous_count： 已经连续签到次数`
`total_count： 签到总次数`

2.根据用户ID签到
参数：`person_id`
返回值：
`success: true表示成功，同时返回以下参数，否则只返回false`
`is_sign: 1表示当前日期已签，0表示未签`
`continuous_count： 已经连续签到次数`
`total_count： 签到总次数`