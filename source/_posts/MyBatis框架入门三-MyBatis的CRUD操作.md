---
title: MyBatis框架入门三. MyBatis基于XML的CRUD操作
color: '#EBF8FF'
tags:
  - Java
  - MyBatis
categories: MyBatis框架入门
abbrlink: 21570
date: 2019-11-16 23:29:39
---

# MyBatis框架入门三. MyBatis基于XML的CRUD操作

## 基于代理dao实现CRUD操作

**使用要求：**

- 持久层接口和持久层接口的映射配置必须在相同的包下；
- 持久层映射配置中 mapper 标签的 namespace 属性取值必须是持久层接口的全限定类名；
- SQL 语句的配置标签 \<select>、\<insert>、\<delete>、\<update> 的 id 属性必须和持久层接口的方法名相同。 

### 查询所有用户

1. 在dao接口中添加方法
    ```java
        /**
        * 查询所有用户
        * @return
        */
       List<User> findAll();
    ```
   
2. 在映射配置文件中添加配置
    ```xml
        <!-- 配置查询所有用户，id要写方法名称, resultType要写返回类型-->
        <select id="findAll" resultType="com.ydx.domain.User">
            SELECT * FROM user;
        </select>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试查询所有用户操作
        */
       @Test
       public void testFindAll(){
           //使用代理对象执行方法
           List<User> users = userDao.findAll();
           for (User user: users) {
               System.out.println(user);
           }
       }
    ```

### 插入用户

1. 在dao接口中添加方法
    ```java
        /**
         * 插入用户
         * @param user
         */
        void insertUser(User user);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
       <!-- 插入用户-->
       <insert id="insertUser">
           INSERT INTO user(username, address, sex, birthday) VALUES(#{username}, #{address}, #{sex}, #{birthday});
       </insert>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试插入用户操作
        */
       @Test
       public void testInsertUser() {
           User user = new User();
           user.setUsername("杨得轩");
           user.setAddress("上海");
           user.setSex("男");
           user.setBirthday(new Date());
   
           //执行插入用户方法
           userDao.insertUser(user);
       }
    ```
   
#### 插入用户并返回其id

1. 在dao接口中添加方法
    ```java
        /**
         * 插入用户并返回其id
         * @param user
         * @return
         */
        int insertUser2(User user);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
        <!-- 插入用户并返回其id-->
        <insert id="insertUser2">
            <!-- 返回最后插入的用户的id-->
            <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
                SELECT LAST_INSERT_ID();
            </selectKey>
            INSERT INTO user(username, address, sex, birthday) VALUES(#{username}, #{address}, #{sex}, #{birthday});
        </insert>
    ```
   
3. 在测试类中添加测试方法
    ```java
        /**
         * 测试插入用户操作，并返回其id
         */
        @Test
        public void testInsertUser2() {
            User user = new User();
            user.setUsername("杨得轩");
            user.setAddress("上海");
            user.setSex("男");
            user.setBirthday(new Date());
    
            //执行插入用户方法
            userDao.insertUser(user);
        }
    ```

### 更新用户

1. 在dao接口中添加方法
    ```java
        /**
         * 更新用户
         * @param user
         */
        void updateUser(User user);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
       <!-- 更新用户-->
       <update id="updateUser" parameterType="com.ydx.domain.User">
           UPDATE user SET username = #{username}, address = #{address}, sex = #{sex}, birthday = #{birthday} WHERE id = #{id};
       </update>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试更新用户操作
        */
       @Test
       public void testUpdateUser() {
           User user = new User();
           user.setId(51);
           user.setUsername("杨得轩");
           user.setAddress("上海青浦区");
           user.setSex("男");
           user.setBirthday(new Date());
   
           //执行更新用户方法
           userDao.updateUser(user);
       }
    ```

### 删除用户

1. 在dao接口中添加方法
    ```java
        /**
         * 删除用户
         * @param userId
         */
        void deleteUser(Integer userId);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
       <!-- 删除用户-->
       <delete id="deleteUser" parameterType="Integer">
           DELETE FROM user WHERE id = #{uid};
       </delete>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试删除用户操作
        */
       @Test
       public void testDeleteUser() {
           //执行删除用户方法
           userDao.deleteUser(51);
       }
    ```

### 根据id查询用户

1. 在dao接口中添加方法
    ```java
        /**
         * 根据id查询用户
         * @param userId
         * @return
         */
        User findById(Integer userId);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
       <!-- 根据id查询用户-->
       <select id="findById" parameterType="Integer" resultType="com.ydx.domain.User">
           SELECT * FROM user WHERE id = #{uid};
       </select>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试根据id查询用户操作
        */
       @Test
       public void testFindById() {
           //执行根据id查询方法
           User user = userDao.findById(48);
           System.out.println(user);
       }
    ```
 
### 根据名字模糊查询用户

#### 第一种方式

1. 在dao接口中添加方法
    ```java
        /**
         * 根据名字模糊查询用户信息
         * @param userName
         * @return
         */
        List<User> findByName(String userName);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
       <!-- 根据名字模糊查询用户-->
       <select id="findByName" parameterType="String" resultType="com.ydx.domain.User">
           SELECT * FROM user WHERE username LIKE #{name};
       </select>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试名字模糊查询用户操作
        */
       @Test
       public void testFindByName() {
           //执行根据名字模糊查询方法
           List<User> users = userDao.findByName("%王%");
           for (User user: users) {
               System.out.println(user);
           }
       }
    ```
   
#### 第二种方式

1. 在dao接口中添加方法
    ```java
        /**
         * 根据名字模糊查询用户信息，第二种方式（不推荐）
         * @param userName
         * @return
         */
        List<User> findByName2(String userName);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
       <!-- 根据名字模糊查询用户，另一种传参方式（不推荐）-->
       <select id="findByName2" parameterType="String" resultType="com.ydx.domain.User">
           SELECT * FROM user WHERE username LIKE '%${value}%';
       </select>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试名字模糊查询用户操作，第二种方式（不推荐）
        */
       @Test
       public void testFindByName2() {
           //执行根据名字模糊查询方法，第二种方式（不推荐）
           List<User> users = userDao.findByName2("王");
           for (User user: users) {
               System.out.println(user);
           }
       }
    ```

#### #{}与${}的区别

- **\#{}表示一个占位符号**
    通过#{}可以实现 preparedStatement 向占位符中设置值，自动进行 Java 类型和 JDBC 类型转换，#{}可以有效防止 SQL 注入；
    \#{}可以接收简单类型值或 pojo 属性值，如果 parameterType 传输单个简单类型值，#{}括号中可以是 value 或其它名称。
    
- **${}表示拼接 SQL 串**
    通过${}可以将 parameterType 传入的内容拼接在 SQL 中且不进行 JDBC 类型转换；
    ${}可以接收简单类型值或 POJO 属性值，如果 parameterType 传输单个简单类型值，${}括号中只能是 value。 

### 使用聚合查询总用户数

1. 在dao接口中添加方法
    ```java
       /**
        * 使用聚合查询总用户数
        * @return
        */
       int findTotal();
    ```
   
2. 在映射配置文件中添加配置
    ```xml
       <!-- 使用聚合查询总用户数-->
       <select id="findTotal" resultType="int">
           SELECT COUNT(id) FROM user;
       </select>
    ```
   
3. 在测试类中添加测试方法
    ```java
       /**
        * 测试使用聚合查询总用户数
        */
       @Test
       public void testFindTotal() {
           //执行使用聚合查询总用户数
           int count = userDao.findTotal();
           System.out.println(count);
       }
    ```

### **注意事项**
基本类型和 String 等常用的类我们可以直接写类型名称，也可以使用包名 . 类名的方式，例如：java.lang.String。

实体类类型，目前我们只能使用全限定类名。

究其原因，是 MyBaits 在加载时已经把常用的数据类型注册了别名，从而我们在使用时可以不写包名，而我们的是实体类并没有注册别名，所以必须写全限定类名。

> 引用 [MyBatis 3.5.3 官方文档](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)

> 这是一些为常见的 Java 类型内建的相应的类型别名。它们都是不区分大小写的，注意对基本类型名称重复采取的特殊命名风格。

|别名	|映射的类型|
|---|---|
_byte	|byte
_long	|long
_short	|short
_int	|int
_integer	|int
_double	|double
_float	|float
_boolean	|boolean
string	|String
byte	|Byte
long	|Long
short	|Short
int	|Integer
integer	|Integer
double	|Double
float	|Float
boolean	|Boolean
date	|Date
decimal	|BigDecimal
bigdecimal	|BigDecimal
object	|Object
map	|Map
hashmap	|HashMap
list	|List
arraylist	|ArrayList
collection	|Collection
iterator	|Iterator

>

## MyBatis参数

### parameterType（输入类型）
- 输入简单类型

- 输入POJO对象

    MyBatis 使用 OGNL 表达式解析对象字段的值，#{}或${}中的值为 POJO 属性名称。

- 输入POJO包装对象

    开发中通过 POJO 传递查询条件，查询条件是综合的查询条件，不仅包括用户查询条件还包括其它的查询条件（比如根据用户名查询用户信息，将用户信息也作为查询条件），这时可以使用包装对象传递输入参数。
    
    1. 编写QueryVo
        ```java 
            package com.ydx.domain;
            
            public class QueryVo {
            
                private User user;
            
                public User getUser() {
                    return user;
                }
            
                public void setUser(User user) {
                    this.user = user;
                }
            }
        ```
    2. 在dao接口中添加方法
        ```java 
           /**
            * 根据queryVo中的条件查询用户
            * @param vo
            * @return
            */
           List<User> findByVo(QueryVo vo);
        ```
       
    3. 在映射配置文件中添加配置
        ```java 
            <!-- 根据queryVo的条件查询用户-->
            <select id="findByVo" parameterType="com.ydx.domain.QueryVo" resultType="com.ydx.domain.User">
                SELECT * FROM user WHERE username LIKE #{user.username};
            </select>
        ```
        
    4. 在测试类中添加测试方法
        ```java 
           /**
            * 测试根据queryVo中的条件查询用户
            */
           @Test
           public void testFindByVo() {
               QueryVo vo = new QueryVo();
               User user = new User();
               user.setUsername("%王%");
               vo.setUser(user);
       
               //执行根据queryVo中的条件查询用户
               List<User> users = userDao.findByVo(vo);
               for (User u: users) {
                   System.out.println(u);
               }
           }
        ```
       
### returnType（输出类型）
- 输出简单类型

- 输出POJO对象

- 输出POJO列表

### resultMap（结果类型）
