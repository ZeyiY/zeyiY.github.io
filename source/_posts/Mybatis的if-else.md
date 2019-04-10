---
title: Mybatis的if-else
date: 2019-04-10 22:20:51
tags:
---

## Mybatis的if-else

#### mybaits 中没有else要用chose when otherwise 代替

表示方法：

```
<choose>
    <when test="">
        //...
    </when>
    <otherwise>
        //...
    </otherwise>
</choose>
```

choose为一个整体 
when是if 
otherwise是else

示例：

```
<select id="selectSelective" resultMap="xxx" parameterType="xxx">
    select
    <include refid="Base_Column_List"/>
    from xxx
    where del_flag=0
    <choose>
        <when test="xxx !=null and xxx != ''">
            and xxx like concat(concat('%', #{xxx}), '%')
        </when>
        <otherwise>
            and xxx like '**%'
        </otherwise>
    </choose>
</select>
```

