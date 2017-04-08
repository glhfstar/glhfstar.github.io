---
title: SQLite数据库升级处理
date: 2016-04-02 21:22:44
tags:
---
**在app开发过程中，如果本地存储采用了SQLite。那么当app有新的需求时需要更改本地数据库的结构，比如说增加新的字段或者删除原有的字段。此时，如果没有采取正确的措施，会导致升级后的app进行数据存储时发生异常。**
**接下来介绍下如何处理数据库结构变化所导致的问题。**

### 增加字段
比如在原有的版本上增加一个`new`字段.先判断当前的数据库表中是否已经包含`new`字段，如果没有则为当前数据库表插入字段。

{% codeblock lang:objc %}
if (![fmdb columnExists:@"new" inTableWithName:@"app"]) {
            NSString *alertStr = [NSString stringWithFormat:@"alter table %@ add %@ varchar",@"app",@"new"];
            BOOL isSucess = [fmdb executeUpdate:alertStr];
            if (isSucess) {

            }else{
                
            }
        }
{% endcodeblock %}

### 删除字段
由于SQLite 仅仅支持 `ALTER TABLE` 语句的一部分功能，我们可以用  `ALTER TABLE` 语句来更改一个表的名字，也可向表中增加一个字段（列），但是我们不能删除一个已经存在的字段，或者更改一个已经存在的字段的名称、数据类型、限定符等等。 

个人觉得有以下两种处理方式：
**第一种是不修改表的结构，如果只有少量字段不需要使用了，保留着影响也不大。**
**第二种是把原来的数据库表改名为其它，然后新建新的数据库表，接着把旧的数据库表数据重新导入到新的数据库表，最后删掉旧的数据库表。**