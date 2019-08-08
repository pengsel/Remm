# Mybatis Generator

MyBatis属于半自动ORM，可以通过generator自动生成DAO、Model、Mapping文件

### 相关文件

1. MySQL数据库的驱动包
2. 配置文件：generatorConfig.xml

## mybatis-generator-maven-plugin插件生成实体类

1. 在pom.xml添加插件支持

```xml

    <build>
        <finalName>mavenweb</finalName>
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
            </plugin>
        </plugins>
     </build>

```

2. 添加generatorConfig.xml

```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE generatorConfiguration
           PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
           "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
   <!--mybatis-generator-maven-plugin生成数据库实体的配置文件-->
   <generatorConfiguration>
       <!--导入属性配置，前面我们写的一个配置文件，你也可以直接使用mybatis的jdbc的配置文件 -->
       <properties resource="jdbc.properties"></properties>
       <!-- 数据库驱动，注意，这里必须要修改成你的数据库的驱动地址-->
       <classPathEntry
               location="/home/admin/.m2/repository/mysql/mysql-connector-java/5.1.30/mysql-connector-java-5.1.30.jar"/>
       <context id="DB2Tables" targetRuntime="MyBatis3">
           <commentGenerator>
               <property name="suppressDate" value="true"/>
               <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
               <property name="suppressAllComments" value="true"/>
           </commentGenerator>
           <!--数据库链接URL，用户名、密码 -->
           <jdbcConnection driverClass="${driver}" connectionURL="${url}" userId="${username}" password="${password}">
           </jdbcConnection>
           <javaTypeResolver>
               <property name="forceBigDecimals" value="false"/>
           </javaTypeResolver>
           <!-- 生成模型的包名和位置-->
           <javaModelGenerator targetPackage="com.dingliqc.web.entity" targetProject="src/main/java">
               <property name="enableSubPackages" value="true"/>
               <property name="trimStrings" value="true"/>
           </javaModelGenerator>
           <!-- 生成映射文件的包名和位置-->
           <sqlMapGenerator targetPackage="com.dingliqc.web.mapping" targetProject="src/main/java">
               <property name="enableSubPackages" value="true"/>
           </sqlMapGenerator>
           <!-- 生成DAO的包名和位置-->
           <javaClientGenerator type="XMLMAPPER" targetPackage="com.dingliqc.web.mapper" targetProject="src/main/java">
               <property name="enableSubPackages" value="true"/>
           </javaClientGenerator>
    
           <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名，这里举例，只配置了一个table，你可以配置多个-->
    
           <table tableName="表名字" domainObjectName="生成实体的名字" enableCountByExample="false"
                  enableUpdateByExample="false"
                  enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"/>
    
       </context>
   </generatorConfiguration> 
```

   

3. 添加jdbc.properties，存储一些数据库链接信息

```

#驱动名字
driver=com.mysql.jdbc.Driver
#数据库地址
url=jdbc:mysql://127.0.0.1:3306/test
#数据库用户名
username=root
#数据库密码
password=root
```



4. 在maven菜单中找到mybatis-generator插件运行