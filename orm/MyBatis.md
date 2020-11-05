# MyBatis

[TOC]

### 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？

>Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取。
>
>而Mybatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，称之为半自动ORM映射工具。

### 占位符使用$和#的区别?

>$是传值直接拼接到sql中，存在sql注入风险，#将参数替换成问号

### 实体类属性名和表中字段名不一致怎么办？

> 1、sql语句中定义字段别名
>
> 2、resultMap中映射

```sql
<select id=”selectorder” parametertype=”int” resultetype=”me.gacl.domain.order”>
       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};
    </select>
```

```sql
<select id="getOrder" parameterType="int" resultMap="orderresultmap">
        select * from orders where order_id=#{id}
    </select>
 
   <resultMap type=”me.gacl.domain.order” id=”orderresultmap”>
        <!–用id属性来映射主键字段–>
        <id property=”id” column=”order_id”>
 
        <!–用result属性来映射非主键字段，property为实体类属性名，column为数据表中的属性–>
        <result property = “orderno” column =”order_no”/>
        <result property=”price” column=”order_price” />
    </reslutMap>
```

### 模糊查询like怎么写？

> 1、代码中加通配符
>
> 2、sql语句中拼接通配符，参数用$

```sql
string wildcardname = “%smi%”;
    list<name> names = mapper.selectlike(wildcardname);
 
    <select id=”selectlike”>
     select * from foo where bar like #{value}
    </select>
```

```sql
 string wildcardname = “smi”;
    list<name> names = mapper.selectlike(wildcardname);
 
    <select id=”selectlike”>
         select * from foo where bar like "%"${value}"%"
    </select>
```



### mapper如何传递多个参数

>1、\#{0} 数字表示第几个参数
>
>2、方法传参时使用 @param 注解
>
>3、多个参数封装成map

```java
//DAO层的函数
Public UserselectUser(String name,String area);  
//对应的xml,#{0}代表接收的是dao层中的第一个参数，#{1}代表dao层中第二参数，更多参数一致往后加即可。
<select id="selectUser"resultMap="BaseResultMap">  
    select * from user_user_t where user_name = #{0} and user_area = #{1}  
</select> 
```

```java
public interface usermapper {
   user selectuser(@param(“username”) string username,@param(“hashedpassword”) string hashedpassword);
}
然后,就可以在xml像下面这样使用(推荐封装为一个map,作为单个参数传递给mapper):
<select id=”selectuser” resulttype=”user”>
         select id, username, hashedpassword
         from some_table
         where username = #{username}
         and hashedpassword = #{hashedpassword}
</select>
```

```
List<user> getUserList(Map<String, Object> paramsMap);

<select id=”selectuser” resulttype=”user”>
         select id, username, hashedpassword
         from some_table
         where username = #{paramsMap.username}
         and hashedpassword = #{paramsMap.hashedpassword}
</select>
```

### mybatis动态传参数

>trim | where | set | foreach | if | choose | when | otherwise | bind
>
>完成多条件判断，动态的拼接sql

### 多表联查

>resultMap中，写关联表的映射

```sql
<mapper namespace="com.lcb.mapping.userMapper">  
    <!--association  一对一关联查询 -->  
    <select id="getClass" parameterType="int" resultMap="ClassesResultMap">  
        select * from class c,teacher t where c.teacher_id=t.t_id and c.c_id=#{id}  
    </select>  
 
    <resultMap type="com.lcb.user.Classes" id="ClassesResultMap">  
        <!-- 实体类的字段名和数据表的字段名映射 -->  
        <id property="id" column="c_id"/>  
        <result property="name" column="c_name"/>  
        <association property="teacher" javaType="com.lcb.user.Teacher">  
            <id property="id" column="t_id"/>  
            <result property="name" column="t_name"/>  
        </association>  
    </resultMap>  
 
 
    <!--collection  一对多关联查询 -->  
    <select id="getClass2" parameterType="int" resultMap="ClassesResultMap2">  
        select * from class c,teacher t,student s where c.teacher_id=t.t_id and c.c_id=s.class_id and c.c_id=#{id}  
    </select>  
 
    <resultMap type="com.lcb.user.Classes" id="ClassesResultMap2">  
        <id property="id" column="c_id"/>  
        <result property="name" column="c_name"/>  
        <association property="teacher" javaType="com.lcb.user.Teacher">  
            <id property="id" column="t_id"/>  
            <result property="name" column="t_name"/>  
        </association>  
 
        <collection property="student" ofType="com.lcb.user.Student">  
            <id property="id" column="s_id"/>  
            <result property="name" column="s_name"/>  
        </collection>  
    </resultMap>  
</mapper>
```

### 如何分页的

>物理分页：根据查询条件，数据库查询时完成
>
>逻辑分页：查出全部数据，在内存完成
>
>
>
>使用RowBounds对象是支持物理分页，传入的页码，每页大小，修改sql语句，根据条件限制查询



### 怎么扫描到dao、mapper

>在Application.java启动文件中，加注解：
>
>```
>@MapperScan(basePackages = {"com.youyuan.modules.*.mapper"})
>@MapperScan(basePackages = {"com.youyuan.modules.*.dao"})
>```
>
>配置文件中
>
>```
>#mybatis
>mybatis-plus:
>  mapper-locations: classpath*:mapper/**/*.xml
>  #实体扫描，多个package用逗号或者分号分隔
>  typeAliasesPackage: com.youyuan.modules.*.entity
>```

### 使用Mybatis-plus（增强工具）

>为了避免重复定义curd功能，使用mybatis-plus，继承一个公用的BaseMapper<T>接口，获得一组通用的crud方法
>
>使用Mybatis-Plus不使用xml文件，而是基于一组注解来解决实体类和数据库表的映射问题。

| 注解                    | 功能       | 说明                          |
| ----------------------- | ---------- | ----------------------------- |
| @TableName("tableName") | 指定表名   | 表名和类名一致时可以不写value |
| @TableId                | 指定主键   | 名称一致时可以不写value       |
| @TableField             | 指定字段名 | 名称一致时可以不写value       |

例：

```java
@TableName("DYNAMIC_ACTIVITY")
public class DynamicActivityEntity implements Serializable {
	private static final long serialVersionUID = 1L;

	/**
	 * 活动排期id
	 */
	@TableId
	private Integer activityId;
	/**
	 * 话题id
	 */
	private Integer topicId;
}
```

