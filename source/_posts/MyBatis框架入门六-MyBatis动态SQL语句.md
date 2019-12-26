---
title: MyBatis框架入门六. MyBatis动态SQL语句
color: '#EBF8FF'
tags:
  - Java
  - MyBatis
categories: MyBatis框架入门
abbrlink: 29950
date: 2019-11-19 15:22:51
---
# MyBatis框架入门六. MyBatis动态SQL语句

## if标签

1. 在dao接口中添加方法
    ```java
        /**
         * 根据输入参数条件查询用户
         * @param user 查询的条件，有可能有用户名、性别、地址等条件
         * @return
         */
        List<User> findByCondition(User user);
    ```
   
2. 在映射配置文件中添加配置
    ```xml
        <!-- 根据输入参数条件查询用户-->
        <select id="findByCondition" parameterType="com.ydx.domain.User" resultType="com.ydx.domain.User">
            SELECT * FROM user WHERE 1 = 1
            <if test="username != null">
                AND username LIKE #{username};
            </if>
        </select>
    ```
   
3. 在测试类中添加测试方法
    ```java
        /**
         * 测试根据输入参数条件查询用户
         */
        @Test
        public void testFindByCondition() {
            User user = new User();
            user.setUsername("老王");
    
            //执行根据输入参数条件查询用户
            List<User> users = userDao.findByCondition(user);
            for (User u : users) {
                System.out.println(u);
            }
        }
    ```

## where标签

1. 在映射配置文件中修改配置
    ```xml
        <!-- 根据输入参数条件查询用户-->
        <select id="findByCondition" parameterType="com.ydx.domain.User" resultType="com.ydx.domain.User">
            SELECT * FROM user
            <where>
                <if test="username != null">
                    AND username LIKE #{username}
                </if>
                <if test="sex != null">
                    AND sex LIKE #{sex};
                </if>
            </where>
        </select>
    ```
   
2. 在测试类中修改测试方法
    ```java
        /**
         * 测试根据输入参数条件查询用户
         */
        @Test
        public void testFindByCondition() {
            User user = new User();
            user.setUsername("老王");
            user.setSex("女");
   
            //执行根据输入参数条件查询用户
            List<User> users = userDao.findByCondition(user);
            for (User u : users) {
                System.out.println(u);
            }
        }
    ```

## foreach标签

1. 利用QueryVo，添加属性ids表示用户的id集合，并生成get、set方法

    ```java 
        private List<Integer> ids; 
   
        public List<Integer> getIds() {
            return ids;
        }
    
        public void setIds(List<Integer> ids) {
            this.ids = ids;
        }
    ```

2. 在dao接口中添加方法
    ```java
        /**
         * 根据QueryVo中提供的用户id查询用户
         * @param vo
         * @return
         */
        List<User> findByIds(QueryVo vo);
    ```
   
3. 在映射配置文件中添加配置
    ```xml
        <!-- 根据QueryVo中提供的用户id查询用户-->
        <select id="findInIds" parameterType="com.ydx.domain.QueryVo" resultType="com.ydx.domain.User">
            SELECT * FROM user
            <where>
                <if test="ids != null and ids.size() > 0">
                    <foreach collection="ids" open="id IN (" close=")" item="uid" separator=", ">
                        #{uid}
                    </foreach>
                </if>
            </where>
        </select>
    ```
   
4. 在测试类中添加测试方法
    ```java
        /**
         * 测试根据QueryVo中提供的用户id查询用户
         */
        @Test
        public void testFindInIds() {
            QueryVo vo = new QueryVo();
            List<Integer> list = new ArrayList<Integer>();
            list.add(41);
            list.add(42);
            list.add(43);
            vo.setIds(list);
    
            //执行根据QueryVo中提供的用户id查询用户
            List<User> users = userDao.findInIds(vo);
            for (User user : users) {
                System.out.println(user);
            }
        }
    ```
   
## sql标签与include标签

sql标签和include标签可以抽取重复的SQL语句。

1. 在映射配置文件中添加配置

    ```xml
        <!-- 抽取重复的SQL语句-->
        <sql id="defaultTable">
            SELECT * FROM user
        </sql>
    ```
   
2. 修改其他配置

    ```xml
        <!-- 根据QueryVo中提供的用户id查询用户-->
        <select id="findInIds" parameterType="com.ydx.domain.QueryVo" resultType="com.ydx.domain.User">
            <include refid="defaultTable"/>
            <where>
                <if test="ids != null and ids.size() > 0">
                    <foreach collection="ids" open="id IN (" close=")" item="uid" separator=", ">
                        #{uid}
                    </foreach>
                </if>
            </where>
        </select>
    ```
