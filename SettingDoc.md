# MBG详细配置

MBG的层次结构很严格，标签的顺序必须严格遵守。

## 配置文件头

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
```

## 根节点&#60;generatorConfiguration&#62;

generatorConfiguration节点没有任何属性，直接写节点即可，如下：

```xml
<generatorConfiguration>
    <!-- 具体配置内容 -->
</generatorConfiguration> 
```

**&#60;generatorConfiguration&#62;的子元素：**

从这段开始，就是配置的主要内容，这些配置都是generatorConfiguration元素的子元素。

### &#60;classPathEntry&#62; (0个或多个)

这个元素可以0或多个，不受限制。

最常见的用法是通过这个属性指定驱动的路径，例如：

```xml
<classPathEntry location="E:\mysql\mysql-connector-java-5.1.29.jar"/>
```

<font color=red>注意，classPathEntry只在下面这两种情况下才有效：

>- 当加载 JDBC 驱动内省数据库时
>- 当加载根类中的 JavaModelGenerator 检查重写的方法时</font>

因此，如果你需要加载其他用途的jar包，classPathEntry起不到作用，不能这么写，解决的办法就是将你用的jar包添加到类路径中，在Eclipse等IDE中运行的时候，添加jar包比较容易。当从命令行执行的时候，需要用java -cp xx.jar,xx2.jar xxxMainClass这种方式在-cp后面指定来使用(注意-jar会导致-cp无效)。
    
### &#60;context&#62; 元素

在MBG的配置中，至少需要有一个&#60;context&#62;元素。

&#60;context&#62;元素用于指定生成一组对象的环境。例如指定要连接的数据库，要生成对象的类型和要处理的数据库中的表。运行MBG的时候还可以指定要运行的&#60;context&#62;。

该元素只有一个**必选属性**id，用来唯一确定一个&#60;context&#62;元素，该id属性可以在运行MBG的使用。

此外还有几个**可选属性**：

1. defaultModelType：**这个属性很重要**，这个属性定义了MBG如何生成**实体类**。这个属性有以下可选值：

    1. conditional：**这是默认值**，这个模型和下面的hierarchical类似，除了如果那个单独的类将只包含一个字段，将不会生成一个单独的类。 因此,如果一个表的主键只有一个字段,那么不会为该字段生成单独的实体类,会将该字段合并到基本实体类中。
    
    2. flat：该模型为每一张表只生成一个实体类。这个实体类包含表中的所有字段。**这种模型最简单，推荐使用。**
    
    3. hierarchical：如果表有主键，那么该模型会产生一个单独的主键实体类，如果表还有BLOB字段， 则会为表生成一个包含所有BLOB字段的单独的实体类，然后为所有其他的字段生成一个单独的实体类。 MBG会在所有生成的实体类之间维护一个继承关系。

2. targetRuntime：此属性用于指定生成的代码的运行时环境。该属性支持以下可选值：

    1. MyBatis3:**这是默认值**
    
    2. MyBatis3Simple
    
    3. Ibatis2Java2
    
    4. Ibatis2Java5
    
    一般情况下使用默认值即可，有关这些值的具体作用以及区别请查看中文文档的详细内容。
    
3. introspectedColumnImpl：该参数可以指定扩展org.mybatis.generator.api.IntrospectedColumn该类的实现类。该属性的作用可以查看扩展MyBatis Generator。

一般情况下，我们使用如下的配置即可：

```xml
<context id="Mysql" defaultModelType="flat">
    ...
</context>
```
如果你希望不生成和Example查询有关的内容，那么可以按照如下进行配置：

```xml
<context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
    ...
</context>
```

使用MyBatis3Simple可以避免在后面的&#60;table&#62;中逐个进行配置（后面会提到）。

MBG配置中的其他几个元素，基本上都是&#60;context&#62;的子元素，这些子元素（有严格的配置顺序）包括：

1. &#60;property&#62; (0个或多个)

    &#60;property&#62;属性比较特殊，后面讲解的时候都会和父元素一起进行讲解。在讲解&#60;property&#62;属性前，我们先看看**什么是分隔符？**。
    
    这里通过一个例子说明。
    
    假设在Mysql数据库中有一个表名为user info，你没有看错，中间是一个空格，这种情况下如果写出select * from user info这样的语句，肯定是要报错的，在Mysql中的时候我们一般会写成如下的样子:
    
    ```sql
    select * from `user info`
    ```
    
    这里的使用的**反单引号(`)**就是**分隔符**，**分隔符**可以用于**表名**或者**列名**。
    
    下面继续看&#60;property&#62;支持的属性：
    
    - autoDelimitKeywords
    
    - beginningDelimiter
    
    - endingDelimiter

    - javaFileEncoding

    - javaFormatter

    - xmlFormatter
    
    由于这些属性比较重要，这里一一讲解。

    首先是autoDelimitKeywords，当表名或者字段名为SQL关键字的时候，可以设置该属性为true，MBG会自动给表名或字段名添加**分隔符**。

    然后这里继续上面的例子来讲beginningDelimiter和endingDelimiter属性。
    
    由于beginningDelimiter和endingDelimiter的默认值为双引号(")，在Mysql中不能这么写，所以还要将这两个默认值改为**反单引号(`)**，配置如下：
    
    ```xml
    <property name="beginningDelimiter" value="`"/>
    <property name="endingDelimiter" value="`"/>  
    ```
    
    属性javaFileEncoding设置要使用的Java文件的编码，默认使用当前平台的编码，只有当生产的编码需要特殊指定时才需要使用，一般用不到。
    
    最后两个javaFormatter和xmlFormatter属性**可能会**很有用，如果你想使用模板来定制生成的java文件和xml文件的样式，你可以通过指定这两个属性的值来实现。
    
    接下来分节对其他的子元素逐个进行介绍。

2. &#60;plugin&#62; (0个或多个)

    该元素可以配置0个或者多个，不受限制。
    
    &#60;plugin&#62;元素用来定义一个插件。插件用于扩展或修改通过MyBatis Generator(MBG)代码生成器生成的代码。

    插件将按在配置中配置的顺序执行。
    
    目前我用过的插件如下：
    
    ```xml
    <!-- 指定生成 Mapper 的继承模板 -->
    <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
        <property name="mappers" value="com.xxx.dao.BaseMapper" />
    </plugin>

    <!-- 生成 JavaBean 对象重写 toString方法 -->
    <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />

    <!-- 生成 JavaBean 对象继承 Serializable 类 -->
    <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
    ```
    
3. &#60;commentGenerator&#62; (0个或1个)

    该元素最多可以配置1个。
    
    这个元素非常有用，相信很多人都有过这样的需求，就是希望MBG生成的代码中可以包含**注释信息**，具体就是生成表或字段的备注信息。
    
    使用这个元素就能很简单的实现我们想要的功能。这里先介绍该元素，介绍完后会举例如何扩展实现该功能。
    
    该元素有一个可选属性type,可以指定用户的实现类，该类需要实现：org.mybatis.generator.api.CommentGenerator接口。而且必有一个默认的构造方法。这个属性接收默认的特殊值DEFAULT，会使用默认的实现类org.mybatis.generator.internal.DefaultCommentGenerator。
    
    默认的实现类中提供了两个可选属性，需要通过&#60;property&#62;属性进行配置：
    
    ```xml
    <commentGenerator>
        <!--阻止生成注释，默认为false-->
        <property name="suppressAllComments" value="true"/>
        <!--阻止生成的注释包含时间戳，默认为false-->
        <property name="suppressDate" value="true"/>
    </commentGenerator>
    ```
    一般情况下由于MBG生成的注释信息没有任何价值，而且有时间戳的情况下每次生成的注释都不一样，使用**版本控制**的时候每次都会提交，因而一般情况下我们都会屏蔽注释信息。

4. &#60;jdbcConnection&#62; (1个)

    &#60;jdbcConnection&#62;用于指定数据库连接信息，该元素必选，并且只能有一个。
    
    配置该元素只需要注意如果JDBC驱动不在**classpath**下，就需要通过&#60;classPathEntry&#62;元素引入jar包，这里**推荐**将jar包放到**classpath**下。
    
    该元素有两个必选属性：
    
    - driverClass：访问数据库的JDBC驱动程序的完全限定类名
    
    - connectionURL：访问数据库的JDBC连接URL
    
    该元素还有两个可选属性：
    
    - userId:访问数据库的用户ID
    
    - password:访问数据库的密码
    
    此外该元素还可以接受多个&#60;property&#62;子元素，这里配置的&#60;property&#62;属性都会添加到JDBC驱动的属性中。
    
    这个元素配置起来最容易，这里举个简单例子：
    
    ```xml
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                    connectionURL="jdbc:mysql://localhost:3306/test"
                    userId="root"
                    password="root">
    </jdbcConnection>
    ```

5. &#60;javaTypeResolver&#62; (0个或1个)

    该元素最多可以配置一个。
    
    这个元素的配置用来指定JDBC类型和Java类型如何转换。
    
    该元素提供了一个可选的属性type，和&#60;commentGenerator&#62;比较类型，提供了默认的实现DEFAULT，一般情况下使用默认即可，需要特殊处理的情况可以通过其他元素配置来解决，不建议修改该属性。
    
    该属性还有一个可以配置的&#60;property&#62;元素。
    
    可以配置的属性为forceBigDecimals，该属性可以控制是否强制DECIMAL和NUMERIC类型的字段转换为Java类型的java.math.BigDecimal，默认值为false，一般不需要配置。
    
    默认情况下的转换规则为：
    
    ```
    如果精度>0或者长度>18，就会使用java.math.BigDecimal
    如果精度=0并且10<=长度<=18，就会使用java.lang.Long
    如果精度=0并且5<=长度<=9，就会使用java.lang.Integer
    如果精度=0并且长度<5，就会使用java.lang.Short
    ```
    
    如果设置为true，那么一定会使用java.math.BigDecimal，配置示例如下：
    
    ```xml
    <javaTypeResolver >
        <property name="forceBigDecimals" value="true" />
    </javaTypeResolver>
    ```
    
6. &#60;javaModelGenerator&#62; (1个)

    该元素必须配置一个，并且最多一个。
    
    该元素用来控制生成的实体类，根据&#60;context&#62;中配置的defaultModelType，一个表可能会对应生成多个不同的实体类。一个表对应多个类实际上并不方便，所以前面也推荐使用flat，这种情况下一个表对应一个实体类。
    
    该元素只有两个属性，都是必选的。
    
    - targetPackage：生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(&#60;table&#62;中会提到)。
    
    - targetProject：指定目标项目路径，可以是绝对路径或相对路径（如 targetProject="src/main/java"）。
    
    该元素支持以下几个&#60;property&#62;子元素属性：  
    
    - constructorBased:该属性只对MyBatis3有效，如果true就会使用构造方法入参，如果false就会使用setter方式。默认为false。
    
    - enableSubPackages:如果true，MBG会根据catalog和schema来生成子包。如果false就会直接用targetPackage属性。默认为false。
    
    - immutable:该属性用来配置实体类属性是否可变，如果设置为true，那么constructorBased不管设置成什么，都会使用构造方法入参，并且不会生成setter方法。如果为false，实体类属性就可以改变。默认为false。
    
    - rootClass:设置所有实体类的基类。如果设置，需要使用类的全限定名称。并且如果MBG能够加载rootClass，那么MBG不会覆盖和父类中完全匹配的属性。匹配规则：
    
        - 属性名完全相同
        
        - 属性类型相同
        
        - 属性有getter方法
        
        - 属性有setter方法
        
    - trimStrings:是否对数据库查询结果进行trim操作，如果设置为true就会生成类似这样public void setUsername(String username) {this.username = username == null ? null : username.trim();}的setter方法。默认值为false。          
    
    配置示例如下：
    
    ```xml
    <javaModelGenerator targetPackage="test.model" targetProject="src\main\java">
        <property name="enableSubPackages" value="true" />
        <property name="trimStrings" value="true" />
    </javaModelGenerator>
    ```

7. &#60;sqlMapGenerator&#62; (0个或1个)

    该元素可选，最多配置一个。但是有如下两种必选的特殊情况：
    
    - 如果targetRuntime目标是**iBATIS2**，该元素必须配置一个。
    
    - 如果targetRuntime目标是**MyBatis3**，只有当&#60;javaClientGenerator&#62;需要XML时，该元素必须配置一个。 如果没有配置&#60;javaClientGenerator&#62;，则使用以下的规则：
    
        - 如果指定了一个&#60;sqlMapGenerator&#62;，那么MBG将只生成XML的SQL映射文件和实体类。
        
        - 如果没有指定&#60;sqlMapGenerator&#62;，那么MBG将只生成实体类。
        
    该元素只有两个属性（和前面提过的&#60;javaModelGenerator&#62;的属性含义一样），都是必选的。
    
    - targetPackage：生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(&#60;table&#62;中会提到)。
    
    - targetProject：指定目标项目路径，可以是绝对路径或相对路径（如 targetProject="src/main/resources"）。
    
    该元素支持&#60;property&#62;子元素，只有一个可以配置的属性：
    
    - enableSubPackages:如果true，MBG会根据catalog和schema来生成子包。如果false就会直接用targetPackage属性。默认为false。
    
    配置示例：
    
    ```xml
    <sqlMapGenerator targetPackage="test.xml"  targetProject="src\main\resources">
        <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>
    ```

8. &#60;javaClientGenerator&#62; (0个或1个)

    该元素可选，最多配置一个。

    如果不配置该元素，就不会生成Mapper接口。

    该元素有3个必选属性：
    
    - type：该属性用于选择一个预定义的客户端代码（可以理解为Mapper接口）生成器，用户可以自定义实现，需要继承org.mybatis.generator.codegen.AbstractJavaClientGenerator类，必选有一个默认的构造方法。 该属性提供了以下预定的代码生成器，首先根据&#60;context&#62;的targetRuntime分成三类：
    
        - MyBatis3：
            
            - ANNOTATEDMAPPER：基于注解的Mapper接口，不会有对应的XML映射文件

            - MIXEDMAPPER：XML和注解的混合形式，(上面这种情况中的)SqlProvider注解方法会被XML替代。
            
            - XMLMAPPER：所有的方法都在XML中，接口调用依赖XML文件。
    
        - MyBatis3Simple:
        
            - ANNOTATEDMAPPER：基于注解的Mapper接口，不会有对应的XML映射文件

            - XMLMAPPER：所有的方法都在XML中，接口调用依赖XML文件。

        - Ibatis2Java2或**Ibatis2Java5**:

            - IBATIS：生成的对象符合iBATIS的DAO框架（不建议使用）。

            - GENERIC-CI：生成的对象将只依赖于SqlMapClient，通过构造方法注入。

            - GENERIC-SI：生成的对象将只依赖于SqlMapClient，通过setter方法注入。

            - SPRING：生成的对象符合Spring的DAO接口
            
        - targetPackage:生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(&#60;table&#62;中会提到)。
            
        - targetProject:指定目标项目路径，可以是绝对路径或相对路径（如 targetProject="src/main/java"）。
        
        该元素还有一个可选属性：

        - implementationPackage：如果指定了该属性，实现类就会生成在这个包中。
        
        该元素支持&#60;property&#62;子元素设置的属性：

        - enableSubPackages

        - exampleMethodVisibility

        - methodNameCalculator

        - rootInterface

        - useLegacyBuilder
        
        这几个属性不太常用，具体作用请看完整的文档，这里对rootInterface做个简单介绍。
        
        rootInterface用于指定一个所有生成的接口都继承的父接口。 这个值可以通过&#60;table&#62;配置的rootInterface属性覆盖。
        
        这个属性对于通用Mapper来说，可以让生成的所有接口都继承该接口。
        
        配置示例：
        
        ```xml
        <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"
            targetProject="src\main\java"/>
        ```

9. &#60;table&#62; (1个或多个)

    该元素至少要配置一个，可以配置多个。

    该元素用来配置要通过内省的表。只有配置的才会生成实体类和其他文件。

    该元素有一个必选属性：

    tableName：指定要生成的表名，可以使用SQL通配符匹配多个表。

    例如要生成全部的表，可以按如下配置：
    
    ```xml
    <table tableName="%" />
    ```
    
    该元素包含多个可选属性：
    
    - schema：数据库的schema,可以使用SQL通配符匹配。如果设置了该值，生成SQL的表名会变成如schema.tableName的形式。

    - catalog：数据库的catalog，如果设置了该值，生成SQL的表名会变成如catalog.tableName的形式。

    - alias：如果指定，这个值会用在生成的select查询SQL的表的别名和列名上。 列名会被别名为 alias_actualColumnName(别名_实际列名) 这种模式。
    
    - domainObjectName：生成对象的基本名称。如果没有指定，MBG会自动根据表名来生成名称。

    - enableXXX：XXX代表多种SQL方法，该属性用来指定是否生成对应的XXX语句。

    - selectByPrimaryKeyQueryId：DBA跟踪工具会用到，具体请看详细文档。
    
    - selectByExampleQueryId：DBA跟踪工具会用到，具体请看详细文档。

    - modelType：和&#60;context&#62;的defaultModelType含义一样，这里可以针对表进行配置，这里的配置会覆盖&#60;context&#62;的defaultModelType配置。
    
    - escapeWildcards：这个属性表示当查询列，是否对schema和表名中的SQL通配符 ('_' and '%') 进行转义。 对于某些驱动当schema或表名中包含SQL通配符时（例如，一个表名是MY_TABLE，有一些驱动需要将下划线进行转义）是必须的。默认值是false。

    - delimitIdentifiers：是否给标识符增加**分隔符**。默认false。当catalog,schema或tableName中包含空白时，默认为true。

    - delimitAllColumns：是否对所有列添加**分隔符**。默认false。
    
    该元素包含多个可用的&#60;property&#62;子元素，可选属性为：
    
    - constructorBased：和&#60;javaModelGenerator&#62;中的属性含义一样。
    
    - ignoreQualifiersAtRuntime：生成的SQL中的表名将不会包含schema和catalog前缀。

    - immutable：和&#60;javaModelGenerator&#62;中的属性含义一样。

    - modelOnly：此属性用于配置是否为表只生成实体类。如果设置为true就不会有Mapper接口。如果配置了&#60;sqlMapGenerator&#62;，并且modelOnly为true，那么XML映射文件中只有实体对象的映射元素(&#60;resultMap&#62;)。如果为true还会覆盖属性中的enableXXX方法，将不会生成任何CRUD方法。

    - rootClass：和&#60;javaModelGenerator&#62;中的属性含义一样。

    - rootInterface：和&#60;javaClientGenerator&#62;中的属性含义一样。

    - runtimeCatalog：运行时的catalog，当生成表和运行环境的表的catalog不一样的时候可以使用该属性进行配置。

    - runtimeSchema：运行时的schema，当生成表和运行环境的表的schema不一样的时候可以使用该属性进行配置。

    - runtimeTableName：运行时的tableName，当生成表和运行环境的表的tableName不一样的时候可以使用该属性进行配置。

    - selectAllOrderByClause：该属性值会追加到selectAll方法后的SQL中，会直接跟order by拼接后添加到SQL末尾。

    - useActualColumnNames：如果设置为true,那么MBG会使用从数据库元数据获取的列名作为生成的实体对象的属性。 如果为false(默认值)，MGB将会尝试将返回的名称转换为驼峰形式。 在这两种情况下，可以通过 元素显示指定，在这种情况下将会忽略这个（useActualColumnNames）属性。

    - useColumnIndexes：如果是true，MBG生成resultMaps的时候会使用列的索引，而不是结果中列名的顺序。

    - useCompoundPropertyNames：如果是true，那么MBG生成属性名的时候会将列名和列备注接起来。 这对于那些通过第四代语言自动生成列(例如：FLD22237)，但是备注包含有用信息(例如："customer id")的数据库来说很有用。在这种情况下，MBG会生成属性名FLD2237_CustomerId。
    
    除了&#60;property&#62;子元素外，&#60;table&#62;还包含以下子元素：
    
    1.  &#60;generatedKey&#62; (0个或1个)
    
        这个元素最多可以配置一个。
        
        这个元素用来指定自动生成主键的属性（identity字段或者sequences序列）。
        
        如果指定这个元素，MBG在生成insert的SQL映射文件中插入一个&#60;selectKey&#62;元素。 这个元素**非常重要**，这个元素包含下面两个必选属性：
        
        - column：生成列的列名。
        
        - sqlStatement:将返回新值的 SQL 语句。如果这是一个identity列，您可以使用其中一个预定义的的特殊值。预定义值如下：
        
            - Cloudscape
            
            - DB2
            
            - DB2_MF
            
            - Derby
            
            - HSQLDB
            
            - Informix
            
            - MySql
            
            - SqlServer
            
            - SYBASE
            
            - JDBC
            
                这会配置MBG使用MyBatis3支持的JDBC标准的生成key来生成代码。
                
                这是一个独立于数据库获取标识列中的值的方法。
                
                重要: 只有当目标运行为MyBatis3时才会产生正确的代码。 
                
                如果与iBATIS2一起使用目标运行时会产生运行时错误的代码。                
                
            这个元素还包含两个可选属性：
            
            - identity：当设置为true时,该列会被标记为identity列， 并且&#60;selectKey&#62;元素会被插入在insert后面。 当设置为false时，&#60;selectKey&#62;会插入到insert之前（通常是序列）。**重要**: 即使您type属性指定为post，您仍然需要为identity列将该参数设置为true。 这将标志MBG从插入列表中删除该列。默认值是false。
            
            - type：type=post and identity=true的时候生成的&#60;selectKey&#62;中的order=AFTER,当type=pre的时候，identity只能为false，生成的&#60;selectKey&#62;中的order=BEFORE。可以这么理解，自动增长的列只有插入到数据库后才能得到ID，所以是AFTER,使用序列时，只有先获取序列之后，才能插入数据库，所以是BEFORE
    
    2. &#60;columnRenamingRule&#62; (0个或1个)
    
        该元素最多可以配置一个，使用该元素可以在生成列之前，对列进行重命名。这对那些存在同一前缀的字段想在生成属性名时去除前缀的表非常有用。 例如假设一个表包含以下的列：
        
        - CUST_BUSINESS_NAME
        
        - CUST_STREET_ADDRESS
        
        - CUST_CITY

        - CUST_STATE
        
        生成的所有属性名中如果都包含CUST的前缀可能会让人不爽。这些前缀可以通过如下方式定义重命名规则:

        ```xml
        <columnRenamingRule searchString="^CUST_" replaceString="" />
        ```
    
        注意，在内部，MBG使用java.util.regex.Matcher.replaceAll方法实现这个功能。请参阅有关该方法的文档和在Java中使用正则表达式的例子。
        
        当&#60;columnOverride&#62;匹配一列时，这个元素（&#60;columnRenamingRule&#62;）会被忽略。&#60;columnOverride&#62;优先于重命名的规则。

        该元素有一个必选属性：

        - searchString：定义将被替换的字符串的正则表达式。

        该元素有一个可选属性：

        - replaceString：这是一个用来替换搜索字符串列每一个匹配项的字符串。如果没有指定，就会使用空字符串。

        关于&#60;table&#62;的&#60;property&#62;属性useActualColumnNames对此的影响可以查看完整文档。
    
    3. &#60;columnOverride&#62; (0个或多个)
    
        该元素可选，可以配置多个。

        该元素从将某些属性默认计算的值更改为指定的值。
        
        该元素有一个必选属性:
        
        - column：要重写的列名。
        
        该元素有多个可选属性：
        
        - property：要使用的Java属性的名称。如果没有指定，MBG会根据列名生成。 例如，如果一个表的一列名为STRT_DTE，MBG会根据&#60;table&#62;的useActualColumnNames属性生成STRT_DTE或strtDte。

        - javaType：该列属性值为完全限定的Java类型。如果需要，这可以覆盖由JavaTypeResolver计算出的类型。 对某些数据库来说，这是必要的用来处理**“奇怪的”**数据库类型（例如MySql的unsigned bigint类型需要映射为java.lang.Object)。
        
        - jdbcType：该列的JDBC类型(INTEGER, DECIMAL, NUMERIC, VARCHAR等等)。 如果需要，这可以覆盖由JavaTypeResolver计算出的类型。 对某些数据库来说，这是必要的用来处理怪异的JDBC驱动 (例如DB2的LONGVARCHAR类型需要为iBATIS 映射为VARCHAR)。

        - typeHandler：用户定义的需要用来处理这列的类型处理器。它必须是一个继承iBATIS的TypeHandler类或TypeHandlerCallback接口（该接口很容易继承）的全限定的类名。如果没有指定或者是空白，iBATIS会用默认的类型处理器来处理类型。**重要**:MBG不会校验这个类型处理器是否存在或者可用。 MGB只是简单的将这个值插入到生成的SQL映射的配置文件中。
        
        - delimitedColumnName：指定是否应在生成的SQL的列名称上增加**分隔符**。 如果列的名称中包含空格，MGB会自动添加**分隔符**， 所以这个重写只有当列名需要强制为一个合适的名字或者列名是数据库中的保留字时是必要的。

        配置示例：
        
        ```xml
        <table schema="DB2ADMIN" tableName="ALLTYPES" >
            <columnOverride column="LONG_VARCHAR_FIELD" javaType="java.lang.String" jdbcType="VARCHAR" />
        </table>
        ```
    
    4. &#60;ignoreColumn&#62; (0个或多个)
    
        该元素可选，可以配置多个。

        该元素可以用来屏蔽不需要生成的列。
        
        该元素有一个必选属性：
        
        - column：要忽略的列名。
        
        该元素还有一个可选属性：

        - delimitedColumnName：匹配列名的时候是否区分大小写。如果为true则区分。默认值为false，不区分大小写。
                
----------

本文转载于 [MyBatis Generator 详解](https://blog.csdn.net/isea533/article/details/42102297) 并二次修改。 

----------

