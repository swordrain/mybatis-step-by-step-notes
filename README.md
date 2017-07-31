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
### if
