---
title: mysql数据库填坑
date: 2017-11-28 19:08:25
categories: 杂烩
tags: database
---

> mysql是一个比较轻量级的数据库，在日常的开发过程中经常会用到，这里记录一下我在用mysql数据库时遇到的坑。

# 坑一，check约束无效。

在使用mysql数据库的时候，我们经常需要对数据进行约束，例如，我们有一个这样的表，

|user_id|user_name|user_sex|user_age|user_socer|
|--|--|--|--|--|
|int|varchar(10)|varchar(1)|int|int|

对应的mysql常见语句可以为：
```
CREATE TABLE `usr` (
  `user_id` int(11) NOT NULL,
  `user_name` varchar(10) NOT NULL,
  `user_sex` varchar(1) NOT NULL,
  `user_age` int(11) NOT NULL,
  `user_socer` int(11) NOT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
注意，这里指定的主键为``use_id``,且指定的方式入上述语句。

这个时候，如果我们想对其中的项进行约束，例如``user_sex``，我们只希望出现``f``代表女生,和``m``代表男生，其他的值都是无效的，这个时候用``check``语句是没有效果的，这个时候我们可以通过``enum``或者``set``类型来对其数据进行约束，具体实现为：
方式一：
创建时指定：
```
CREATE TABLE `usr` (
  `user_id` int(11) NOT NULL,
  `user_name` varchar(10) NOT NULL,
  `user_sex` SET('f','m') NOT NULL,
  `user_age` int(11) NOT NULL,
  `user_socer` int(11) NOT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
或者用enum来实现：
```
CREATE TABLE `usr` (
  `user_id` int(11) NOT NULL,
  `user_name` varchar(10) NOT NULL,
  `user_sex` enum('f','m') NOT NULL,
  `user_age` int(11) NOT NULL,
  `user_socer` int(11) NOT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
方式二：
在已经创建表的基础上，修改表的数据类型:
```
ALTER TABLE usr MODIFY COLUMN user_sex SET('f','m');
```
或者
```
ALTER TABLE usr MODIFY COLUMN user_sex enum('f','m');
```

至于添加其他的约束条件可以参考[这里](http://blog.csdn.net/peng_666666/article/details/54813098).

# 坑二，无法建立外键约束

如果公共关键字在一个关系中是主关键字，那么这个公共关键字被称为另一个关系的外键。由此可见，外键表示了两个关系之间的相关联系。以另一个关系的外键作主关键字的表被称为主表，具有此外键的表被称为主表的从表。

先记录一下完整的外键建立流程吧，假设我们已经建立了上面的``usr``数据库，我们还需要建立一个``action``数据库来记录用户的行为，其各内容如表格所示：
|action_id|action_user_id|action_time|action_conten|
|----|---|---|--|
|int|int|time|varchar(50)|

我们可以使用如下的sql语句进行创建：
```
CREATE TABLE `action` (
`action_id`  int NOT NULL ,
`action_user_id`  int NOT NULL ,
`action_time`  time NOT NULL ,
`action_content`  varchar(50) NOT NULL ,
PRIMARY KEY (`action_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

```
创建完成之后，我们需要在``action``和``usr``表中创建索引。本例中，我们将设置``usr``表中的``usr_id``为``action``表中``action_usr_id``的外键，下面介绍三种方式来创建外键。

首先介绍建立外键的几个必要条件：

* 两个表必须是InnoDB表类型。
* 使用在外键关系的域必须为索引型(Index)。
* 使用在外键关系的域必须与数据类型相同。
* 外键的不能是主键

稍微对上面的必要条件进行一个简单的说明，

## 方式一、通过sql语句

1.创建索引
首先为``user``表中的``user_id``创建索引：
```
ALTER TABLE usr ADD INDEX idx_user (user_id);
```
这里的idx_user是创建的索引的名字，并不重要，我们可以随便去，并且我们后面也不会用到它。我们只需要创建了他就好了。
下面我们来为``action``表中的``action_user_id``创建索引：
```
ALTER TABLE action ADD INDEX idx_action (action_user_id);
```

2.创建外键

再创建了索引之后，我们就可以创建外键了，sql语句如下：
```
ALTER TABLE `action` ADD FOREIGN KEY (`action_user_id`) 
REFERENCES `usr` (`user_id`) 
ON DELETE RESTRICT ON UPDATE RESTRICT;
```

这里对上面的sql语句进行一个简单的说明，第一行指定了我们的外键为``action``表中的``action_user_id``；
第二行指定了从键为``usr``表中的``user_id``；至于第三行代码，主要对外键的属性进行设置，简单介绍一下四个属性的作用：

* RESTRICT ：父表删除(更新)且外键对应子表记录存在时，则不允许删除(更新)。
* CASCADE ：父表删除(更新)时，外键对应子表记录同时删除(更新)。
* SET NULL ：父表删除(更新)时，外键对应子表记录同时置为NULL.(必须允许为NULL）
* NO ACTION：父表删除(更新)且外键对应子表记录存在时，则不允许删除(更新)。

到此，我们的外键就建立完毕了，如果一切比较顺利的话还是比较简单的。
上面是通过sql语句直接创建外键，不是特别直观，下面通过介绍两个工具的使用来介绍如何创建外键。

## 方式二、通过phpMyAdmin创建

1.创建索引
![创建索引](http://ovec6nnof.bkt.clouddn.com/phpMyAdmin%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95.png)
相同的，我们也需要为``action``表中的``action_user_id``也创建一个索引。

2.创建外键

![创建外键](http://ovec6nnof.bkt.clouddn.com/phpMyAdmin%E5%88%9B%E5%BB%BA%E5%A4%96%E9%94%AE.png)

![创建外键成功](http://ovec6nnof.bkt.clouddn.com/phpMyAdmin%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95%E5%AE%8C%E6%88%90.png)

## 方式三、通过navicat创建

1. 创建索引

这一步可以省略，因为通过navicat创建外键时会自动创建索引，这一步也可以手动操作。

选择： 设计表 -> 索引 -> 填写相应参数 -> 保存。

![创建索引](http://ovec6nnof.bkt.clouddn.com/navicat%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95.jpg)
2. 创建外键
创建外键也很简单，只需要选择：
设计表 -> 外键 -> 填写相应参数 -> 点击保存。即可。

![创建外键](http://ovec6nnof.bkt.clouddn.com/nativcat%E5%88%9B%E5%BB%BA%E5%A4%96%E9%94%AE.png)


坑：外键和从键的类型和长度必须一致，甚至于属性也需要一致，否则会创建失败，创建失败的的代号为：1215 。
提示为：1215 Cannot add the foreign key constraint。
这个时候就仔细查看一下主从建之间是不是存在某些差别。

# 参考链接

* [数据库的几个概念：主键，外键，索引，唯一索引](http://blog.csdn.net/xrt95050/article/details/5556411)
* [MySQL常见的建表选项及约束](https://www.cnblogs.com/geaozhang/p/6786105.html)
* [MySQL数据库——修改约束基本操作](http://blog.csdn.net/peng_666666/article/details/54813098)