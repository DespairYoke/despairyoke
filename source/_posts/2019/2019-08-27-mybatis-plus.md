---
layout: post
title: springboot2.0使用mybatis-plus
category: springboot
tags:
    - springboot
    - mybatis-plus
---

[参考官网demo](https://github.com/baomidou/mybatis-plus-samples)
### 代码生成器
```java
package com.baomidou.mybatisplus.samples.generator;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.FileOutConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.TemplateConfig;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

/**
 * <p>
 * mysql 代码生成器演示例子
 * </p>
 *
 * @author jobob
 * @since 2018-09-12
 */
public class MysqlGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    /**
     * RUN THIS
     */
    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/mybatis-plus-sample-generator/src/main/java");
        gc.setAuthor("jobob");
        gc.setOpen(false);
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://47.96.127.51:3306/test?useUnicode=true&serverTimezone=GMT&useSSL=false&characterEncoding=utf8");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("adminadmin");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
//        pc.setModuleName(scanner("模块名"));
        pc.setParent("com.baomidou.mybatisplus.samples.generator");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        List<FileOutConfig> focList = new ArrayList<>();
        focList.add(new FileOutConfig("/templates/mapper.xml.ftl") {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输入文件名称
                return projectPath + "/mybatis-plus-sample-generator/src/main/resources/mapper/"
                        + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);
        mpg.setTemplate(new TemplateConfig().setXml(null));

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
//        strategy.setSuperEntityClass("com.baomidou.mybatisplus.samples.generator.common.BaseEntity");
        strategy.setEntityLombokModel(true);
//        strategy.setSuperControllerClass("com.baomidou.mybatisplus.samples.generator.common.BaseController");
//        strategy.setInclude(scanner("表名"));
        strategy.setSuperEntityColumns("id");
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        // 选择 freemarker 引擎需要指定如下加，注意 pom 依赖必须有！
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}

```

### insert
```java
    @Resource
    private UserMapper mapper;

    @Test
    public void aInsert() {
        User user = new User();
        user.setName("小羊");
        user.setAge(3);
        user.setEmail("abc@mp.com");
        assertThat(mapper.insert(user)).isGreaterThan(0);
        // 成功直接拿会写的 ID
        assertThat(user.getId()).isNotNull();
    }
```
### select
```java
@Test
public void dSelect() {
    mapper.insert(
            new User().setId(10086L)
                    .setName("miemie")
                    .setEmail("miemie@baomidou.com")
                    .setAge(3));
    assertThat(mapper.selectById(10086L).getEmail()).isEqualTo("miemie@baomidou.com");
    User user = mapper.selectOne(new QueryWrapper<User>().lambda().eq(User::getId, 10086));
    assertThat(user.getName()).isEqualTo("miemie");
    assertThat(user.getAge()).isEqualTo(3);

    mapper.selectList(Wrappers.<User>lambdaQuery().select(User::getId))
            .forEach(x -> {
                assertThat(x.getId()).isNotNull();
                assertThat(x.getEmail()).isNull();
                assertThat(x.getName()).isNull();
                assertThat(x.getAge()).isNull();
            });
    mapper.selectList(new QueryWrapper<User>().select("id", "name"))
            .forEach(x -> {
                assertThat(x.getId()).isNotNull();
                assertThat(x.getEmail()).isNull();
                assertThat(x.getName()).isNotNull();
                assertThat(x.getAge()).isNull();
            });
}
```

### update
```java
@Test
public void cUpdate() {
    assertThat(mapper.updateById(new User().setId(1L).setEmail("ab@c.c"))).isGreaterThan(0);
    assertThat(
            mapper.update(
                    new User().setName("mp"),
                    Wrappers.<User>lambdaUpdate()
                            .set(User::getAge, 3)
                            .eq(User::getId, 2)
            )
    ).isGreaterThan(0);
    User user = mapper.selectById(2);
    assertThat(user.getAge()).isEqualTo(3);
    assertThat(user.getName()).isEqualTo("mp");

    mapper.update(
            null,
            Wrappers.<User>lambdaUpdate().set(User::getEmail, null).eq(User::getId, 2)
    );
    assertThat(mapper.selectById(1).getEmail()).isEqualTo("ab@c.c");
    user = mapper.selectById(2);
    assertThat(user.getEmail()).isNull();
    assertThat(user.getName()).isEqualTo("mp");

    mapper.update(
            new User().setEmail("miemie@baomidou.com"),
            new QueryWrapper<User>()
                    .lambda().eq(User::getId, 2)
    );
    user = mapper.selectById(2);
    assertThat(user.getEmail()).isEqualTo("miemie@baomidou.com");

    mapper.update(
            new User().setEmail("miemie2@baomidou.com"),
            Wrappers.<User>lambdaUpdate()
                    .set(User::getAge, null)
                    .eq(User::getId, 2)
    );
    user = mapper.selectById(2);
    assertThat(user.getEmail()).isEqualTo("miemie2@baomidou.com");
    assertThat(user.getAge()).isNull();
}
```

### delete
```java
@Test
public void bDelete() {
    assertThat(mapper.deleteById(3L)).isGreaterThan(0);
    assertThat(mapper.delete(new QueryWrapper<User>()
            .lambda().eq(User::getName, "Sandy"))).isGreaterThan(0);
}
```