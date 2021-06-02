上一篇介绍自己写`XML`模板的文章，日常开发迭代快，可能需要高效的完成 `CRUD`。这里简单介绍两种常见的生成 `Mapper接口` `Model模型` `MapperXML` 。
首先建一个简单的工程 `mybatis-demo`，`pom.xml` 文件引入相关组件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>mybatis-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.5</version>
                <configuration>
                    <configurationFile>src/main/resources/mybatis.xml</configurationFile>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.3.5</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```
接着在`resources` 文件新建 `mybatis.xml` 配置生成规则，这里注意`XML`中的`&`需要转义`&amp;`
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>

    <!-- 本地数据库驱动程序jar包的全路径 -->
    <classPathEntry location="C:/Users/康盼Java开发工程师/.m2/repository/mysql/mysql-connector-java/8.0.15/mysql-connector-java-8.0.15.jar"/>

    <context id="context" targetRuntime="MyBatis3">

        <!--定义生成的java类的编码格式-->
        <property name="javaFileEncoding" value="UTF-8" />

        <!-- 数据库的相关配置 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/kangpan?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8&amp;serverTimezone=UTC&amp;useAffectedRows=true" userId="root" />

        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和
           NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!-- 实体类生成的位置 -->
        <javaModelGenerator targetPackage="demo.generator" targetProject="src/main/java">
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- *Mapper.xml 文件的位置 -->
        <sqlMapGenerator targetPackage="demo.generator" targetProject="src/main/java">
        </sqlMapGenerator>

        <!-- Mapper 接口文件的位置 -->
        <javaClientGenerator targetPackage="demo.generator" targetProject="src/main/java" type="XMLMAPPER">
        </javaClientGenerator>

        <!-- 相关表的配置   tableName 表名 -->
        <table tableName="t_user" enableSelectByExample="true"
               enableDeleteByExample="true" enableCountByExample="true"
               enableUpdateByExample="true" selectByExampleQueryId="true">
            <property name="ignoreQualifiersAtRuntime" value="false" />
            <property name="useActualColumnNames" value="false" />
        </table>
    </context>
</generatorConfiguration>
```
点击`IDEA` 右侧栏里面的 `Maven -> Plugins `双击运行。
第二种方式，直接上代码 `pom` 文件
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>mybatis-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.7</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
`resources` 中新建 `generatorConfig.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- classPathEntry:数据库的JDBC驱动-->
    <classPathEntry location="C:/Users/康盼Java开发工程师/.m2/repository/mysql/mysql-connector-java/8.0.15/mysql-connector-java-8.0.15.jar" />
    <context id="DBTables" targetRuntime="MyBatis3">
        <plugin type="org.mybatis.generator.plugins.FluentBuilderMethodsPlugin" />
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
        <plugin type="org.mybatis.generator.plugins.RowBoundsPlugin" />
        <!-- 去除自动生成的注释 -->
        <commentGenerator><property name="suppressAllComments" value="true" /></commentGenerator>

        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/kangpan?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8&amp;serverTimezone=UTC&amp;useAffectedRows=true"
                        userId="root"
                        password="">
        </jdbcConnection>

        <javaTypeResolver >
        <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!-- targetProject:自动生成model代码的位置 -->
        <javaModelGenerator targetPackage="demo.generator.model"
                            targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- targetProject:自动生成mappers代码的位置 -->
        <sqlMapGenerator targetPackage="demo.generator"
                         targetProject="./src/main/resources/mapper">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- targetProject:自动生成dao代码的位置 -->
        <javaClientGenerator type="MIXEDMAPPER"
                             targetPackage="demo.generator.mapper"
                             targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- tableName:用于自动生成代码的数据库表;domainObjectName:对应于数据库表的javaBean类名-->
        <table tableName="t_coffee" domainObjectName="Coffee" >

        </table>
    </context>
</generatorConfiguration>
```
启动类中添加
```
@SpringBootApplication
@MapperScan("demo.mapper")
public class Application implements ApplicationRunner {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
       mybatisGenerator();
    }

    private void mybatisGenerator() throws Exception{
        List<String> warnings = new ArrayList<>();
        ConfigurationParser configurationParser = new ConfigurationParser(warnings);
        Configuration configuration = configurationParser
                .parseConfiguration(this.getClass().getResourceAsStream("/generatorConfig.xml"));
        DefaultShellCallback defaultShellCallback = new DefaultShellCallback(true);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(configuration, defaultShellCallback, warnings);
        myBatisGenerator.generate(null);
    }
}
```
因为 `pom` 文件引入了 `mybatis-spring-boot-starter`组件，需要在`application.properties`文件中配置数据源
```
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/kangpan?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=UTC&useAffectedRows=true
spring.datasource.username=root
spring.datasource.password=
mybatis.mapper-locations=classpath*:/mapper/**/*.xml
```
