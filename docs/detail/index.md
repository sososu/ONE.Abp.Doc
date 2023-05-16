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
