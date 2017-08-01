# 《MyBatis从入门到精通》笔记

![cover](http://img14.360buyimg.com/n1/jfs/t6367/53/683249106/144792/c9ff8258/59434a3fN793cb89f.jpg)

## MyBatis入门

**创建Maven项目**

**修改`pom`文件**

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
	                    http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>tk.mybatis</groupId>
	<artifactId>simple</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<java.version>1.6</java.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.4</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.38</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.12</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.12</version>
		</dependency>
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.17</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

**准备数据库**

**配置MyBatis**

在`src/main/resources`下面创建`mybatis-config.xml`

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
    
     <typeAliases>
        <package name="tk.mybatis.simple.model"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC">
                <property name="" value=""/>
            </transactionManager>
            <dataSource type="UNPOOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://192.168.16.137:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="tk/mybatis/simple/mapper/CountryMapper.xml"/>
    </mappers>
</configuration>

```

**创建POJO**

**Mapper.xml文件**

在`src/main/resources`创建`tk/mybatis/simple/mapper`目录，然后创建`CountryMapper.xml`文件

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
					"http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="tk.mybatis.simple.mapper.CountryMapper">
	<select id="selectAll" resultType="Country">
		select id,countryname,countrycode from country
	</select>
</mapp
```

**配置Log4j**

在`src/main/resources`中添加`log4j.properties`配置文件

```
#全局配置
log4j.rootLogger=ERROR, stdout

#MyBatis 日志配置
log4j.logger.tk.mybatis.simple.mapper=TRACE

#控制台输出配置
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

**代码**

```
package tk.mybatis.simple.mapper;

import java.io.IOException;
import java.io.Reader;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.BeforeClass;
import org.junit.Test;

import tk.mybatis.simple.model.Country;

public class CountryMapperTest {
	
	private static SqlSessionFactory sqlSessionFactory;
	
	@BeforeClass
	public static void init(){
		try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            //InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml")
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
            reader.close();
        } catch (IOException ignore) {
        	ignore.printStackTrace();
        }
	}
	
	@Test
	public void testSelectAll(){
		SqlSession sqlSession = sqlSessionFactory.openSession();
		try {
			List<Country> countryList = sqlSession.selectList("selectAll");
			printCountryList(countryList);
		} finally {
			sqlSession.close();
		}
	}
	
	private void printCountryList(List<Country> countryList){
		for(Country country : countryList){
			System.out.printf("%-4d%4s%4s\n",country.getId(), country.getCountryname(), country.getCountrycode());
		}
	}
}
```

**运行结果**

```
DEBUG [main] - ==>  Preparing: select id, countryname, countrycode from country 
DEBUG [main] - ==> Parameters: 
TRACE [main] - <==    Columns: id, countryname, countrycode
TRACE [main] - <==        Row: 1, 中国, CN
TRACE [main] - <==        Row: 2, 美国, US
TRACE [main] - <==        Row: 3, 英国, GB
TRACE [main] - <==        Row: 4, 俄罗斯, RU
TRACE [main] - <==        Row: 5, 法国, FR
DEBUG [main] - <==      Total: 5
1     中国  CN
2     美国  US
3     英国  GB
4    俄罗斯  RU
5     法国  FR
```

## MyBatis XML方式的基本用法
### 权限控制需求
#### 数据库准备
```
CREATE TABLE `sys_user` (
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `user_name` varchar(50) DEFAULT NULL COMMENT '用户名',
  `user_password` varchar(50) DEFAULT NULL COMMENT '密码',
  `user_email` varchar(50) DEFAULT NULL COMMENT '邮箱',
  `user_info` text COMMENT '简介',
  `head_img` blob COMMENT '头像',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1002 DEFAULT CHARSET=utf8;

CREATE TABLE `sys_role` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `role_name` varchar(50) DEFAULT NULL COMMENT '角色名称',
  `enabled` int(11) DEFAULT NULL COMMENT '有效标志',
  `create_by` bigint(11) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

CREATE TABLE `sys_privilege` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '权限ID',
  `privilege_name` varchar(50) DEFAULT NULL COMMENT '权限名称',
  `privilege_url` varchar(200) DEFAULT NULL COMMENT '权限URL',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;

CREATE TABLE `sys_user_role` (
  `user_id` bigint(20) DEFAULT NULL COMMENT '用户ID',
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色ID'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `sys_role_privilege` (
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色ID',
  `privilege_id` bigint(20) DEFAULT NULL COMMENT '权限ID'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 创建实体类
在`self.edu.mybatis.model`包下创建对应5张表的POJO

### 使用XML方式
MyBatis支持接口调用。接口即可以配合XML使用也可以配置注解使用，注解只能在接口里使用

在`self.edu.mybatis.model`包下创建对应5张表的Mapper接口，例如

```
public interface UserMapper {

}
```

在`src/main/resources/self/edu/mybatis/mapper`文件夹下创建对应5张表的Mapper.xml文件，例如

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
					"http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="self.edu.mybatis.mapper.UserMapper">

</mapper>
```

在`mybatis-config.xml`里配置

```
<mappers>
	<mapper resource="self/edu/mybatis/mapper/UserMapper/xml" />
	<mapper resource="self/edu/mybatis/mapper/RoleMapper/xml" />
	<mapper resource="self/edu/mybatis/mapper/PrivilegeMapper/xml" />
	<mapper resource="self/edu/mybatis/mapper/UserRoleMapper/xml" />
	<mapper resource="self/edu/mybatis/mapper/RolePrivilegeMapper/xml" />
</mappers>
```

也可以使用自动的方式

```
<mappers>
	<package name="self.edu.mybatis.mapper" />
</mappers>
```

### select用法
在`UserMapper.xml`里添加

```
<mapper namespace="self.edu.mybatis.mapper.UserMapper">
	<resultMap id="userMap" type="self.edu.mybatis.model.SysUser">
		<id property="id" column="id"/>
		<result property="userName" column="user_name"/>
		<result property="userPassword" column="user_password"/>
		<result property="userEmail" column="user_email"/>
		<result property="userInfo" column="user_info"/>
		<result property="headImg" column="head_img" jdbcType="BLOB"/>
		<result property="createTime" column="create_time" jdbcType="TIMESTAMP"/>
	</resultMap>
	<select id="selectById" resultMap="userMap">
		select * from sys_user where id = #{id}
	</select>
</mapper>
```

`<select>`标签的id和接口的方法名一致，这样把两者连接起来。当只用XML而不用接口的时候，`namespace`可以随便取值。

`ResultMap`用法参考[这里](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#Result_Maps)

`<select>`用法参考[这里](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#select)

如果用`resultType`定义返回对象，需要映射数据库列名和对象属性名

```
<select id="selectAll" resultType="self.edu.mybatis.model.SysUser">
    select id, 
    	user_name userName, 
        user_password userPassword,
        user_email userEmail,
        user_info userInfo,
        head_img headImg,
        create_time createTime
    from sys_user
</select>
```
接口中是

```
List<SysUser> selectAll();
```
如果在`mybatis-config.xml`中开启

```
<settings>
	<setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```
那么可以进行自动映射

```
<select id="selectAll" resultType="self.edu.mybatis.model.SysUser">
    select id, 
    	user_name,
        user_password,
        user_email,
        user_info,
        head_img,
        create_time
    from sys_user
</select>
```

测试方法和测试结果

```
UserMapper userMapper = session.getMapper(UserMapper.class);
SysUser user = userMapper.selectById(1L);
List<SysUser> users = userMapper.selectAll();

Mon Jul 31 14:53:59 CST 2017 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
DEBUG [main] - ==>  Preparing: select * from sys_user where id = ? 
DEBUG [main] - ==> Parameters: 1(Long)
TRACE [main] - <==    Columns: id, user_name, user_password, user_email, user_info, head_img, create_time
TRACE [main] - <==        Row: 1, admin, 123456, admin@mybatis.tk, <<BLOB>>, <<BLOB>>, 2016-04-01 17:00:58.0
DEBUG [main] - <==      Total: 1
DEBUG [main] - Cache Hit Ratio [self.edu.mybatis.mapper.UserMapper]: 0.0
Mon Jul 31 14:54:00 CST 2017 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
DEBUG [main] - ==>  Preparing: select id, user_name userName, user_password userPassword, user_email userEmail, user_info userInfo, head_img headImg, create_time createTime from sys_user 
DEBUG [main] - ==> Parameters: 
TRACE [main] - <==    Columns: id, userName, userPassword, userEmail, userInfo, headImg, createTime
TRACE [main] - <==        Row: 1, admin, 123456, admin@mybatis.tk, <<BLOB>>, <<BLOB>>, 2016-04-01 17:00:58.0
TRACE [main] - <==        Row: 1001, test, 123456, test@mybatis.tk, <<BLOB>>, <<BLOB>>, 2016-04-01 17:01:52.0
DEBUG [main] - <==      Total: 2
```

多表联合查询时，返回的字段不在一个对象里，这时

1. 新建继承对象，添加字段
2. 使用`<association>`或`<collection>`
3. 映射嵌套，见如下代码

```
//增强POJO
public class SysRole {
	...
	private SysUser user;
}

//修改映射
<select id="selectRoesByUserId" resultType="self.edu.mybatis.model.SysRole">
    select r.id, 
    	r.role_name roleName,
    	r.enabled,
    	r.create_by createBy,
    	r.create_time createTime,
    	u.user_name as "user.userName",
    	u.user_email as "user.userEmail"
    from sys_user u
    inner join sys_user_role ur on u.id = ur.user_id
    inner join sys_role r on ur.role_id = r.id
    where u.id = #{userId}
</select>
```

### insert用法
#### 简单的insert方法
接口添加

```
int insert(SysUser sysUser);
```

`UserMapper.xml`添加

```
<insert id="insert">
		insert into sys_user(
			id, user_name, user_password, user_email, 
			user_info, head_img, create_time)
		values(
			#{id}, #{userName}, #{userPassword}, #{userEmail}, 
			#{userInfo}, #{headImg, jdbcType=BLOB}, #{createTime, jdbcType=TIMESTAMP})
	</insert>
```
具体配置参考[这里](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#insert_update_and_delete)

测试代码与运行结果

```
try{
	UserMapper userMapper = session.getMapper(UserMapper.class);
	SysUser user = new SysUser();
	user.setUserName("test2");
	user.setUserPassword("123456");
	user.setUserEmail("test@mybastis.tk");
	user.setUserInfo("test info");
	user.setCreateTime(new Date());
	//返回值是执行SQL影响的行数
	int result = userMapper.insert(user);
	session.commit();
}finally{
	session.close();
}

DEBUG [main] - ==>  Preparing: insert into sys_user( id, user_name, user_password, user_email, user_info, head_img, create_time) values( ?, ?, ?, ?, ?, ?, ?) 
DEBUG [main] - ==> Parameters: null, test(String), 123456(String), test@mybastis.tk(String), test info(String), null, 2017-07-31 15:38:21.159(Timestamp)
DEBUG [main] - <==    Updates: 1
```

#### 使用JDBC方式返回主键自增的值
`<select>`语句改为

```
<insert id="insert" useGeneratedKeys="true" keyProperty="id">
	insert into sys_user(
		user_name, user_password, user_email, 
		user_info, head_img, create_time)
	values(
		#{userName}, #{userPassword}, #{userEmail}, 
		#{userInfo}, #{headImg, jdbcType=BLOB}, #{createTime, jdbcType=TIMESTAMP})
</insert>
```

测试代码与运行结果(从对象里拿到生成的id)

```
try{
	UserMapper userMapper = session.getMapper(UserMapper.class);
	SysUser user = new SysUser();
	user.setUserName("test");
	user.setUserPassword("123456");
	user.setUserEmail("test@mybastis.tk");
	user.setUserInfo("test info");
	user.setCreateTime(new Date());
	//返回值是执行SQL影响的行数
	int result = userMapper.insert(user);
	//回写到user里
	System.out.println(user.getId());
	session.commit();
}finally{
	session.close();
}

DEBUG [main] - ==>  Preparing: insert into sys_user( user_name, user_password, user_email, user_info, head_img, create_time) values( ?, ?, ?, ?, ?, ?) 
DEBUG [main] - ==> Parameters: test(String), 123456(String), test@mybastis.tk(String), test info(String), null, 2017-07-31 15:40:50.57(Timestamp)
DEBUG [main] - <==    Updates: 1
1008
```

#### 使用selectKey返回主键的值
```
<insert id="insert3">
	insert into sys_user(
		user_name, user_password, user_email, 
		user_info, head_img, create_time)
	values(
		#{userName}, #{userPassword}, #{userEmail}, 
		#{userInfo}, #{headImg, jdbcType=BLOB}, #{createTime, jdbcType=TIMESTAMP})
	<selectKey keyColumn="id" resultType="long" keyProperty="id" order="AFTER">
		SELECT LAST_INSERT_ID()
	</selectKey>
</insert>
```

如果是Oracle数据库，主次次序设成BEFORE，表示要先从序列获得值再进行插入，而MySql是在插入成功后再获得主键

```
<selectKey keyColumn="id" resultType="long" keyProperty="id" order="BEFORE">
	SELECT SEQ_ID.nextval from dual
</selectKey>
```

* DB2 使用 VALUES IDENTITY_VAL_LOCAL()
* SQLSERVER 使用 SELECT SCOPE_IDENTITY()
* CLOUDSCAPE 使用 VALUES IDENTITY_VAL_LOCAL()
* DERBY 使用 VALUES IDENTITY_VAL_LOCAL()
* HSQLDB 使用 CALL IDENTITY()
* SYBASE 使用 SELECT @@IDENTITY
* DB2_MF 使用 SELECT IDENTITY_VAL_LOCAL() FROM SYSTEM.SYSDUMMY1
* INFORMIX 使用 select dbinfo('sqlca.sqlerrd1' from systables where tabid=1)

### update用法
```
<update id="updateById">
	update sys_user 
	set user_name = #{userName},
		user_password = #{userPassword},
		user_email = #{userEmail},
		user_info = #{userInfo},
		head_img = #{headImg, jdbcType=BLOB},
		create_time = #{createTime, jdbcType=TIMESTAMP}
	where id = #{id}
</update>
```

测试代码和运行结果

```
try{
	UserMapper userMapper = session.getMapper(UserMapper.class);
	SysUser user = userMapper.selectById(1L);
	user.setUserName("admin_test");
	//返回值是执行SQL影响的行数
	int result = userMapper.updateById(user);
	session.commit();
}finally{
	session.close();
}

DEBUG [main] - ==>  Preparing: select * from sys_user where id = ? 
DEBUG [main] - ==> Parameters: 1(Long)
TRACE [main] - <==    Columns: id, user_name, user_password, user_email, user_info, head_img, create_time
TRACE [main] - <==        Row: 1, admin, 123456, admin@mybatis.tk, <<BLOB>>, <<BLOB>>, 2016-04-01 17:00:58.0
DEBUG [main] - <==      Total: 1
DEBUG [main] - ==>  Preparing: update sys_user set user_name = ?, user_password = ?, user_email = ?, user_info = ?, head_img = ?, create_time = ? where id = ? 
DEBUG [main] - ==> Parameters: admin_test(String), 123456(String), admin@mybatis.tk(String), 管理员(String), null, 2016-04-01 17:00:58.0(Timestamp), 1(Long)
DEBUG [main] - <==    Updates: 1
```

### delete用法
```
<delete id="deleteById">
	delete from sys_user where id = #{id}
</delete>
```
测试代码与结果

```
try{
	UserMapper userMapper = session.getMapper(UserMapper.class);
	SysUser user = userMapper.selectById(1008L);
	//返回值是执行SQL影响的行数
	int result = userMapper.deleteById(user);
	session.commit();
}finally{
	session.close();
}

DEBUG [main] - ==>  Preparing: select * from sys_user where id = ? 
DEBUG [main] - ==> Parameters: 1008(Long)
TRACE [main] - <==    Columns: id, user_name, user_password, user_email, user_info, head_img, create_time
TRACE [main] - <==        Row: 1008, test, 123456, test@mybastis.tk, <<BLOB>>, <<BLOB>>, 2017-07-31 15:40:51.0
DEBUG [main] - <==      Total: 1
DEBUG [main] - ==>  Preparing: delete from sys_user where id = ? 
DEBUG [main] - ==> Parameters: 1008(Long)
DEBUG [main] - <==    Updates: 1
```

### 多个接口参数的用法
* 使用JavaBean包装多个参数
* 使用Map类型作为参数
* 使用@Param注解

接口定义

```
List<SysRole> selectRolesByUserAndRole(@Param("user")SysUser user, @Param("role")SysRole role);
```

在xml中就可以使用`#{user.id}`这样的格式（用`#{param1}`代替也是可以的，但不具有可读性）

```
<select id="selectRolesByUserAndRole" resultType="self.edu.mybatis.model.SysRole">
    select 
		r.id, 
		r.role_name roleName, 
		r.enabled,
		r.create_by createBy,
		r.create_time createTime
	from sys_user u
	inner join sys_user_role ur on u.id = ur.user_id
	inner join sys_role r on ur.role_id = r.id
    where u.id = #{user.id} and r.enabled = #{role.enabled}
</select>
```

### Mapper接口动态代理实现原理
JDK动态代理使得Mapper接口没有实现类也能被正常调用

## MyBatis注解方式的基本用法
### @Select注解
```
@Select({"select id, role_name roleName, enabled,
			create_by createBy,
			create_time createTime
			from sys_role
			where id=#{id}"})
SysRole selectById(Long id);
```
映射的方式
1. 在SQL语句里映射
2. 配置`mapUnderscoreToCamelCase`
3. 使用@Results方式

```
@Results({
	@Result(property="id", column="id", id=true),
	@Result(property="roleName", column="role_name"),
	@Result(property="enabled", column="enabled"),
	@Result(property="createBy", column="create_by"),
	@Result(property="createTime", column="create_time")
})
@Select({"select id, role_name, enabled,
			create_by,
			create_time
			from sys_role
			where id=#{id}"})
SysRole selectById(Long id);
```

3.3.0之前的版本，`@Results`不能复用，3.3.1开始，`@Results`可以配置id属性

```
@Results(id="roleResultMap", value={
	@Result(property="id", column="id", id=true),
	...
})

@ResultMap("roleResultMap")
@Select("select * from sys_role")
List<SysRole> selectAll();
```

### @Insert注解
不返回自增主键

```
@Insert({"insert into sys_role(id, role_name, enabled, create_by, create_time)", "values(#{id}, #{roleName}, #{enabled}, #{createBy}, #{createTime, jdbcType=TIMESTAMP})"})
int insert(SysRole sysRole);
```

返回自增主键

```
@Insert({"insert into sys_role(role_name, enabled, create_by, create_time)", "values(#{roleName}, #{enabled}, #{createBy}, #{createTime, jdbcType=TIMESTAMP})"})
@Options(useGeneratedKeys = true, keyProperty = "id")
int insert2(SysRole sysRole);
```

返回非自增主键

```
@Insert({"insert into sys_role(role_name, enabled, create_by, create_time)", "values(#{roleName}, #{enabled}, #{createBy}, #{createTime, jdbcType=TIMESTAMP})"})
@SelectKey(statement = "SELECT LAST_INSERT_ID()", keyProperty = "id", resultType = Long.class, before = false)
int insert3(SysRole sysRole);
```

### @Update注解和@Delete注解
```
@Update({"update sys_role",
		     "set role_name = #{roleName},",
				 "enabled = #{enabled},",
				 "create_by = #{createBy},",
				 "create_time = #{createTime, jdbcType=TIMESTAMP}",
			 "where id = #{id}"
		})
int updateById(SysRole sysRole);
	
@Delete("delete from sys_role where id = #{id}")
int deleteById(Long id);
```

### @Provider注解
```
@CacheNamespace
public interface PrivilegeMapper {

	@SelectProvider(type = PrivilegeProvider.class, method = "selectById")
	SysPrivilege selectById(Long id);
	
	@SelectProvider(type = PrivilegeProvider.class, method = "selectByPrivilege")
	List<SysPrivilege> selectByPrivilege(SysPrivilege privilege);
	
	@SelectProvider(type = PrivilegeProvider.class, method = "selectAll")
	List<SysPrivilege> selectAll();
	
	//这个方法和上面的方法同名，都会使用 tk.mybatis.simple.mapper.PrivilegeMapper.selectAll 方法
	List<SysPrivilege> selectAll(RowBounds rowBounds);
	
	@InsertProvider(type = PrivilegeProvider.class, method = "insert")
	@Options(useGeneratedKeys = true, keyProperty = "id")
	int insert(SysPrivilege sysPrivilege);
	
	//TODO 使用相同的insert方法，但是主键方式不同
	@InsertProvider(type = PrivilegeProvider.class, method = "insert")
	@SelectKey(statement = "SELECT LAST_INSERT_ID()", keyProperty = "id", resultType = Long.class, before = false)
	int insert2(SysPrivilege sysPrivilege);
	
	@UpdateProvider(type = PrivilegeProvider.class, method = "updateById")
	int updateById(SysPrivilege sysPrivilege);
	
	@DeleteProvider(type = PrivilegeProvider.class, method = "deleteById")
	int deleteById(Long id);
	
}
```
在PrivilegeProvider类里

```
public class PrivilegeProvider {
	
	public String selectById(final Long id){
		return new SQL(){
			{
				SELECT("id, privilege_name, privilege_url");
				FROM("sys_privilege");
				WHERE("id = #{id}");
			}
		}.toString();
	}
	
	public String selectByPrivilege(final SysPrivilege privilege){
		return new SQL(){
			{
				SELECT("id, privilege_name, privilege_url");
				FROM("sys_privilege");
				//参数不为空的时候才能使用这些条件进行查询，参数 null 时查询所有
				if(privilege != null){
					if(privilege.getId() != null){
						WHERE("id = #{id}");
					}
					if(privilege.getPrivilegeName() != null && privilege.getPrivilegeName().length() > 0){
						//注意 MySql 中的 concat 函数
						WHERE("privilege_name like concat('%',#{privilegeName},'%')");
					}
					if(privilege.getPrivilegeUrl() != null && privilege.getPrivilegeUrl().length() > 0){
						//这里为了举例，直接拼接的字符串
						WHERE("privilege_url like '%" +privilege.getPrivilegeUrl()+ "%'");
					}
				}
			}
		}.toString();
	}
	
	public String selectAll(){
		return "select * from sys_privilege";
	}
	
	public String insert(final SysPrivilege sysPrivilege){
		return new SQL(){
			{
				INSERT_INTO("sys_privilege");
				VALUES("privilege_name", "#{privilegeName}");
				VALUES("privilege_url", "#{privilegeUrl}");
			}
		}.toString();
	}
	
	public String updateById(final SysPrivilege sysPrivilege){
		return new SQL(){
			{
				UPDATE("sys_privilege");
				SET("privilege_name = #{privilegeName}");
				SET("privilege_url = #{privilegeUrl}");
				WHERE("id = #{id}");
			}
		}.toString();
	}
	
	public String deleteById(final Long id){
		return new SQL(){
			{
				DELETE_FROM("sys_privilege");
				WHERE("id = #{id}");
			}
		}.toString();
	}
}
```

## MyBatis动态SQL
参考[这里](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html)

OGNL用法
1. e1 or e2
2. e1 and e2
3. e1 == e2 / e1 eq e2
4. e1 != e2 / e1 neq e2
5. e1 lt e2 (lte gt gte)
6. e1 + e2 (* / - %)
7. !e /not e
8. e.method(args)
9. e.property
10. e1[e2]
11. @class@method(args) 调用类的静态方法
12. @class@field 调用类的静态字段

## Mybatis代码生成器
参考[这里](http://www.mybatis.org/generator/)

### XML配置详解
参考[这里](http://www.mybatis.org/generator/configreference/xmlconfig.html)

作者[博文](http://blog.csdn.net/isea533/article/details/42102297)

XML文件头

```
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
```
根节点

```
<generatorConfiguration>

</generatorConfiguration>
```

`generatorConfiguration`下一级的3个子标签是`properties`、`classPathEntry`、`context`。顺序要和列举的顺序一致，在后面列举其他标签的时候也必须严格按照列举这些标签的顺序进行配置

**properties**用来指定外部的属性元素，可以配置0-1个，引入属性文件后，可以在配置中使用`${property}`这种形式的引用。

properties标签包含`resource`和`url`两个属性，只能使用其中一个

* resource：指定classpath下的属性文件，类型`com/myproject/generatorConfig.properties`
* url：指定文件系统上的特定位置，例如`file:///C:/myfolder/generatorConfig.properties`

**classPathEntry**可以配置0到多个，最常见的用法是通过属性`location`指定驱动的路径

```
<classPathEntry location="E:\mysql\mysql-connector-java-5.1.29.jar" />
```

**context**配置1到多个，用于指定一组对象的环境，如数据库，生成对象的类型和数据库中的表。运行MBG的时候还可以指定要运行的`context`

`context`标签只有一个必选的`id`属性，可以在运行MBG时使用，其他可选属性如下

* defaultModelType - 定义了MBG如何生成实体类
	*  conditional - 默认值，和下面的hierarchical类似，如果一个表的主键只有一个字段，那么不会为该字段生成单独的实体类，而是会将该字段合并到基本实体类中
	*  flat - 该模型只为每张表生成一个实体类。这个实体类包含表中所有字段，这种模型最简单，推荐使用
	*  hierarchical - 如果表有主键，那么该模型会产生一个单独的主键实体类，如果表还有BLOB字段，则会为表生成一个包含所有BLOB字段的单独的实体类，然后为所有其他的字段另外生成一个单独的实体类。MBG会在所有生成的实体类之间维护一个继承关系
* targetRuntime - 此属性用于指定生成的代码的运行时环境，支持以下
	* MyBatis3 - 默认值
	* MyBatis3Simple - 不会生成与Example相关的方法
	* Ibatis2Java2
	* Ibatis2Java5
* introspectedColumnImpl - 该参数可以指定扩展`org.mybatis.generator.api.Introspected.Column`类的实现类

一般情况下，使用如下配置即可

```
<context id="mysql" defaultModelType="flat">
``` 
如果不希望生成Example查询有关的内容，则可以

```
<context id="mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
```

其他context的子标签有（有严格的配置顺序）

* property (0..N)
* plugin (0..N)
* commentGenerator (0 or 1)
* connectionFactory (either connectionFactory or jdbcConnection is Required)
* jdbcConnection (either connectionFactory or jdbcConnection is Required)
* javaTypeResolver (0 or 1)
* javaModelGenerator (1 Required)
* sqlMapGenerator (0 or 1)
* javaClientGenerator (0 or 1)
* table (1..N)

#### property
与分隔符有关的

* autoDelimitKeywords
* beginningDelimiter
* endingDelimiter

分隔符默认是`"`，MySql里要改为`

```
<context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
	<property name="autoDelimitKeywords" value="true" />
	<property name="beginningDelimiter" value="`"/>
	<property name="endingDelimiter" value="`"/> 
</context>
```

其他属性

* javaFileEncoding
* javaFormatter
* xmlFormatter

属性javaFileEncoding设置要使用的Java文件的编码，默认使用当前平台的编码

最后两个javaFormatter和xmlFormatter属性**可能会**很有用，如果你想使用模板来定制生成的java文件和xml文件的样式，你可以通过指定这两个属性的值来实现。

#### plugin
用来定义一个插件，用于扩展或修改通过MBG生成的代码，按在配置中声明的顺序执行。

例如，缓存插件`org.mybatis.generator.plugins.CachePlugin`

* cache_eviction
* cache_flushInterval
* cache_readOnly
* cache_size
* cache_type

```
<plugin type="org.mybatis.generator.plugins.CachePlugin">
	<property name="cache_eviction" value="LRU" />
	<property name="cache_size" value="1024" />
</plugin>
```
插件参考[这里](http://www.mybatis.org/generator/reference/plugins.html)

#### commentGenerator
该元素有一个可选属性`type`,可以指定用户的实现类，该类需要实现`org.mybatis.generator.api.CommentGenerator`接口。而且必有一个默认的构造方法。这个属性接收默认的特殊值`DEFAULT`，会使用默认的实现类`org.mybatis.generator.internal.DefaultCommentGenerator`

默认的实现类中提供了三个可选属性，需要通过<property>属性进行配置。

* suppressAllComments - 阻止生成注释，默认为false
* suppressDate - 阻止生成的注释包含时间戳，默认为false
* addRemarkComments - 注释是否添加数据库表的备注信息，默认为false

一般情况下由于MBG生成的注释信息没有任何价值，而且有时间戳的情况下每次生成的注释都不一样，使用**版本控制**的时候每次都会提交，因而一般情况下我们都会屏蔽注释信息，可以如下配置：

```
<commentGenerator>
	<property name="suppressAllComments" value="true"/>
	<property name="suppressDate" value="true"/>
</commentGenerator>
```
#### jdbcConnection
用于指定MBG要连接的数据库信息

配置该元素只需要注意如果JDBC驱动不在**classpath**下，就需要通过`<classPathEntry>`元素引入jar包，这里**推荐**将jar包放到**classpath**下，或者参考前面`<classPathEntry>`配置JDBC驱动的方法

该元素有两个必选属性:

* driverClass:访问数据库的JDBC驱动程序的完全限定类名
* connectionURL:访问数据库的JDBC连接URL

该元素还有两个可选属性:

* userId:访问数据库的用户ID
* password:访问数据库的密码

此外该元素还可以接受多个<property>子元素，这里配置的<property>属性都会添加到JDBC驱动的属性中。

```
<jdbcConnection driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://localhost:3306/test"
                userId="root"
                password="">
</jdbcConnection>
```

Oracle通过特殊配置获得列的注释信息

```
<jdbcConnection driverClass="oracle.jdbc.driver.OracleDriver"
                connectionURL="jdbc:oracle:thin://localhost:1521/test"
                userId="root"
                password="">
	<property name="remarksReporting" value="true" />
</jdbcConnection>
```
#### javaTypeResolver
用来指定JDBC类型和Java类型如何转换

该元素提供了一个可选的属性type，和`<commentGenerator>`比较类型，提供了默认的实现DEFAULT，一般情况下使用默认即可，需要特殊处理的情况可以通过其他元素配置来解决，不建议修改该属性。

该属性还有一个可以配置的`<property>`元素。可以配置的属性为`forceBigDecimals`，该属性可以控制是否强制DECIMAL和NUMERIC类型的字段转换为Java类型的`java.math.BigDecimal`,默认值为false，一般不需要配置。

默认情况下的转换规则为：

* 如果精度>0或者长度>18，就会使用java.math.BigDecimal
* 如果精度=0并且10<=长度<=18，就会使用java.lang.Long
* 如果精度=0并且5<=长度<=9，就会使用java.lang.Integer
* 如果精度=0并且长度<5，就会使用java.lang.Short

如果设置为true，那么一定会使用java.math.BigDecimal，配置示例如下：

```
<javaTypeResolver >
    <property name="forceBigDecimals" value="true" />
</javaTypeResolver>
```
#### javaModelGenerator
用来控制生成的实体类，根据`<context>`中配置的defaultModelType，一个表可能会对应生成多个不同的实体类。一个表对应多个类实际上并不方便，所以前面也推荐使用flat，这种情况下一个表对应一个实体类。

该元素只有两个属性，都是必选的。

* targetPackage:生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(<table>中会提到)
* targetProject:指定目标项目路径，可以是绝对路径或相对路径（如 targetProject="src/main/java"），该元素支持以下几个<property>子元素属性
	* constructorBased:该属性只对MyBatis3有效，如果true就会使用构造方法入参，如果false就会使用setter方式。默认为false。
	* enableSubPackages:如果true，MBG会根据catalog和schema来生成子包。如果false就会直接用targetPackage属性。默认为false。
	* immutable:该属性用来配置实体类属性是否可变，如果设置为true，那么constructorBased不管设置成什么，都会使用构造方法入参，并且不会生成setter方法。如果为false，实体类属性就可以改变。默认为false。
	* rootClass:设置所有实体类的基类。如果设置，需要使用类的全限定名称。并且如果MBG能够加载rootClass，那么MBG不会覆盖和父类中完全匹配的属性。匹配规则：

		* 属性名完全相同
		* 属性类型相同
		* 属性有getter方法
		* 属性有setter方法
	* trimStrings:是否对数据库查询结果进行trim操作，如果设置为true就会生成类似这样`public void setUsername(String username) {this.username = username == null ? null : username.trim();}`的setter方法。默认值为false。
配置示例如下：

```
<javaModelGenerator targetPackage="test.model" targetProject="src\main\java">
    <property name="enableSubPackages" value="true" />
    <property name="trimStrings" value="true" />
</javaModelGenerator>
```
#### sqlMapGenerator
用于配置Mapper.xml的属性。如果`targetRuntime`设置为MyBatis3，则只有当`javaClientGenerator`配置需要XML时，该标签才必须配置一个，如果没有配置`javaClientGenerator`，则使用以下规则

* 如果指定了一个sqlMapGenerator，那么MBG将只生成XML的SQL映射文件和实体类
* 如果没有指定sqlMapGenerator，那么MBG将只生成实体类

该标签有两个必选属性和一个可选属性

* targetPackage - 生成Mapper.xml文件存放的包名
* targetProject - 指定目标项目路径
* property（可选）- 子标签属性enableSubPackages如果为true，MBG会根据catalog和schema来生成子包，如果为false就会直接用targetPackage属性，默认为false

```
<sqlMapGenerator targetPackage="test.xml" targetProject="E:\MyProject\src\main\resources">
	<property name="enableSubPackage" value="false" />
</sqlMapGenerator>
```