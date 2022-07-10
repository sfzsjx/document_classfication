

## idea多模块显示问题



![1618626734697](idea多模块显示问题.assets/1618626734697.png)

## 1. 先找到顶级的工程，打开.idea目录下的workspace.xml

![1618626833242](idea多模块显示问题.assets/1618626833242.png)

## 2. 添加下面的模块

```bash
<component name="RunDashboard">
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
    <option name="ruleStates">
      <list>
        <RuleState>
          <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
        </RuleState>
        <RuleState>
          <option name="name" value="StatusDashboardGroupingRule" />
        </RuleState>
      </list>
    </option>
  </component>
```



## 3. 重启的就可以了