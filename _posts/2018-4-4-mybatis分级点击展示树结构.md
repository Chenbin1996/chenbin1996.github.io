---
layout:     post
title:      mybatis分级点击展示树结构
subtitle:   一般的web项目中，都需要树结构
date:       2018-04-04
author:     如漩涡
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - mybatis
---

> 更多文章查看本人CSDN博客 [《如漩涡的博客》](https://blog.csdn.net/m0_37701381)

<p>  在WEB项目中，一二三级树形数据是很常见的事情，各种PID关系多个表等等，可以在每个XML中写个SQL，根据PID查询出数据，在Java代码中循环构造树，可以参考我之前的文章，但那都是数据量小的情况，表中加个索引，就非常快了。</p>

<p>   那一旦数据量大起来怎么办？比如十万表数据，用循环构造树的话，那得循环多少遍，产生多少内存多少对象，访问多少次数据库，而且前端展示效果极差，数据太大循环多次查询，前端请求后不能马上给出数据，因为后台还在与数据库做交流呢，这都是不可行的，那就只好使用分级查询。什么叫分级查询？比方说，浙江省下面是杭州市，杭州市下面有各个区/县，区/县下面又有街道/镇，街道/镇下面又有村/村委会，这里分为了五级，分级查询的意思就是说，我刚给前端的数据，就只有一级，浙江省，等用户点击了浙江省之后，前端再发送请求告诉后台，请求二级，快把数据给我，这时后台再把杭州市等市区二级数据给前端，点三级再给三级，这样分级给的数据就很快了，你想要几级的我去查几级，不需要重复查，前端传哪个级别的code给后台，后台就去查跟这个code有关系的是哪些数据。</p>

<p>   上个星期我就遇到了这么个情况，需要用户表和行政区划表数据做关系构造树，那可是十万条数据啊，我猜到肯定会很慢要用到分级查询了，但是好奇心害死猫，我还是用循环套循环的去构造这两张表关系的树，然后不出意外，请求了以后，后台的打印台一直在跑，跑了好几分钟啊那是，加了索引了，还好公司配置还行，不然肯定卡上天了，几分钟之后才把数据展示给了前端，这不GG了吗，肯定不行的，后面乖乖分级查询，研究了一下这两张表，行政区划表是没有任何用户表的关联字段的，但是用户表有行政区划表的字段，有五个，分别是省市县镇村五个字段的行政区划code/编号，那就行了，既然是一张表的事情，更省事了，还好不是多表，多表其实也不难，就是要写多个相同的SQL语句比较累，根据前端传入的等级，switch一下，哪一级做哪一级的查询，再把前端传入的行政区划code参数带到SQL中。接下来我来写一下分级查询的</p>

<p>首先是表结构，看一下用户表的行政区划字段，分别是省级，市级，县级，镇级，村级，字段类型都是长整型</p>
![表结构](https://github.com/Chenbin1996/chenbin1996.github.io/blob/master/img/2018-4-4-1.png)

<p>接下来核心部分，mybatis中XML里的SQL语句</p>
```XML
<select id="findRegionCode" resultType="com.uhope.rl.watersource.core.RegionTreeNode" parameterType="java.util.HashMap">
  SELECT
  su.`NAME` AS name,
  su.ID AS id,
  su.chairmanlevel AS regionLevel,
  0 AS type,
  0 AS isParent
  FROM
  `sm_user` su,
  md_administrative_region mar
  WHERE
  <if test="chairmanLevel == 1">su.PROVINCEID = mar.area_code</if>
  <if test="chairmanLevel == 2">su.CITYID = mar.area_code</if>
  <if test="chairmanLevel == 3">su.COUNTYID = mar.area_code</if>
  <if test="chairmanLevel == 4">su.TOWNID = mar.area_code</if>
  <if test="chairmanLevel == 5 or chairmanLevel == 6">su.VILLAGEID = mar.area_code</if>
  AND CASE #{chairmanLevel}
  WHEN 2 THEN
  su.PROVINCEID
  WHEN 3 THEN
  su.CITYID
  WHEN 4 THEN
  su. COUNTYID
  WHEN 5 THEN
  su.TOWNID
  ELSE
  su.VILLAGEID
  END = #{code}
  AND CASE #{chairmanLevel}
  WHEN 1 THEN
  su.cityid
  WHEN 2 THEN
  su.countyid
  WHEN 3 THEN
  su.townid
  WHEN 4 THEN
  su.villageid
  END IS NULL
  UNION ALL
  SELECT
  area_name AS name,
  area_code AS id,
  grade AS regionLevel,
  1 AS type,
  1 AS isParent
  FROM
  md_administrative_region
  WHERE
  parent_code = #{code}
</select>
```

<p>我的结果集是一个TreeNode类，专门构建树的，之前构造树的文章中有TreeNode的代码，传参类型我是用HashMap，我重头来解释一下这个SQL语句，两张表，在SQL中直接连接成树，第一次查询是用户表和行政区划表的多表查询，行政区划中就只有一个area_code字段是存放所有五级的，但用户表有五个，这个时候只好使用mybatis的if标签，根据前端传入的等级，传入是哪个等级，由于前端要根据我返回的等级+1后给我传下一个等级，多做了一个“or chairmanLevel = 6”，area_code就和五个中其中一个做关联，接着用SQL中的CASE WHEN THEN END，如果前端传入的等级是哪个，比如我在controller接口中默认给了1，接着前端就会传入2（因为说了前端会在这个等级上+1），传入2的时候，由于下一级要根据上一级的关系，就把省级的返回，就选出哪个字段，和if标签里做的差不多意思，然后END结束这个CASE，在 = 前端传入的code，等价于PROVINCEID/CITYID/COUNTYID/TOWNID/VILLAGEID = code，后面又有一个CASE，也是根据前端传入的等级，将下一级的字段内容略掉不返回，好让前端准确传入当前等级的code，现在用户表是做好了，要和行政区划表做关联，既然知道前端会传入code了，那好办，行政区划表有一个parent_code，能够找出和传入的code有关的下一级，用SQL的 UNION ALL函数，将两个查询出来的数据连接在一起，同时返回，那就达到要求了，需要注意的是，使用UNION ALL函数，两张表合并查询返回的字段要相同，所以我用了AS，都只返回了name，id两个字段，我用的结果集是TreeNode类，所以刚好匹配，接着是需要写好mapper接口，在Service中调用就可以了。</p>