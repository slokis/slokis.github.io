---
layout: post
title:  "MySQL之多级类别字典表"
date:   2020-06-01 15:21:36 +0530
categories: test
---
最近，在项目实践中遇到了一个多级分类的需求，即产品的字段中有类别字段。在创建数据库时一开始打算创建多个类别字段（大类、中类、小类）和多张类别表对应，可后来一想，如果这样设计的话，产品表中就关联了多张类别表的ID。十分不利于数据维护，效率也十分低下。

这时候，就想到了用一个类别字典表来代替原来的三张类别表，类别字典表中有clothCategory_id（自身ID），clothCategory_categoryNumber（编号），clothCategory_supCategory（父级ID），clothCategory_categoryName（类别名称）

类别字典表设计如图：
![avatar](/home/picture/1.png)


这样一来，我们只要在产品表里设计一个类别字段关联类别字典表（该字段存储多个类别中的最下级类别），因为照字典表设计，每一个类别都能找到唯一一条、从下往上的路径，这条路径就是该产品所有的类别了


那么，怎么样才能根据一个最下级类别找到它所有的上级类别？
我们只需要一个递归函数来解决：
代码如下：
```MySQL
DELIMITER $$

CREATE
    FUNCTION getParentList(rootId INT)
    RETURNS VARCHAR(100)
   
    BEGIN
	DECLARE pTemp VARCHAR(100);
	DECLARE cTemp VARCHAR(100);  #定义两个中间变量
	
	SET pTemp = '$';
	SET cTemp =CAST(rootId AS CHAR);  #将rootID转为char
	
	WHILE cTemp IS NOT NULL DO  
		 SET pTemp = CONCAT(pTemp,',',cTemp);  #用,连接两中间变量
		 SELECT GROUP_CONCAT(clothCategory_supCategory) INTO cTemp FROM tb_clothcategory   
		 WHERE FIND_IN_SET(clothCategory_id,cTemp)>0;
	END WHILE;  
	RETURN pTemp;
    END$$

DELIMITER ;
```
创建好了之后，我们只要用
```SELECT clothCategory_id,clothCategory_categoryName FROM tb_clothcategory WHERE FIND_IN_SET(clothCategory_id,getParentList(14));```
来查询，即可获得该ID所有的父级类别（包含自己）
来看看运行后的情况吧：
![avatar](/home/picture/1.png)
