---
layout: post
title: "Java 常用验证方法（commons-validator，hutool）"
date: "2020-08-29 19:00"
category: Java
tags: Java commons-validator hutool
author: jiangliheng
---
* content
{:toc}



# 背景

为了实现接口响应全量字段断言，开发断言表达式框架时，除了调研常用的断言框架之外，也调研了一些验证框架和方法（非```hibernate-validator```、```spring-validator```验证注解）。

简单学习下构建工具```Gradle```如何使用。

# commons-validator

> A common issue when receiving data either electronically or from user input is verifying the integrity of the data. This work is repetitive and becomes even more complicated when different sets of validation rules need to be applied to the same set of data based on locale. Error messages may also vary by locale. This package addresses some of these issues to speed development and maintenance of validation rules.


**Apache**开源的通用验证框架，目前最新版本**1.7**。


# hutool

> A set of tools that keep Java sweet.

**Hutool**是一个小而全的**Java**工具类库，通过静态方法封装，降低相关**API**的学习成本，提高工作效率，使**Java**拥有函数式语言般的优雅，让**Java**语言也可以“甜甜的”。

**Hutool** 是项目中 “util” 包友好的替代，它节省了开发人员对项目中公用类和公用工具方法的封装时间，使开发专注于业务，同时可以最大限度的避免封装不完善带来的 bug。

# 验证方法比较

仅仅从验证方法比较：
- ```commons-validator```除了通用验证方法外，还支持国际通用数字标准验证，比如：```IBAN (International Bank Account Number) ```、```ISSN（International Standard Serial Number) ```、```ISBN（International Standard Book Number)```等
- ```hutool（Validator）```通用验证方法与```commons-validator```基本一致，由于国人开源，验证方法较“中国”化些，比如：身份证、手机号、车牌号、邮政编码、社会统一信用代码、是否汉字等。

PS：```hutool```作者问题交流和合并 PR 那是极快的。

# 验证测试工程（基于Gradle）

**Gradle 配置文件**

```gradle
plugins {
    id 'java'
}

group 'com.jlh'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.testng:testng:6.14.3'
    compile 'commons-validator:commons-validator:1.7'
    compile 'cn.hutool:hutool-all:5.4.0'
}
```

**验证方法使用演示**

```java
package com.jlh.validator;

import cn.hutool.core.lang.Validator;
import cn.hutool.core.util.IdcardUtil;
import org.apache.commons.validator.GenericValidator;
import org.apache.commons.validator.routines.InetAddressValidator;
import org.testng.Assert;
import org.testng.annotations.Test;

import java.util.UUID;

/**
 * 验证框架测试
 *
 * @Author：jiangliheng
 * @Date：2020/8/28 13:31
 */
public class ValidatorTest {

    /**
     * apache commons validator 使用演示
     */
    @Test
    public void commonsValidatorTest() {
        // null 或 空断言
        Assert.assertTrue(GenericValidator.isBlankOrNull(""));
        Assert.assertTrue(GenericValidator.isBlankOrNull(null));
        // int，其他类型一样：byte,short，float，double，long
        Assert.assertTrue(GenericValidator.isInt("1"));
        // 日期
        Assert.assertTrue(GenericValidator.isDate("20200829", "yyyyMMdd",true));
        // int 在指定范围内，其他类型一样：byte,short，float，double，long
        Assert.assertTrue(GenericValidator.isInRange(1, 0,2));
        // int 最大最小，其他类型一样：float，double，long
        Assert.assertTrue(GenericValidator.minValue(1, 1));
        Assert.assertTrue(GenericValidator.maxValue(1, 1));
        // 字符串 最大最小长度
        Assert.assertTrue(GenericValidator.maxLength("daodaotest", 10));
        Assert.assertTrue(GenericValidator.minLength("daodaotest", 10));
        // 正则表达式
        Assert.assertTrue(GenericValidator.matchRegexp("daodaotest", "^d.*t$"));
        // 信用卡验证
        Assert.assertTrue(GenericValidator.isCreditCard("6227612145830440"));
        // url
        Assert.assertTrue(GenericValidator.isUrl("http://www.baidu.com"));
        // email
        Assert.assertTrue(GenericValidator.isEmail("dao@test.com"));
        // ip
        Assert.assertTrue(InetAddressValidator.getInstance().isValid("192.168.1.1"));
        Assert.assertTrue(InetAddressValidator.getInstance().isValid("CDCD:910A:2222:5498:8475:1111:3900:2020"));
    }

    /**
     * hutools validator 使用演示
     */
    @Test
    public void huTollsValidatorTest() {
        // null 空 布尔
        Assert.assertTrue(Validator.isNull(null));
        Assert.assertTrue(Validator.isNotNull("daodaotest"));
        Assert.assertTrue(Validator.isEmpty(""));
        Assert.assertTrue(Validator.isNotEmpty("daodaotest"));
        Assert.assertTrue(Validator.isTrue(true));
        Assert.assertTrue(Validator.isFalse(false));
        // 相等
        Assert.assertTrue(Validator.equal("daodaotest","daodaotest"));
        // 是否汉字，包含汉字
        Assert.assertTrue(Validator.hasChinese("daodaotest叨叨软件测试"));
        Assert.assertTrue(Validator.isChinese("叨叨软件测试"));
        // 是否为数字
        Assert.assertTrue(Validator.isNumber("1.1"));
        // 是否字母，包括大写和小写字母
        Assert.assertTrue(Validator.isWord("daodaotest"));
        // 是否为英文字母 、数字和下划线， 还支持：大写和小写字母和汉字（isLetter）
        Assert.assertTrue(Validator.isGeneral("dao_1"));
        // 是否全为大写或小写字母
        Assert.assertTrue(Validator.isLowerCase("daodaotest"));
        Assert.assertTrue(Validator.isUpperCase("DAODAOTEST"));
        // 检查给定的数字是否在指定范围内
        Assert.assertTrue(Validator.isBetween(1,1,1));
        // 生日
        Assert.assertTrue(Validator.isBirthday("20200830"));
        // 18位 身份证号格式校验，已经提 PR，改为调用 IdcardUtil 的方法，估计5.4.1 版本更新
        Assert.assertTrue(Validator.isCitizenId("11010119900307299X"));
        // 身份证校验，支持18位、15位和港澳台的10位
        // 支持：10位（isValidCard10），15位（isValidCard15），18位（isValidCard18），香港（isValidHKCard），台湾（isValidTWCard）
        Assert.assertTrue(IdcardUtil.isValidCard("11010119900307299X"));
        Assert.assertTrue(IdcardUtil.isValidCard18("11010119900307299X"));
        // 统一社会信用代码（营业执照注册号）
        Assert.assertTrue(Validator.isCreditCode("91350500676532779B"));
        // 中国车牌号
        Assert.assertTrue(Validator.isPlateNumber("京A88888"));
        // 邮编
        Assert.assertTrue(Validator.isZipCode("100000"));
        // uuid
        Assert.assertTrue(Validator.isUUID(UUID.randomUUID().toString()));
        // 正则表达式
        Assert.assertTrue(Validator.isMatchRegex("^d.*t$","daodaotest"));
        // 手机号
        Assert.assertTrue(Validator.isMobile("13888888888"));
        // url
        Assert.assertTrue(Validator.isUrl("http://www.baidu.com"));
        // email
        Assert.assertTrue(Validator.isEmail("dao@test.com"));
        // ip
        Assert.assertTrue(Validator.isIpv4("192.168.1.1"));
        Assert.assertTrue(Validator.isIpv6("CDCD:910A:2222:5498:8475:1111:3900:2020"));
    }

}
```

> 微信公众号：daodaotest
