---
title: 七. MyBatis表间关系
tags:
  - Java
  - MyBatis
categories: MyBatis
abbrlink: 11259
date: 2019-11-20 09:04:57
---
# 七. MyBatis表间关系

以下案例对应的表为user和account

- user表字段

    ![user表字段](https://s2.ax1x.com/2019/11/20/MWKWnS.png)

- user表数据

    ![user表数据](https://s2.ax1x.com/2019/11/20/MWQBRI.png)

- account表字段

    ![account表字段](https://s2.ax1x.com/2019/11/20/MWK5kj.png)

- account表数据

    ![account表数据](https://s2.ax1x.com/2019/11/20/MWQsQP.png)

## 一对一关系

### 通过子类方式进行操作

1. 创建Account类

    ```java 
        package com.ydx.domain;
        
        import java.io.Serializable;
        
        public class Account implements Serializable {
        
            private Integer id;
            
            private Integer uid;
            
            private Double money;
            
            public Integer getId() {
                return id;
            }
            
            public void setId(Integer id) {
                this.id = id;
            }
            
            public Integer getUid() {
                return uid;
            }
            
            public void setUid(Integer uid) {
                this.uid = uid;
            }
            
            public Double getMoney() {
                return money;
            }
            
            public void setMoney(Double money) {
                this.money = money;
            }
            
            @Override
            public String toString() {
                return "Account{" +
                       "id=" + id +
                       ", uid=" + uid +
                       ", money=" + money +
                       '}';
            }
        }
    ```
2. 创建dao接口AccountDao并向其中添加方法

    ```java 
        /**
         * 查询所有账户并带有用户名称和地址信息
         * @return
         */
        List<AccountUser> findAllAccount();
    ```

3. 创建Account类的子类AccountUser类

    ```java 
        package com.ydx.domain;
        
        public class AccountUser extends Account {
        
            private String username;
        
            private String address;
        
            public String getUsername() {
                return username;
            }
        
            public void setUsername(String username) {
                this.username = username;
            }
        
            public String getAddress() {
                return address;
            }
        
            public void setAddress(String address) {
                this.address = address;
            }
        
            @Override
            public String toString() {
                return super.toString() + "\tAccountUser{" +
                        "username='" + username + '\'' +
                        ", address='" + address + '\'' +
                        '}';
            }
        }

    ```

4. 创建Mapper AccountDao.xml，并添加配置

    ```java 
        <!-- 查询所有账户并带有用户名称和地址信息-->
        <select id="findAllAccount" resultType="com.ydx.domain.AccountUser">
            SELECT a.*, u.username, u.address FROM account AS a, user AS u WHERE u.id = a.UID;
        </select>
    ```

5. 创建测试类，并添加测试方法

    ```java 
        /**
         * 查询所有账户并带有用户名称和地址信息
         */
        @Test
        public void testFindAllAccountUser() {
            List<AccountUser> aus = accountDao.findAllAccount();
            for (AccountUser au : aus) {
                System.out.println(au);
            }
        }
    ```

### 通过建立实体类关系的方式

1. 从表实体添加主表实体对象引用及其set、get方法

    ```java 
        //一对一关系：从表实体应该包含一个主表实体的引用
        private User user;
        
        public User getUser() {
            return user;
        }
        
        public void setUser(User user) {
            this.user = user;
        }
    ```
   
2. 在Mapper中封装resultMap

    ```xml
        <!-- 定义封装Account和User的resultMap -->
        <resultMap id="accountUserMap" type="com.ydx.domain.Account">
            <id property="id" column="aid"></id>
            <result property="uid" column="uid"></result>
            <result property="money" column="money"></result>
            <!-- 一对一的关系映射，配置User的内容-->
            <association property="user" column="uid">
                <id property="id" column="id"></id>
                <result column="username" property="username"></result>
                <result column="address" property="address"></result>
                <result column="sex" property="sex"></result>
                <result column="birthday" property="birthday"></result>
            </association>
        </resultMap>
    ```

3. 在dao接口中添加方法

    ```java 
        /**
         * 查询所有用户
         * @return
         */
        List<Account> findAll();
    ```

4. Mapper中添加配置

    ```xml
        <!-- 查询所有账户-->
        <select id="findAll" resultMap="accountUserMap">
            SELECT u.*, a.id AS aid, a.uid, a.money FROM account AS a, user AS u WHERE u.id = a.uid;
        </select>
    ```

5. 在测试类中添加测试方法

    ```java 
        /**
         * 测试查询所有账户操作
         */
        @Test
        public void testFindAll() {
            //使用代理对象执行方法
            List<Account> accounts = accountDao.findAll();
            for (Account account: accounts) {
                System.out.println(account);
                System.out.println(account.getUser());
            }
        }
    ```

## 一对多关系

1. 主表实体添加从表实体的引用

    ```java 
        //一对多关系映射：主表实体应该包含从表实体
        private List<Account> accounts;
    
        public List<Account> getAccounts() {
            return accounts;
        }
    
        public void setAccounts(List<Account> accounts) {
            this.accounts = accounts;
        }
    ```

2. 
    
    
