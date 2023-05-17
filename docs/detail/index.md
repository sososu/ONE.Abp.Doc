#

# **快速开始**

首先,如果你没有安装[ONEABP CLI](https://docs.abp.io/zh-Hans/abp/latest/CLI),请先安装它:

    dotnet tool install -g ONE.Abp.Cli

在一个空文件夹使用 `abp new` 命令创建新解决方案:

base模板

    oneabp new Acme.BookStore -t base -d ef -dbms postgresql

micro模板

    oneabp new Acme.BookStore -t micro -d ef -dbms postgresql

ONEABP.CLI在ABP.CLI基础上增加了两个模板

1.base模板: 解决方案包含网关，认证服务和基础服务项目

2.micro模板:解决方案包含微服务项目

# **租户管理**

待完善

# **文件管理**

待完善

# **数据字典管理**

*第一种 配置方式*

```c#
 Configure<AbpEnumDicOption>(option =>
            {
                option.Add<Sex>("性别");
            });
```

*第二种 注册方式*

```c#
  public class ONEEnumDataItemRegisterion : EnumDataItemRegisterionBase
    {
        public override void Register(AbpEnumDicOption option)
        {
            option.Add<Sex>("性别");
        }
    }
```

*注意：base模板和micro模板已经引用该模块*

# **数据权限管理**

*在 Domain 层引用 ONE.Abp.DataPermission NuGet 包并且添加\[DependsOn(typeof(AbpDataPermissionModule))]模块*

## **配置规则**

```c#
 Configure<AbpRuleOptions>(option =>
            {
                option.DataTargetOption.Add<Customer>(needUpdateShadowProperty:true);
            });
```

***这里是你可以配置的选项列表:***

*   DataTargetOption：添加数据源对象

    <span style="font-size:10px;">参数 needUpdateShadowProperty(默认值 false):是否需要更新影子属性，true 为更新</span>

*   RuleExtraFieldManager：规则扩展管理，下面介绍

*   IsReleasedIfNoRulesAreMatched(默认值 true)：在没有匹配任何规则下是否放行，true 为放行

**使用规则 ToPagedResultForRuleAsync 方法**

*在 Application 层引用 ONE.Abp.DataPermission.Extension NuGet 包并且添加\[DependsOn(typeof(AbpDataPermissionExtensionModule))]模块*

```c#
   await (await CustomerRepository.WithDetailsAsync()).ToPagedResultForRuleAsync<Customer, CustomerDto>(input);
```

**规则扩展**

**扩展用户规则属性** 如扩展组织编码属性

```c#

            Configure<AbpRuleOptions>(option =>
            {
                option.RuleExtraFieldManager.AddUserExtraProperty("OrganizationCode");
            });
```

扩展数据规则预定义值

*首先定义 IOrganizationCode 接口，需要继承自 IShadowProperty 接口*

```c#

            Configure<AbpRuleOptions>(option =>
            {
               option.RuleExtraFieldManager.AddDataExtraProperty<IOrganizationCode>("OrganizationCode");
            });
```

*注意：base模板已经引用该模块,micro模板按需引用*

# **应用管理**

待完善

# **管理设置**

待完善

# **分页查询**

查询应该是开发中比较高频的使用，ONE.Abp提供了简单快捷的查询实现方式

1.接口层引用ONE.Abp.Pagination.Contracts

     [DependsOn( typeof(AbpPaginationContractsModule) )]

2.应用层引用ONE.Abp.Pagination

     [DependsOn( typeof(AbpPaginationModule) )]

> *注意：使用模板创建项目自带了分页引用*

## **定义请求参数类**

       public class SampleQueryInput:PagedQuery
        {
            [Query(Compare =ONE.Abp.Shared.QueryCompare.Like)] 
            public string? Name { get; set; }

            [Query("CreationTime", Compare = ONE.Abp.Shared.QueryCompare.GreaterThanOrEqual)]
            [Query(Compare = ONE.Abp.Shared.QueryCompare.Like)]
            public DateTime? Start { get; set; }

            [Query("CreationTime", Compare = ONE.Abp.Shared.QueryCompare.LessThan)]
            public DateTime? End { get; set; }


            [Query(Compare = ONE.Abp.Shared.QueryCompare.Equal)]
            public int? Value { get; set; } 
        }

### **要点：**

*   参数类要继承PagedQuery
*   定义可空类型表示值为空在查询时忽略该参数条件
*   QueryAttribute特性

    > *PropertyPath属性表示映射的字段名，如果参数名跟字段名一致可以不填*
    >
    > *Compare属性表示比较方式，该枚举值覆盖了大多数通用的比较方式。*
*   参数条件之间默认是与运算，如果需要采用或运算，比如 Name跟Value是或运算，那么则使用OrGroup属性，如下示例：

        public class SampleQueryInput:PagedQuery
        {
            [Query(Compare =ONE.Abp.Shared.QueryCompare.Like,OrGroup ="cc")] 
            public string? Name { get; set; }

            [Query("CreationTime", Compare = ONE.Abp.Shared.QueryCompare.GreaterThanOrEqual)]
            [Query(Compare = ONE.Abp.Shared.QueryCompare.Like)]
            public DateTime? Start { get; set; }

            [Query("CreationTime", Compare = ONE.Abp.Shared.QueryCompare.LessThan)]
            public DateTime? End { get; set; }


            [Query(Compare = ONE.Abp.Shared.QueryCompare.Equal, OrGroup = "cc")]
            public int? Vaule { get; set; } 
        }

## **查询使用**

    public async Task<PagedResult<SampleDto>> GetQueryAsync(SampleQueryInput input)
    {
        return await (await _repository.WithDetailsAsync()).ToPagedResultAsync<Sample,SampleDto>(input);
    }

> ToPagedResultAsync方法是IQueryable<>的静态扩展方法。

> *注意：需要返回非实体类时，记得对象映射 CreateMap< Sample, SampleDto >();*

