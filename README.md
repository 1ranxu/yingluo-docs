# 樱络 - 伙伴匹配系统

简介:帮助大家找到志同道合的伙伴，移动端的H5网页（尽量兼容PC端）

用户可以去用户中心注册，登录用户中心，每个用户可以打上一些标签，伙伴匹配系统根据标签帮家找到志同道合的伙伴

## 需求分析

1. 用户给自己添加标签；修改标签，系统根据标签对用户进行分类（有哪些标签，如何对标签进行分类）分类比如：学习方向，就业状态，理想城市，工作，大学等
2. 主动搜索：允许用户根据标签去搜索其他用户
   1. 把需要被搜索的数据加到Redis缓存

3. 组队（前端的同学想找后端的同学帮助自己完成后端，后端的同学也需要前端的同学帮助自己完成前端，需要帮助的同学就可以创建一个队伍，等待有缘人，只能由一方单向发起组队）
   1. 创建队伍
   2. 加入队伍
   3. 根据标签查询队伍
   4. 删除队伍
   5. 编辑队伍
   6. 邀请其他人
   7. 队伍人数由上限，但为了防止有人放鸽子，可以多加几个人作为替补

4. 推荐相似伙伴
   1. 使用相似度计算算法　＋　本地分布式（采用多线程）


## 技术选型

### 前端

1. Vue3框架（提高页面开发效率）
2. Vant3 UI组件库（基于Vue的移动端组件库）（它也有基于React的版本Zent）
3. Vite4（打包工具，速度快）
4. Nginx 单机部署

### 后端

1. Java8 （编程语言）
2. Spring（依赖注入框架，管理Java对象，集成一些其他内容） 
3. SpringMvc （Web框架，提供接口访问，restful接口等能力）
4. Mybatis（Java操作数据库的框架，持久层框架，对JDBC的封装） 
5. Mybatis-Plus（对mybatis的增强，不用写SQL也能实现增删改查） 
6. SpringBoot（快速启动/快速集成项目，帮助管理Spring的配置，帮助整合框架） 
7. MySQL（数据库）
8. Redis（缓存，NoSQL）
9. Swagger + Knife4j 接口文档



## 计划

1. **前端项目初始化**
2. **前端主页 + 组件概览**
3. **数据库表设计**
   1. 标签表
   2. 用户表
4. **后端项目初始化**
5. **后端开发**
   1. 使用标签搜索用户
   2. 整合Swagger + Knife4j 接口文档
   3. 存量用户信息导入及同步（爬虫）
   4. 用户修改个人信息
   5. 主页推荐（默认推荐和自己兴趣相当的用户）
   6. 优化主页性能（缓存 +定时任务+分布式锁）
   7. 标签内容整理
   8. 组队
6. **前端开发**
   1. 整合路由
   2. 封装myAxios
   3. 封装获取当前登录用户功能
   4. 搜索页面-根据标签搜索用户
   5. 用户信息页
   6. 用户信息修改页
   7. 搜索结果页
   8. 登录页
   9. 主页
7. **部分细节优化**




## 前端项目初始化

**文档介绍**

![image-20230912203234599](assets/image-20230912203234599.png)

### 脚手架初始化项目

使用[ Vue CLI ](https://cli.vuejs.org/zh/)或者[Vite 官方](https://cn.vitejs.dev/guide/)

本项目使用Vite

![image-20230912214604415](assets/image-20230912214604415.png)

```
yarn create vite
```

![image-20230912215308003](assets/image-20230912215308003.png)

```sh
#安装依赖
npm install
```

![image-20230913133810106](assets/image-20230913133810106.png)

![image-20230913134340025](assets/image-20230913134340025.png)

### 整合Vant组件库

1. 安装vant

![image-20230913141746828](assets/image-20230913141746828.png)

```
npm i vant
```

2. 安装插件

```sh
#它可以自动引入组件，并按需引入组件的样式。
npm i vite-plugin-style-import@1.4.1 -D
```

3. 让Vite认识Vant-在vite.config,ts中修改代码

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import styleImport,{ VantResolve } from 'vite-plugin-style-import';
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(),styleImport({
    resolves:[VantResolve()],
    libs: [
      {
        libraryName: 'vant',
        esModule: true,
        resolveStyle: (name) => `../es/${name}/style`
      }
    ]
  }),],
})
```

4. 引入组件

![image-20230913142523914](assets/image-20230913142523914.png)

5. 修改main.ts文件

```ts
//不用多引入样式
import { createApp } from 'vue'
import App from './App.vue'
import {Button, NavBar} from "vant";

const app=createApp(App)

app.mount('#app')
```

## 前端主页 + 组件概览

**开发页面经验**：

1. 多参考
2. 从整体到局部
3. 先明确页面的布局，再写代码

**精简页面**

![image-20230913193149609](assets/image-20230913193149609.png)

![image-20230913193359006](assets/image-20230913193359006.png)

### 页面设计

1. 导航条：展示当前页面名称
2. 主页搜索框 ==> 搜索页（标签筛选页，通过标签筛选用户）==> 搜索结果页
3. 内容：

2. Tab栏
   - 主页（推荐页 + **广告**）
     - 搜锁框
     - banner
     - 推荐信息流
   - 队伍
   - 用户页（消息-暂时考虑发邮件）

### 组件概览

1. 导航栏

   ![image-20230913195343599](assets/image-20230913195343599.png)

2. TabBar

   ![image-20230914153548155](assets/image-20230914153548155.png)

3.  

### 开发

#### 通用布局

1. 抽象一个通用的布局（Layout），使得其他页面能够复用布局中的组件/样式，利于维护

![image-20230913195129701](assets/image-20230913195129701.png)

2. 复制导航栏的代码到这个布局中

![image-20230914123843394](assets/image-20230914123843394.png)

3. 在App.vue中添加布局

![image-20230914151646028](assets/image-20230914151646028.png)

4. 在main.ts中引入NavBar,Icon

![image-20230914152329292](assets/image-20230914152329292.png)

5. 对导航栏进行修改

![image-20230914152614404](assets/image-20230914152614404.png)

![image-20230914152636314](assets/image-20230914152636314.png)

6. 引入TabBar

![image-20230914153823356](assets/image-20230914153823356.png)

![image-20230914154945288](assets/image-20230914154945288.png)

![image-20230914163717175](assets/image-20230914163717175.png)

7. 效果

![image-20230914163929186](assets/image-20230914163929186.png)

![image-20230914163956560](assets/image-20230914163956560.png)

## 数据库表设计

### 新增标签表

**标签**：

性别：男，女

方向：Java，C++，Go，前端，后端

目标：考研，春招，秋招，社招，考公，竞赛，转行，跳槽

段位：初级，中级，高级，码神

身份：小学，初中，高中，大一，大二，大三，大四，学生，待就业，已就业，研一，研二，研三

状态：乐观，聒噪，狂热，一般，单身，已婚，有对象

用户自己定义标签：开发者不可能想到所有用户需要的标签，让用户去决定要右哪些标签

**字段**：

id   		   bigint          主键 	

tagName vachar  	 非空标签名（标签具体内容，比如男）（唯一索引）

userId 	 bigint 		上传标签的用户 （用户可以自定义标签，然后上传） （普通索引，根据userId查询已上传的标签）

parentId  bigint 		父标签id (就是标签的抽象，比如性别)

isParent   tinyint   	是否为父标签（0-不是，1-是） 

createTime datetime 创建时间

updateTime datetime 更新时间

isDeleted tinyint		 逻辑删除(0-未删除，1-已删除)

通过父标签id来分组 

```mysql
DROP TABLE IF EXISTS tag;

create table user_center.tag
(
    id         bigint auto_increment comment '(主键) '
        primary key,
    tagName    varchar(256)                       null comment '标签名称',
    userId     bigint                             null comment '用户id',
    parentId   bigint                             null comment '父标签id',
    isParent   tinyint                            null comment '0-不是父标签，1-是父标签',
    createTime datetime default CURRENT_TIMESTAMP not null comment '用户创建时间',
    updateTime datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '用户更新时间',
    isDeleted  tinyint  default 0                 not null comment '标签是否已删除',
    constraint uniIdx_tagName
        unique (tagName)
)
    comment '标签表';

create index Idx_userId
    on tag (userId);
```



### 修改用户表

**根据自己的需求！！！**

1. 直接在用户表补充tags字段，['Java','男'] Json（**采用**）

   优点：查询方便，不用新建关联表，标签是用户的固有属性（除了该系统，其他系统可能也会用到），节省开发成本。可以用缓存，提高用户查询性能

   缺点：用户表多一列

2. 加一个关联表，记录用户和标签的关系（**尽量减少关联查询**）

   优点：查询灵活，可以正查反查

   缺点：查询100个用户的列表时，查完用户表得到用户id，又要根据用户id查关联表获取标签id，又要根据标签id查标签表获取标签，影响扩展和查询性能

**用户表**：

```mysql
DROP TABLE IF EXISTS user;

create table user_center.user
(
    id           bigint auto_increment comment '(主键) '
        primary key,
    userAccount  varchar(256)                       null comment '登录账号',
    username     varchar(256)                       null comment '用户昵称',
    avatarUrl    varchar(1024)                      null comment '用户头像',
    gender       tinyint                            null comment '用户性别',
    userPassword varchar(256)                       not null comment '用户密码',
    phone        varchar(128)                       null comment '用户电话',
    email        varchar(512)                       null comment '用户邮箱',
    userStatus   int      default 0                 not null comment '用户状态 0-正常 ',
    createTime   datetime default CURRENT_TIMESTAMP not null comment '用户创建时间',
    updateTime   datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '用户更新时间',
    isDeleted    tinyint  default 0                 not null comment '用户是否删除',
    userRole     int      default 0                 not null comment '用户角色 0-普通用户 1-管理员',
    authCode     varchar(512)                       null comment '付费系统编号，用于校验用户'
    
)
    comment '用户表';
#补充tags字段    
alter table user add COLUMN tags varchar(1024) null comment '标签列表Json'
#补充profile字段  
alter table user add COLUMN profile varchar(512) null comment '个人简介'
```

## 后端项目初始化

本来想把标签的增删改查等业务放到新项目里，实际分析后，因为标签和用户关联比较强，所以还是把标签的增删改查放到用户中心项目里

用户中心提供用户的查询，操作，注册，登录，鉴权

## 后端开发

### 使用标签搜索用户

建议通过实际测试来分析哪种查询比较快，数据量大的时候验证效果更明显！

- 如果参数可以分析，根据用户的参数去选择查询方式，比如标签数。标签数少，用SQL查询；标签数多，用内存查询

- 如果参数不可分析，并且数据库连接足够、内存空间足够，可以并发同时查询，谁先返回用谁。
- 还可以 SQL 查询与内存查询相结合，比如先用 SQL 过滤掉部分 tag

**SQL查询**

1. 允许用户传入多个标签，多个标签在用户的tags字段都存在，用户才能被搜出来 

   ```
   tags like '%Java%' and  tags like '%C++%' and tags like '%Go%'
   ```

2. 允许用户传入多个标签，任何标签在用户的tags字段存在，用户就能被搜出来

   ```
   tags like '%Java%' or  tags like '%C++%' or tags like '%Go%'
   ```

```java
/**
 * 使用标签搜索用户（SQL查询版）
 *
 * @param tagList 用户传入的标签
 * @return
 */
@Deprecated
public List<UserDTO> queryUsersByTagsBySQL(List<String> tagList) {
    //判空
    if (CollectionUtil.isEmpty(tagList)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "标签不能为空");
    }
    //SQL模糊查询
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    for (String tag : tagList) {
        queryWrapper.like(StringUtils.isNotBlank(tag), User::getTags, tag);
    }
    List<User> users = userMapper.selectList(queryWrapper);
    //脱敏
    return users.stream().map(user -> BeanUtil.copyProperties(user, UserDTO.class)).collect(Collectors.toList());
}
```

**内存查询**

1. 在用户中心的UserServiceImpl中提供queryUsersByTagsByMemory接口

```java
/**
 * 使用标签搜索用户(内存过滤）
 *
 * @param tagList 用户传入的标签
 * @return
 */
@Override
public List<UserDTO> queryUsersByTagsByMemory(List<String> tagList) {
    //判空
    if (CollectionUtil.isEmpty(tagList)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "标签不能为空");
    }
    //内存查询
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    //查询所有用户
    List<User> users = userMapper.selectList(queryWrapper);
    Gson gson = new Gson();
    return users.stream().filter(user -> {
        Set<String> tempTagSet = gson.fromJson(user.getTags(), new TypeToken<Set<String>>() {
        }.getType());
        //有些用户的tags字段可能为空，要先判空，否则报控空指针异常
        tempTagSet = Optional.ofNullable(tempTagSet).orElse(new HashSet<>());
        //过滤
        for (String tag : tagList) {
            if (!tempTagSet.contains(tag))
                return false;
        }
        return true;
    }).map(user -> BeanUtil.copyProperties(user, UserDTO.class)).collect(Collectors.toList());
}
```

2. 因为user表加了字段，相应的做些修改

![image-20230915152948531](assets/image-20230915152948531.png)

![image-20230915153041395](assets/image-20230915153041395.png)

![image-20230915153137469](assets/image-20230915153137469.png)

3. 在UserController中添加searchByTags方法，接受前端请求

```
@GetMapping("/searchByTags")
public Result searchByTags(@RequestParam(required = false) List<String> tags){
    if (CollectionUtil.isEmpty(tags)){
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"标签为空");
    }
    return Result.success(userService.queryUsersByTagsByMemory(tags));
}
```

4. 解决跨域

![image-20230919203356552](assets/image-20230919203356552.png)

### 整合Swagger + Knife4j 接口文档

1. 什么是接口文档？

   用于描述接口信息，包括：

   - 请求参数
   - 响应参数
   - 接口地址
   - 接口名称
   - 请求类型
   - 请求格式
   - 备注

2. 由谁提供，由谁使用？

   一般是后端或者负责人来提供，后端和前端都可以使用

3. 为什么需要接口文档？

   - 便于参考和查阅，便于**沉淀和维护**，拒绝口口相传
   - 便于前后端开发对接，是前后端联调的**介质**
   - 支持在线调试，在线测试， 可以作为工具，提高我们的开发效率

4. 怎么做接口文档？

   - 手写（MarkDown笔记等)
   - 自动化接口文档生成：（自动根据项目代码生成完整的文档或在线调试的网页）
     - **Swagger（采用）**，Postman（侧重接口管理）（国外）
     - apifox，apipost，eolink（国产）

5. 接口文档右哪些技巧？

6. Swagger原理

   1. 引入依赖（Swagger 或 Knife4j：[1.6 快速开始 | knife4j (xiaominfo.com)](https://doc.xiaominfo.com/v2/documentation/get_start.html)）

   2. 自定义Swagger配置类

   3. 定义需要生成接口文档的代码的位置（controller）

   4. 启动项目即可

   5. 可以通过在 controller 方法上添加 @Api、@ApiImplicitParam(name = "name",value = "姓名",required = true)    @ApiOperation(value = "向客人问好") 等注解来自定义生成的接口描述信息

   6. 如果 springboot version >= 2.6，需要添加如下配置：

      ```yaml
      spring:
        mvc:
        	pathmatch:
            matching-strategy: ANT_PATH_MATCHER
      ```

   7. 访问http://localhost:8080/api/doc.html

7. 注意：线上环境不要把接口暴露出去！！！

8. 隐藏 可以通过在 SwaggerConfig 配置类开头加上 `@Profile({"dev", "test"})` 限定配置仅在部分环境开启，其实是限制bean在特定环境下被注册到容器中

**整合**

1. 引入依赖

   ```java
   <!--swagger-->
   <dependency>
       <groupId>com.github.xiaoymin</groupId>
       <artifactId>knife4j-spring-boot-starter</artifactId>
       <!--在引用时请在maven中央仓库搜索2.X最新版本号-->
       <version>2.0.9</version>
   </dependency>
   ```

2. 编写配置类

   ```java
   package com.luoying.config;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import springfox.documentation.builders.ApiInfoBuilder;
   import springfox.documentation.builders.PathSelectors;
   import springfox.documentation.builders.RequestHandlerSelectors;
   import springfox.documentation.service.ApiInfo;
   import springfox.documentation.service.Contact;
   import springfox.documentation.spi.DocumentationType;
   import springfox.documentation.spring.web.plugins.Docket;
   import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;
   
   /**
    * 自定义Swagger接口文档使用的配置
    *
    * @author 落樱的悔恨
    */
   @Configuration
   @EnableSwagger2WebMvc
   public class Swagger2Configuration {
       @Bean(value = "defaultApi2")
       public Docket createRestApi(){
           return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo())
                   .select()
                   //这里标注controller包的位置
                   .apis(RequestHandlerSelectors.basePackage("com.luoying.controller"))
                   .paths(PathSelectors.any())
                   .build();
       }
    
       //api基本信息的配置，信息会在api文档上显示
       private ApiInfo apiInfo(){
           return new ApiInfoBuilder()
                   .title("LUOYING用户中心")
                   .description("LUOYING用户中心相关接口的文档")
                   .termsOfServiceUrl("http://github.com/1ranxu")
                   .contact(new Contact("ranxu","http://github.com/1ranxu","1574925401@qq.com"))
                   .version("1.0")
                   .build();
       }
   }
   ```

3. 修改配置文件

   ```yml
   spring:  
     mvc:
       pathmatch:
         matching-strategy: ant_path_matcher
   #springboot版本>=2.6，
   ```

4. 效果

   ![image-20230916195725901](assets/image-20230916195725901.png)

### 存量用户信息导入及同步（爬虫）

1. 把所有星球用户的信息导入用户表
2. 从每个用户的自我介绍中解析标签，在用户表中给每个用户打标签，并把解析的标签导入标签表

**看上了网页信息，怎么抓取**

FeHelper

1. 分析原网站是用哪个接口获取的[api.zsxq.com/v2/hashtags/48844541281228/topics?count=20](https://api.zsxq.com/v2/hashtags/48844541281228/topics?count=20)

   按 F 12 打开控制台，查看网络请求，复制 curl 代码便于查看和执行：

   ```bash
   curl "https://api.zsxq.com/v2/hashtags/48844541281228/topics?count=20" ^
     -H "authority: api.zsxq.com" ^
     -H "accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7" ^
     -H "accept-language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6" ^
     -H "cache-control: max-age=0" ^
     -H "cookie: zsxq_access_token=AA992A14-3A7E-86B6-02ED-06E0FC6B65FE_2EF81009CEAA9466; sensorsdata2015jssdkcross=^%^7B^%^22distinct_id^%^22^%^3A^%^2218a8ea5df263fd-099a715b3ba0c38-7f5d547e-1327104-18a8ea5df27174d^%^22^%^2C^%^22first_id^%^22^%^3A^%^22^%^22^%^2C^%^22props^%^22^%^3A^%^7B^%^22^%^24latest_traffic_source_type^%^22^%^3A^%^22^%^E5^%^BC^%^95^%^E8^%^8D^%^90^%^E6^%^B5^%^81^%^E9^%^87^%^8F^%^22^%^2C^%^22^%^24latest_search_keyword^%^22^%^3A^%^22^%^E6^%^9C^%^AA^%^E5^%^8F^%^96^%^E5^%^88^%^B0^%^E5^%^80^%^BC^%^22^%^2C^%^22^%^24latest_referrer^%^22^%^3A^%^22https^%^3A^%^2F^%^2Fwww.yuque.com^%^2F^%^22^%^7D^%^2C^%^22identities^%^22^%^3A^%^22eyIkaWRlbnRpdHlfY29va2llX2lkIjoiMThhOGVhNWRmMjYzZmQtMDk5YTcxNWIzYmEwYzM4LTdmNWQ1NDdlLTEzMjcxMDQtMThhOGVhNWRmMjcxNzRkIn0^%^3D^%^22^%^2C^%^22history_login_id^%^22^%^3A^%^7B^%^22name^%^22^%^3A^%^22^%^22^%^2C^%^22value^%^22^%^3A^%^22^%^22^%^7D^%^2C^%^22^%^24device_id^%^22^%^3A^%^2218a8ea5df263fd-099a715b3ba0c38-7f5d547e-1327104-18a8ea5df27174d^%^22^%^7D; abtest_env=product; zsxqsessionid=b0e9ee1668ca2ed22f495c4c2190422f" ^
     -H "sec-ch-ua: ^\^"Microsoft Edge^\^";v=^\^"117^\^", ^\^"Not;A=Brand^\^";v=^\^"8^\^", ^\^"Chromium^\^";v=^\^"117^\^"" ^
     -H "sec-ch-ua-mobile: ?0" ^
     -H "sec-ch-ua-platform: ^\^"Windows^\^"" ^
     -H "sec-fetch-dest: document" ^
     -H "sec-fetch-mode: navigate" ^
     -H "sec-fetch-site: none" ^
     -H "sec-fetch-user: ?1" ^
     -H "upgrade-insecure-requests: 1" ^
     -H "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36 Edg/117.0.2045.31" ^
     --compressed
   ```


2. **用程序去调用接口** （java okhttp httpclient / python 都可以）
3. 处理（清洗）一下数据，之后就可以写到数据库里

**流程**

1. 从 excel 中导入全量用户数据，**判重** 。 [EasyExcel官方文档 - 基于Java的Excel处理工具 | Easy Excel (alibaba.com)](https://easyexcel.opensource.alibaba.com/index.html)
2. 抓取写了自我介绍的同学信息，提取出用户昵称、用户唯一 id、自我介绍信息
3. 从自我介绍中提取信息，然后写入到数据库中

**使用EasyExcel从Excel表中读取数据**

1. 引入依赖

   ```java
   <!-- https://mvnrepository.com/artifact/com.alibaba/easyexcel -->
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>easyexcel</artifactId>
       <version>3.1.1</version>
   </dependency>
   ```

2. 建立对象，和Excel表字段形成映射关系

   ```java
   @Data
   public class DemoData {
       /**
        * 成员编号
        */
       @ExcelProperty("成员编号")
       private String authCode;
   
       /**
        * 用户昵称
        */
       @ExcelProperty("成员昵称")
       private String username;
   
   }
   ```

3. 读取模式（任选一种)

   1. 监听器

   ```java
   // 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
   @Slf4j
   public class DemoDataListener implements ReadListener<DemoData> {
   
       /**
        * 这个每一条数据解析都会来调用
        *
        * @param data    one row value. Is is same as {@link AnalysisContext#readRowHolder()}
        * @param context
        */
       @Override
       public void invoke(DemoData data, AnalysisContext context) {
           System.out.println(data);
       }
   
       /**
        * 所有数据解析完成了 都会来调用
        *
        * @param context
        */
       @Override
       public void doAfterAllAnalysed(AnalysisContext context) {
           System.out.println("完成");
       }
   
   }
   ```

   2. 同步读

   ```java
   /**
    * 同步的读取，不推荐使用，如果数据量大会把数据放到内存里面
    */
   public static void synchronousRead() {
       // 写法2
       String fileName = "prodExcel.xlsx";
       // 这里 需要指定读用哪个class去读，然后读取第一个sheet 同步读取会自动finish
       List<DemoData> toatlDataList = EasyExcel.read(fileName).head(DemoData.class).sheet().doReadSync();
       for (DemoData demoData : toatlDataList) {
           System.out.println(demoData);
       }
   }
   ```

两种读对象的方式：

1. 确定表头：建立对象，和表头形成映射关系
2. 不确定表头：每一行数据映射为 Map<String, Object>

两种读取模式：

1. 监听器：先创建监听器、在读取文件时绑定监听器。单独抽离处理逻辑，代码清晰易于维护；一条一条处理，适用于数据量大的场景。
2. 同步读：无需创建监听器，一次性获取完整数据。方便简单，但是数据量大时会有等待时常，也可能内存溢出。

### 用户修改个人信息

1. 重写userUpdate，添加getLoginUser，迁移isAdmin

![image-20230921112906473](assets/image-20230921112906473.png)

![image-20230921113337257](assets/image-20230921113337257.png)

![image-20230921113435280](assets/image-20230921113435280.png)

![image-20230921113532147](assets/image-20230921113532147.png)

2. 修改UserController

![image-20230921113922404](assets/image-20230921113922404.png)

![image-20230921114205296](assets/image-20230921114205296.png)

![image-20230921114720510](assets/image-20230921114720510.png)

### 主页推荐

1. 目前是查询所有用户，然后返回，之后再优化

```java
@GetMapping("/recommend")
public Result usersRecommend(long currentPage, long pageSize, HttpServletRequest request) {
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper();
    Page<User> page = userService.page(new Page<User>(currentPage, pageSize), wrapper);
    List<UserDTO> userDTOList = page.getRecords().stream().map(user1 -> {
        return BeanUtil.copyProperties(user1, UserDTO.class);
    }).collect(Collectors.toList());
    return Result.success(userDTOList);
}
```

####  导入数据

1. 用可视化界面：适合一次性导入、数据量可控

2. 写程序：for 循环，建议分批，（可以用接口来控制）**要保证可控、幂等，注意线上环境和测试环境是有区别的**
   1. 批量查询解决每次插入会建立和释放数据库链接的问题


```java
//在测试里执行完后一定要删除，不然会影响线上环境
@Test
public void doInsertUsers() {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    final int Insert_Num = 1000;
    List<User> userList = new ArrayList<>();
    for (int i = 0; i < Insert_Num; i++) {
        User user = new User();
        user.setUserAccount("fakeUser" + i);
        user.setUsername("fakeUser" + i);
        user.setAvatarUrl("https://thirdwx.qlogo.cn/mmopen/vi_32/RNJfsfhsEic2dzoJasQgMoHjw0h5580v6aQ5OOCHM0Fk8B3Fw98CWmZlbqcx8LLDEKunkZnwU5aEliaKhic8MFOYQ/132");
        user.setGender(0);
        user.setUserPassword("0123456789");
        user.setPhone("123");
        user.setEmail("123@qq.com");
        user.setUserStatus(0);
        user.setUserRole(0);
        user.setAuthCode("66666");
        user.setTags("[]");
        user.setProfile("精神小伙");
        userList.add(user);
    }
    userService.saveBatch(userList,100);
    stopWatch.stop();
    System.out.println(stopWatch.getTotalTimeMillis());
}
```

2. 并发插入数据

```java
private ExecutorService executorService= new ThreadPoolExecutor(
        60,1000,100000, TimeUnit.SECONDS,new ArrayBlockingQueue<>(10000));
@Test
public void doConcurrencyInsertUsers() {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    int batchSize = 25000;
    List<CompletableFuture<Void>> futureList=new ArrayList<>();
    //分十组，每组100000条
    int j=0;
    for (int i = 0; i < 10; i++) {
        List<User> userList = new ArrayList<>();
        while(true) {
            j++;
            if (j%100000==0){
                break;
            }
            User user = new User();
            user.setUserAccount("fakeUser" + j);
            user.setUsername("fakeUser" + j);
            user.setAvatarUrl("https://thirdwx.qlogo.cn/mmopen/vi_32/RNJfsfhsEic2dzoJasQgMoHjw0h5580v6aQ5OOCHM0Fk8B3Fw98CWmZlbqcx8LLDEKunkZnwU5aEliaKhic8MFOYQ/132");
            user.setGender(0);
            user.setUserPassword("0123456789");
            user.setPhone("123");
            user.setEmail("123@qq.com");
            user.setUserStatus(0);
            user.setUserRole(0);
            user.setAuthCode("66666");
            user.setTags("[]");
            user.setProfile("精神小伙");
            userList.add(user);
        }
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            System.out.println("ThreadName===>"+Thread.currentThread().getName());
            userService.saveBatch(userList, batchSize);
        },executorService);
        futureList.add(future);
    }
    CompletableFuture.allOf(futureList.toArray(new CompletableFuture[]{})).join();
    stopWatch.stop();
    System.out.println(stopWatch.getTotalTimeMillis());
}
```


      2. 执行 SQL 语句：适用于小数据量

并发要注意执行的先后顺序无所谓，不要用到非并发类的集合

数据库慢？预先把数据查出来，放到一个更快读取的地方，不用再查数据库了。（缓存）

预加载缓存，定时更新缓存。（定时任务）

多个机器都要执行任务么？（分布式锁：控制同一时间只有一台机器去执行定时任务，其他机器不用重复执行了）

### 组队

#### **理想的应用场景**

1. 我要跟别人一起参加竞赛或者做项目，可以发起队伍或者加入别人的队伍

#### **需求分析**

1. 用户可以创建一个队伍，设置队伍的人数，队伍名称（标题）、描述、超时时间 P0

   > 队长、剩余的人数
   >
   > 聊天？
   >
   > 公开 或 private 或加密
   >
   > **用户创建队伍最多 5 个**

2. 展示队伍列表，根据名称搜索队伍  P0，信息流中不展示已过期的队伍 P0

3. 修改队伍信息 P0

4. 用户可以加入其他人的队伍（未满、未过期），允许加入多个队伍，但是要有个上限  P0

   > 是否需要队长同意？筛选审批？

5. 用户可以退出队伍（如果队长退出，权限转移给第二早加入的用户 —— 先来后到） P0

6. 队长可以解散队伍 P0

7. 分享队伍 ==> 邀请其他用户加入队伍 P1

   1. 业务流程：

      1. 生成分享链接（分享二维码）
      2. 用户访问链接，可以点击加入

8. 队伍人满后发送消息通知 P1

#### **数据库表设计**

队伍表：

- id bigint 主键（最简单、连续，放 url 上比较简短，但缺点是爬虫）

- name 队伍名称
- description 描述 
- maxNum 最大人数
- expireTime 过期时间
- userId 创建人 id
- status 0 - 公开，1 - 私有，2 - 加密
- password 密码
- createTime 创建时间
- updateTime 更新时间
- isDelete 是否删除

```sql
DROP TABLE IF EXISTS team;
create table team
(
    id          bigint auto_increment comment 'id' primary key,
    name        varchar(256)                       not null comment '队伍名称',
    description varchar(1024)                      null comment '描述',
    maxNum      int      default 1                 not null comment '最大人数',
    expireTime  datetime                           null comment '过期时间',
    userId      bigint comment '用户id (队长 id) ',
    status      int      default 0                 not null comment '0 - 公开，1 - 私有，2 - 加密',
    password    varchar(512)                       null comment '密码',
    createTime  datetime default CURRENT_TIMESTAMP null comment '创建时间',
    updateTime  datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP comment '更新时间',
    isDeleted    tinyint  default 0                 not null comment '是否删除'
)
    comment '队伍';
```

用户 __ 队伍表 user_team

字段：

- id 主键
- userId 用户 id
- teamId 队伍 id
- joinTime 加入时间
- createTime 创建时间
- updateTime 更新时间
- isDelete 是否删除

```sql
DROP TABLE IF EXISTS user_team;
create table user_team
(
    id         bigint auto_increment comment 'id' primary key,
    userId     bigint comment '用户id',
    teamId     bigint comment '队伍id',
    joinTime   datetime                           null comment '加入时间',
    createTime datetime default CURRENT_TIMESTAMP null comment '创建时间',
    updateTime datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP,
    isDeleted   tinyint  default 0                 not null comment '是否删除'
)
    comment '用户队伍关系';
```

两个关系：

1. 用户加了哪些队伍？
2. 队伍有哪些用户？

方式：

1. 建立用户 - 队伍关系表 teamId userId（便于修改，查询性能高一点，可以选择这个，不用全表遍历）
2. 用户表补充已加入的队伍字段，队伍表补充已加入的用户字段（便于查询，不用写多对多的代码，可以直接根据队伍查用户、根据用户查队伍）

#### MybatisX生成代码

#### 系统（接口）设计

##### 1、创建队伍

用户可以 **创建** 一个队伍，设置队伍的人数、队伍名称（标题）、描述、超时时间 P0

> 队长、剩余的人数
>
> 聊天？
>
> 公开 或 private 或加密
>
> 信息流中不展示已过期的队伍

1. 请求参数是否为空？
2. 是否登录，未登录不允许创建
3. 校验信息
   1. 队伍人数 > 1 且 <= 10
   2. 队伍标题 <= 20
   3. 描述 <= 512
   4. status 是否公开（int）不传默认为 0（公开）
   5. 如果 status 是加密状态，一定要有密码，且密码 <= 32
   6. 超时时间 > 当前时间
   7. 校验用户最多创建 5 个队伍
4. 插入队伍信息到队伍表
5. 插入用户  => 队伍关系到关系表

```java
@Data
public class TeamAddRequest {

    private String name;

    private String description;

    private Integer maxNum;

    private Date expireTime;

    private Long userId;

    private Integer status;

    private String password;
}
```

```java
@PostMapping("/add")
public Result addTeam(@RequestBody TeamAddRequest teamAddRequest, HttpServletRequest request) {
    if (teamAddRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"teamAddRequest空值");
    }
    Team team = BeanUtil.copyProperties(teamAddRequest, Team.class);
    UserVO loginUser = userService.getLoginUser(request);
    long teamId = teamService.addTeam(team, loginUser);
    return Result.success(teamId);
}
```

```java
/**
 * 创建队伍
 * @param team
 * @return
 */
long addTeam(Team team, UserVO loginUser);
```

```java
@Override
@Transactional(rollbackFor = Exception.class)
public long addTeam(Team team, UserVO loginUser) {
    //1. 请求参数是否为空？
    if (team == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "team空值");
    }
    // 2. 是否登录，未登录不允许创建
    if (loginUser == null) {
        throw new BusinessException(ErrorCode.NO_LOGIN, "未登录");
    }
    // 3. 校验信息
    //    1. 队伍人数 > 1 且 <= 10
    Integer maxNum = Optional.ofNullable(team.getMaxNum()).orElse(0);
    if (maxNum <= 1 || maxNum > 10) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍人数不满足要求");
    }
    //    2. 队伍标题 <= 20
    String name = team.getName();
    if (StringUtils.isBlank(name) || name.length() > 20) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍标题不满足要求");
    }
    //    3. 描述 <= 512
    String description = team.getDescription();
    if (description.length() > 512) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍描述过长");
    }
    //    4. status 是否公开（int）不传默认为 0（公开）
    Integer status = Optional.ofNullable(team.getStatus()).orElse(0);
    TeamStatusEnum statusEnum = TeamStatusEnum.getEnumByValue(status);
    if (statusEnum == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍状态不满足要求");
    }
    //    5. 如果 status 是加密状态，一定要有密码，且密码 <= 32
    String password = team.getPassword();
    if (TeamStatusEnum.SECRET.equals(statusEnum) && (StringUtils.isBlank(password) || password.length() > 32)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍密码设置不满足要求");
    }
    //    6. 超时时间 > 当前时间
    Date expireTime = team.getExpireTime();
    if (new Date().after(expireTime)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍已过期");
    }
    //    7. 校验用户最多创建 5 个队伍
    // todo 有bug，用户可能连续点100次，创建100个队伍 加分布式锁解决
    LambdaQueryWrapper<Team> queryWrapper = new LambdaQueryWrapper<>();
    Long userId = loginUser.getId();
    queryWrapper.eq(Team::getUserId, userId);
    long hasTeamNum = this.count(queryWrapper);
    if (hasTeamNum >= 5) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "单个用户最多创建5个用户");
    }
    // 4. 插入队伍信息到队伍表
    team.setUserId(userId);
    boolean result = this.save(team);
    Long teamId = team.getId();
    if (!result || teamId == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "创建队伍失败");
    }
    // 5. 插入用户 -队伍关系到关系表
    UserTeam userTeam = new UserTeam();
    userTeam.setUserId(userId);
    userTeam.setTeamId(teamId);
    userTeam.setJoinTime(new Date());
    result = userTeamService.save(userTeam);
    if (!result) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "创建用户-队伍失败");
    }
    return teamId;
}
```

##### 2、查询队伍列表

分页展示队伍列表，根据名称、最大人数等搜索队伍  P0，信息流中不展示已过期的队伍

1. 从请求参数中取出队伍名称等查询条件，如果存在则作为查询条件
2. 不展示已过期的队伍（根据过期时间筛选）
3. 可以通过某个**关键词**同时对名称和描述查询
4. **只有管理员才能查看加密还有非公开的房间**
5. 关联查询已加入队伍的用户信息
6. **关联查询已加入队伍的用户信息（可能会很耗费性能，建议大家用自己写 SQL 的方式实现）**

```java
@Data
@EqualsAndHashCode(callSuper = true)
public class TeamQueryRequest extends PageRequest implements Serializable {

    private Long id;

    private String searchText;

    private String name;

    private String description;

    private Integer maxNum;

    private Date expireTime;

    private Long userId;

    private Integer status;

}
```

```java
@GetMapping("/list")
public Result getTeamList(TeamQueryRequest teamQueryRequest, HttpServletRequest request) {
    if (teamQueryRequest == null) {
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "队伍不存在");
    }
    boolean isAdmin = userService.isAdmin(request);
    List<TeamUserVO> teamList = teamService.listTeams(teamQueryRequest, request, isAdmin);
    return Result.success(teamList);
}
```

```java
/**
 * 搜索队伍
 *
 * @param teamQueryRequest
 * @param request
 * @param isAdmin
 * @return
 */
List<TeamUserVO> listTeams(TeamQueryRequest teamQueryRequest, HttpServletRequest request, boolean isAdmin);
```

```java
@Override
public List<TeamUserVO> listTeams(TeamQueryRequest teamQueryRequest, HttpServletRequest request, boolean isAdmin) {
    LambdaQueryWrapper<Team> teamWrapper = new LambdaQueryWrapper<>();
    //组合队伍查询条件
    if (teamQueryRequest != null) {
        //根据队伍id查询
        Long id = teamQueryRequest.getId();
        teamWrapper.eq(id != null && id > 0, Team::getId, id);
        //根据id列表查询
        List<Long> ids = teamQueryRequest.getIds();
        teamWrapper.in(CollectionUtil.isNotEmpty(ids),Team::getId,ids);
        //根据搜索内容查询
        String searchText = teamQueryRequest.getSearchText();
        teamWrapper
                .like(StringUtils.isNotBlank(searchText), Team::getName, searchText)
                .or()
                .like(StringUtils.isNotBlank(searchText), Team::getDescription, searchText);
        //根据队伍名查询
        String teamName = teamQueryRequest.getName();
        teamWrapper.like(StringUtils.isNotBlank(teamName), Team::getName, teamName);
        //根据队伍描述查询
        String description = teamQueryRequest.getDescription();
        teamWrapper.like(StringUtils.isNotBlank(description), Team::getDescription, description);
        //根据队伍最大人数查询
        Integer maxNum = teamQueryRequest.getMaxNum();
        teamWrapper.eq(maxNum != null && maxNum > 0 && maxNum <= 20, Team::getMaxNum, maxNum);
        //根据创建人id查询
        Long userId = teamQueryRequest.getUserId();
        teamWrapper.eq(userId != null && userId > 0, Team::getUserId, userId);
        //不展示已过期队伍
        teamWrapper.and(qw -> qw.isNull(Team::getExpireTime).or().gt(Team::getExpireTime, new Date()));
        //根据队伍状态查询
        Integer status = teamQueryRequest.getStatus();
        TeamStatusEnum statusEnum = TeamStatusEnum.getEnumByValue(status);
        if (statusEnum == null) {
            //如果队伍状态没传，公开加密都可以查
            teamWrapper.in(Team::getStatus, TeamStatusEnum.PUBLIC.getValue(),TeamStatusEnum.SECRET.getValue());
        }else {
            //如果队伍状态传了,就根据队伍状态查
            teamWrapper.eq(Team::getStatus, statusEnum.getValue());
        }
        if (!isAdmin && statusEnum.equals(TeamStatusEnum.PRIVATE)) {
            throw new BusinessException(ErrorCode.NO_AUTH, "无权限");
        }
    }
    //查询队伍
    List<Team> teamList = this.list(teamWrapper);
    if (CollectionUtil.isEmpty(teamList)) {
        return new ArrayList<>();
    }
    //关联查询用户信息
    List<TeamUserVO> teamUserVOList = new ArrayList<>();
    // 关联查询创建人的用户信息
    for (Team team : teamList) {
        TeamUserVO teamUserVO = BeanUtil.copyProperties(team, TeamUserVO.class);
        User user = userService.getById(team.getUserId());
        UserVO userVO = BeanUtil.copyProperties(user, UserVO.class);
        teamUserVO.setCreateUser(userVO);
        teamUserVOList.add(teamUserVO);
    }
    //获取所有队伍id
    final List<Long> teamIdList = teamUserVOList.stream().map(TeamUserVO::getId).collect(Collectors.toList());
    //判断当前用户是否已加入队伍
    LambdaQueryWrapper<UserTeam> userTeamQueryWrapper = new LambdaQueryWrapper<>();
    try {
        UserVO loginUser = userService.getLoginUser(request);
        userTeamQueryWrapper.eq(UserTeam::getUserId, loginUser.getId());
        userTeamQueryWrapper.in(UserTeam::getTeamId, teamIdList);
        List<UserTeam> userTeamList = userTeamService.list(userTeamQueryWrapper);
        // 已加入的队伍 id 集合
        Set<Long> hasJoinTeamIdSet = userTeamList.stream().map(UserTeam::getTeamId).collect(Collectors.toSet());
        teamUserVOList.forEach(team -> {
            boolean hasJoin = hasJoinTeamIdSet.contains(team.getId());
            team.setHasJoin(hasJoin);
        });
    } catch (Exception e) {
    }
    //查询已加入队伍的所有用户
    userTeamQueryWrapper = new LambdaQueryWrapper<>();
    userTeamQueryWrapper.in(UserTeam::getTeamId, teamIdList);
    List<UserTeam> userTeamList = userTeamService.list(userTeamQueryWrapper);
    // 队伍 id => 加入这个队伍的用户列表
    Map<Long, List<UserTeam>> teamIdUserTeamMap = userTeamList.stream().collect(Collectors.groupingBy(UserTeam::getTeamId));
    //给每个队伍设置队伍人数
    //getOrDefault是map的一个方法，如果这个键teamId的vlaue为null没有队员，就给个空集合
    teamUserVOList.forEach(team -> team.setHasJoinNum(teamIdUserTeamMap.getOrDefault(team.getId(), new ArrayList<>()).size()));
    return teamUserVOList;
}
```

**实现方式**

1）自己写 SQL

```sql
// 1. 自己写 SQL
// 查询队伍和创建人的信息
select * from team t left join user u on t.userId = u.id
// 查询队伍和已加入队伍成员的信息
select * from team t 
    left join user_team ut on t.id = ut.teamId 
    left join user u on ut.userId = u.id;
```



##### 3. 修改队伍信息

1. 判断请求参数是否为空
2. 查询队伍是否存在
3. 只有管理员或者队伍的创建者可以修改
4. 如果用户传入的新值和老值一致，就不用 update 了（可自行实现，降低数据库使用次数）
5. **如果队伍状态改为加密，必须要有密码**
6. 更新成功

```java
@Data
public class TeamUpdateRequest {

    private Long id;

    private String name;

    private String description;

    private Integer maxNum;

    private Date expireTime;

    private Integer status;

    private String password;
}
```

```java
@PostMapping("/update")
public Result updateTeam(@RequestBody TeamUpdateRequest teamUpdateRequest,HttpServletRequest request) {
    if (teamUpdateRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"teamUpdateRequest空值");
    }
    UserVO loginUser = userService.getLoginUser(request);
    boolean result = teamService.updateTeam(teamUpdateRequest,loginUser);
    if (!result) {
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "更新队伍失败");
    }
    return Result.success(true);
}
```

```java
/**
 * 修改队伍
 * @param teamUpdateRequest
 * @param loginUser
 * @return
 */
boolean updateTeam(TeamUpdateRequest teamUpdateRequest, UserVO loginUser);
```

```java
@Override
public boolean updateTeam(TeamUpdateRequest teamUpdateRequest, UserVO loginUser) {
    if (teamUpdateRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "teamUpdateRequest空值");
    }
    //查询队伍是否存在
    Long id = teamUpdateRequest.getId();
    if (id == null || id <= 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍不存在");
    }
    //查询数据库
    Team dbTeam = this.getById(id);
    if (dbTeam == null) {
        throw new BusinessException(ErrorCode.NULL_ERROR, "队伍不存在");
    }
    //只有管理员或者创建人才能修改
    if (loginUser.getId() != dbTeam.getUserId() && !userService.isAdmin(loginUser)) {
        throw new BusinessException(ErrorCode.NO_AUTH, "无权限");
    }
    //用户如果修改的是公开队伍，密码不管有没有，都设置为空
    TeamStatusEnum statusEnum = TeamStatusEnum.getEnumByValue(teamUpdateRequest.getStatus());
    if (TeamStatusEnum.PUBLIC.equals(statusEnum)) {
        teamUpdateRequest.setPassword("");
    }
    //如果修改的是加密队伍，密码不能为空
    if (TeamStatusEnum.SECRET.equals(statusEnum)) {
        if (StringUtils.isBlank(teamUpdateRequest.getPassword())) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR, "密码不能为空");
        }
    }
    return this.updateById(BeanUtil.copyProperties(teamUpdateRequest, Team.class));
}
```

##### 4. 用户可以加入队伍

其他人、未满、未过期，允许加入多个队伍，但是要有个上限  P0

1. 用户最多加入 5 个队伍
2. 队伍必须存在，只能加入未满、未过期的队伍
3. 不能重复加入已加入的队伍（幂等性）
4. 禁止加入私有的队伍
5. 如果加入的队伍是加密的，必须密码匹配才可以
6. 新增队伍 - 用户关联信息

> 注意：并发请求时可能出现问题

**注意，一定要加上事务注解！！！！**

```java
@Data
public class TeamJoinRequest {
    private Long teamId;

    private String password;

}
```

```java
@PostMapping("/join")
public Result joinTeam(@RequestBody TeamJoinRequest teamJoinRequest,HttpServletRequest request){
    if (teamJoinRequest==null){
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"teamJoinRequest空值");
    }
    UserVO loginUser = userService.getLoginUser(request);
    boolean result=teamService.joinTeam(teamJoinRequest,loginUser);
    if (!result){
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"加入队伍失败");
    }
    
    return Result.success(true);
}
```

```java
/**
 * 加入队伍
 * @param teamJoinRequest
 * @param loginUser
 * @return
 */
boolean joinTeam(TeamJoinRequest teamJoinRequest, UserVO loginUser);
```

```java
@Override
public boolean joinTeam(TeamJoinRequest teamJoinRequest, UserVO loginUser) {
    if (teamJoinRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "teamJoinRequest空值");
    }
    //加入的队伍必须要存在
    Long teamId = teamJoinRequest.getTeamId();
    LambdaQueryWrapper<Team> TeamWrapper = new LambdaQueryWrapper<>();
    TeamWrapper.eq(teamId != null && teamId > 0, Team::getId, teamId);
    Team team = this.getOne(TeamWrapper);
    if (team == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍不存在");
    }
    //只能加入未过期的队伍
    if (team.getExpireTime() != null && new Date().after(team.getExpireTime())) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍已过期");
    }
    //不能加入私有队伍
    TeamStatusEnum statusEnum = TeamStatusEnum.getEnumByValue(team.getStatus());
    if (TeamStatusEnum.PRIVATE.equals(statusEnum)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "不能加入私有队伍");
    }
    //加入加密队伍，密码必须匹配
    if (TeamStatusEnum.SECRET.equals(statusEnum)) {
        String password = teamJoinRequest.getPassword();
        if (StringUtils.isBlank(password) || !StringUtils.equals(team.getPassword(), password)) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍密码错误");
        }
    }
    //该用户已加入队伍的数量
    Long userId = loginUser.getId();
    LambdaQueryWrapper<UserTeam> userTeamWrapper = new LambdaQueryWrapper<>();
    userTeamWrapper.eq(userId != null && userId > 0, UserTeam::getUserId, userId);
    long count = userTeamService.count(userTeamWrapper);
    if (count >= 5) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "最多同时加入5个队伍");
    }
    //只能加入未满的队伍
    userTeamWrapper = new LambdaQueryWrapper<>();
    userTeamWrapper.eq(teamId != null && teamId > 0, UserTeam::getTeamId, teamId);
    count = userTeamService.count(userTeamWrapper);
    if (count == team.getMaxNum()) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍人数已满");
    }
    //不能重复加入已加入的队伍
    userTeamWrapper = new LambdaQueryWrapper<>();
    userTeamWrapper.eq(userId != null && userId > 0, UserTeam::getUserId, userId);
    userTeamWrapper.eq(teamId != null && teamId > 0, UserTeam::getTeamId, teamId);
    count = userTeamService.count(userTeamWrapper);
    if (count > 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "不能重复加入已加入的队伍");
    }
    //新增用户-关系表信息
    UserTeam userTeam = new UserTeam();
    userTeam.setUserId(userId);
    userTeam.setTeamId(teamId);
    userTeam.setJoinTime(new Date());
    boolean result = userTeamService.save(userTeam);
    return result;
}
```

##### 5. 用户可以退出队伍

请求参数：队伍 id

1. 校验请求参数

2. 校验队伍是否存在

3. 校验我是否已加入队伍

4. 退出队伍

   1. 如果队伍只剩一人，队伍解散

   2. 如果队伍还有其他人 

      1. 如果是队长退出队伍，权限转移给第二早加入的用户 —— 先来后到

         > 只用取 id 最小的 2 条数据

      2. 非队长，自己退出队伍

```java
@Data
public class TeamQuitRequest {
    private Long teamId;
}
```

```java
@PostMapping("/quit")
public Result quitTeam(@RequestBody TeamQuitRequest teamQuitRequest, HttpServletRequest request) {
    if (teamQuitRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"teamQuitRequest空值");
    }
    UserVO loginUser = userService.getLoginUser(request);
    boolean result=teamService.quitTeam(teamQuitRequest,loginUser);
    if (!result){
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "退出队伍失败");
    }
    return Result.success(true);
}
```

```java
/**
 * 退出队伍
 * @param teamQuitRequest
 * @param loginUser
 * @return
 */
boolean quitTeam(TeamQuitRequest teamQuitRequest, UserVO loginUser);
```

```java
@Override
@Transactional(rollbackFor = Exception.class)
public boolean quitTeam(TeamQuitRequest teamQuitRequest, UserVO loginUser) {
    //1. 校验请求参数
    if (teamQuitRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "teamQuitRequest空值");
    }
    // 2. 校验队伍是否存在
    Long teamId = teamQuitRequest.getTeamId();
    if (teamId == null || teamId <= 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍不存在");
    }
    Team team = this.getById(teamId);
    if (team == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍不存在");
    }
    // 3. 校验我是否已加入队伍
    Long userId = loginUser.getId();
    LambdaQueryWrapper<UserTeam> userTeamWrapper = new LambdaQueryWrapper<>();
    userTeamWrapper.eq(userId != null && userId > 0, UserTeam::getUserId, userId);
    userTeamWrapper.eq(UserTeam::getTeamId, teamId);
    long count = userTeamService.count(userTeamWrapper);
    if (count == 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "非队伍成员");
    }
    //查询队伍人数
    count = countTeamUserByTeamId(teamId);
    //4.退出队伍
    if (count == 1) {
        //4.1.退出队伍只剩一人，队伍解散
        //删除队伍
        this.removeById(teamId);
    } else {
        //4.2. 退出队伍还有其他人
        //如果是队长退出队伍，权限转移给第二早加入的用户 —— 先来后到
        if (team.getUserId().equals(userId)) {
            //取 id 最小的 2 条数据
            userTeamWrapper = new LambdaQueryWrapper<>();
            userTeamWrapper.eq(UserTeam::getTeamId, teamId);
            userTeamWrapper.last("order by id asc limit 2");
            List<UserTeam> userTeamList = userTeamService.list(userTeamWrapper);
            if (CollectionUtil.isEmpty(userTeamList) || userTeamList.size() <= 1) {
                throw new BusinessException(ErrorCode.SYSTEM_ERROR);
            }
            //取出第二早加入队伍的用户
            UserTeam nextUserTeam = userTeamList.get(1);
            Long nextTeamLeaderId = nextUserTeam.getUserId();
            //更新当前队伍的队长
            Team updateTeam = new Team();
            updateTeam.setId(teamId);
            updateTeam.setUserId(nextTeamLeaderId);
            boolean result = this.updateById(updateTeam);
            if (!result) {
                throw new BusinessException(ErrorCode.SYSTEM_ERROR, "更新队长失败");
            }
        }
    }
    //删除当前用户的用户-队伍关系，不管是一个人，队长，非队长，只要调用这个方法都要移除用户-队伍关系
    userTeamWrapper = new LambdaQueryWrapper<>();
    userTeamWrapper.eq(UserTeam::getTeamId, teamId);
    userTeamWrapper.eq(userId != null && userId > 0, UserTeam::getUserId, userId);
    return userTeamService.remove(userTeamWrapper);
}
/**
 * 根据队伍id查询用户关系表，获取队伍人数
 *
 * @param teamId
 * @return
 */
private long countTeamUserByTeamId(Long teamId) {
    LambdaQueryWrapper<UserTeam> userTeamWrapper = new LambdaQueryWrapper<>();
    userTeamWrapper.eq(UserTeam::getTeamId, teamId);
    return userTeamService.count(userTeamWrapper);
}
```



##### 6. 队长可以解散队伍

请求参数：队伍 id

业务流程：

1. 校验请求参数
2. 校验队伍是否存在
3. 校验你是不是队伍的队长
4. 移除所有加入队伍的关联信息
5. 删除队伍

```java
@Data
public class TeamDeleteRequest {
    private Long teamId;
}
```

```java
@PostMapping("/delete")
public Result deleteTeam(@RequestBody TeamDeleteRequest teamDeleteRequest, HttpServletRequest request) {
    if (teamDeleteRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "teamDeleteRequestko空值");
    }
    UserVO loginUser = userService.getLoginUser(request);
    boolean result = teamService.deleteTeam(teamDeleteRequest,loginUser);
    if (!result) {
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "队长解散队伍失败");
    }
    return Result.success(true);
}
```

```java
/**
 * 队长解散队伍
 *
 * @param teamDeleteRequest
 * @param loginUser
 * @return
 */
boolean deleteTeam(TeamDeleteRequest teamDeleteRequest, UserVO loginUser);
```

```java
@Override
@Transactional(rollbackFor = Exception.class)
public boolean deleteTeam(TeamDeleteRequest teamDeleteRequest, UserVO loginUser) {
    //1. 校验请求参数
    if (teamDeleteRequest == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "teamDeleteRequestko空值");
    }
    // 2. 校验队伍是否存在
    Long teamId = teamDeleteRequest.getTeamId();
    Team team = getTeamById(teamId);
    // 3. 校验你是不是队伍的队长
    if (!team.getUserId().equals(loginUser.getId())) {
        throw new BusinessException(ErrorCode.NO_AUTH, "非队长无权解散队伍");
    }
    // 4. 移除所有加入队伍的关联信息
    LambdaQueryWrapper<UserTeam> userTeamWrapper = new LambdaQueryWrapper<>();
    userTeamWrapper.eq(UserTeam::getTeamId, teamId);
    boolean result = userTeamService.remove(userTeamWrapper);
    if (!result) {
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "移除所有加入队伍的关联信息失败");
    }
    // 5. 删除队伍
    result = this.removeById(teamId);
    if (!result) {
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "删除队伍失败");
    }
    return true;
}
/**
 * 根据teamId查询team
 *
 * @param teamId
 * @return
 */
private Team getTeamById(Long teamId) {
    if (teamId == null || teamId <= 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍不存在");
    }
    Team team = this.getById(teamId);
    if (team == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍不存在");
    }
    return team;
}
```

##### 7. 获取当前用户已加入的队伍

```java
/**
 * 获取我加入的队伍
 * @param teamQueryRequest
 * @param request
 * @return
 */
@GetMapping("/list/mine/join")
public Result getMyJoinTeamList(TeamQueryRequest teamQueryRequest, HttpServletRequest request) {
    if (teamQueryRequest == null) {
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "队伍不存在");
    }
    //获取当前登录用户
    UserVO loginUser = userService.getLoginUser(request);
    //根据登录用户id查询该用户的用户-队伍关系
    LambdaQueryWrapper<UserTeam> userTeamWrapper=new LambdaQueryWrapper<>();
    userTeamWrapper.eq(UserTeam::getUserId,loginUser.getId());
    List<UserTeam> userTeamList = userTeamService.list(userTeamWrapper);
    //把用户-队伍关系根据队伍id分组，得到队伍-用户的映射map
    Map<Long, List<UserTeam>> listMap = userTeamList.stream().collect(Collectors.groupingBy(UserTeam::getTeamId));
    //从map中取出的keySet就是当前用户加入的队伍id列表
    ArrayList<Long> teamIdList = new ArrayList<>(listMap.keySet());
    //设置队伍id列表
    teamQueryRequest.setIds(teamIdList);
    //根据队伍id列表查询当前用户加入的队伍
    List<TeamUserVO> teamList = teamService.listTeams(teamQueryRequest,request,true);
    return Result.success(teamList);
}
```

##### 8. 获取当前用户创建的队伍

复用 listTeam 方法，只新增查询条件，不做修改（开闭原则）

```java
/**
 * 获取我创建的队伍
 * @param teamQueryRequest
 * @param request
 * @return
 */
@GetMapping("/list/mine/create")
public Result getMyCreateTeamList(TeamQueryRequest teamQueryRequest, HttpServletRequest request) {
    if (teamQueryRequest == null) {
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "队伍不存在");
    }
    //获取当前登录用户
    UserVO loginUser = userService.getLoginUser(request);
    //设置登录用户id
    teamQueryRequest.setUserId(loginUser.getId());
    //根据登录用户id查询该用户创建的队伍
    List<TeamUserVO> teamList = teamService.listTeams(teamQueryRequest,request, true);
    return Result.success(teamList);
}
```

##### 9. 分享队伍 ==> 邀请其他用户加入队伍 

1. 业务流程：

   1. 生成分享链接（分享二维码）
   2. 用户访问链接，可以点击加入

---



####  事务注解

@Transactional(rollbackFor = Exception.class)

要么数据操作都成功，要么都失败



## 前端开发

### 整合路由

Vue-Router 其实就是帮助你根据不同的 url 来展示不同的页面（组件），不用自己写 if / else

路由配置影响整个项目，所以建议单独用 config 目录、单独的配置文件去集中定义和管理。

有些组件库可能自带了和 Vue-Router 的整合，所以尽量先看组件文档、省去自己写的时间。

[安装 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/installation.html)

1.安装Vue Router

```sh
#如果安装失败，可能是项目正在运行，把项目停掉，删除node_modules目录，再安装
yarn add vue-router@4
```

![image-20230916102856564](assets/image-20230916102856564.png)

2. 引入vue-router

[入门 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/guide/)

![image-20230916110556135](assets/image-20230916110556135.png)

![image-20230916110836957](assets/image-20230916110836957.png)

![image-20230916111026425](assets/image-20230916111026425.png)

![image-20230916111325663](assets/image-20230916111325663.png)

![image-20230916111459403](assets/image-20230916111459403.png)

3. 修改通用布局

   ![image-20230916151307992](assets/image-20230916151307992.png)

   编程式路由参考[编程式导航 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/guide/essentials/navigation.html)

### 封装myAxios

1. 安装axios

```sh
#用来发请求
npm install axios
```

![image-20230919194157437](assets/image-20230919194157437.png)

创建实例[axios中文文档|axios中文网 | axios (axios-js.com)](http://axios-js.com/zh-cn/docs/#axios-create-config)

```js
import axios from "axios";
const my_axios = axios.create({
    baseURL: 'http://localhost:8080/api/',
    timeout: 100000,
    headers: {
        'Authorization': sessionStorage.getItem("token"),
    }
});
```

创建拦截器[axios中文文档|axios中文网 | axios (axios-js.com)](http://axios-js.com/zh-cn/docs/#拦截器)

```js
my_axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    console.log("我要发请求了",config)
    return config;
}, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
});

// 添加响应拦截器
my_axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    console.log("我收到响应了",response)
    return response
}, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
});

export default my_axios;
```

2. 允许请求携带Cookie

![image-20230921110151688](assets/image-20230921110151688.png)

### 封装获取当前登录用户功能

![image-20230921104224748](assets/image-20230921104224748.png)

### 搜索页

1.选择组件

![image-20230916114642295](assets/image-20230916114642295.png)

![image-20230916114750370](assets/image-20230916114750370.png)

![image-20230916131223513](assets/image-20230916131223513.png)

![image-20230916132158857](assets/image-20230916132158857.png)

2. 引入组件

![image-20230916144112168](assets/image-20230916144112168.png)

3. 编写搜索页

   1. 编写导航栏

      ![image-20230916144328890](assets/image-20230916144328890.png)

      ![image-20230916145103806](assets/image-20230916145103806.png)

      ![image-20230916145403396](assets/image-20230916145403396.png)

   2. 分割已选标签和选择标签

      ![image-20230916145221852](assets/image-20230916145221852.png)

   3. 渲染已选标签

      ![image-20230916150007197](assets/image-20230916150007197.png)

      ![image-20230916150254924](assets/image-20230916150254924.png)

   4. 引入TreeSelect组件

      ![image-20230916150405714](assets/image-20230916150405714.png)

      ![image-20230916150909719](assets/image-20230916150909719.png)

   5. 添加搜素页的路由

      ![image-20230916151107567](assets/image-20230916151107567.png)

   6. 最终效果

      ![image-20230916150959126](assets/image-20230916150959126.png)

### 用户信息页

把用户数据库中的用户信息展示到这个页面

1. 选择组件

   ![image-20230916152214015](assets/image-20230916152214015.png)

2. 编写用户类型-方便接受后端的数据

   ![image-20230916160254065](assets/image-20230916160254065.png)

3. 引入组件

   ![image-20230916160321644](assets/image-20230916160321644.png)

   ![image-20230916160455018](assets/image-20230916160455018.png)

   ![image-20230916160535677](assets/image-20230916160535677.png)

4. 效果

   ![image-20230916160600348](assets/image-20230916160600348.png)
   
   5. 修改用户信息页
   
   ![image-20230921105131541](assets/image-20230921105131541.png)
   
   ![image-20230921105342032](assets/image-20230921105342032.png)

### 用户信息修改页

1. 选择组件

   ![image-20230916163707773](assets/image-20230916163707773.png)

   ![image-20230916171923061](assets/image-20230916171923061.png)

2. 引入组件

   ![image-20230916172014807](assets/image-20230916172014807.png)

   ![image-20230916172354874](assets/image-20230916172354874.png)

   ![image-20230916172729570](assets/image-20230916172729570.png)

3. 添加路由

   ![image-20230916173335315](assets/image-20230916173335315.png)

4. 修改用户信息页

   ![image-20230916173316206](assets/image-20230916173316206.png)

5. 修改通用布局页

   ![image-20230916173448107](assets/image-20230916173448107.png)

6. 效果

   ![image-20230916173529752](assets/image-20230916173529752.png)

7. 修改用户信息修改页

![image-20230921105947435](assets/image-20230921105947435.png)

### 搜索结果页

1. 安装qs

```sh
#axios使用qs解决数组传参问题
npm i qs
```

在使用axios请求的过程中,直接在请求时传参数组，axios显示直接传数组去get请求时是 **tags[]=%E7%94%B7	&tags[]=Java** 我们如果想要没有 [] 连接的格式就需要进行`参数序列化`：使用`qs.stringify`,设置axios配置项中的 `paramsSerializer`

![image-20230919195802237](assets/image-20230919195802237.png)

```js
my_axios.get('/user/searchByTags', {
    params: {
      tags: tags
    },
    paramsSerializer: params => qs.stringify(params, {indices: false})
  })
//indices: false可以取消URL里数组的下标，如果不这么处理，后端收不到这个数组（名字因为加了下标而不匹配）
```

![image-20230919200130240](assets/image-20230919200130240.png)

3. 选择组件

![image-20230919200256361](assets/image-20230919200256361.png)

![image-20230919200353616](assets/image-20230919200353616.png)

4. 引入组件

![image-20230919200459410](assets/image-20230919200459410.png)

![image-20230919200947885](assets/image-20230919200947885.png)

5. 添加路由

![image-20230919201604552](assets/image-20230919201604552.png)

6. 修改搜索页

![image-20230919201123344](assets/image-20230919201123344.png)

![image-20230919201422277](assets/image-20230919201422277.png)

7. 修改user.d.ts文件

![image-20230919201800397](assets/image-20230919201800397.png)

8. 在搜素结果页发请求，获取数据，渲染页面

![image-20230919202710179](assets/image-20230919202710179.png)

![image-20230919204047018](assets/image-20230919204047018.png)

9. 效果

![image-20230919204220431](assets/image-20230919204220431.png)

![-](assets/image-20230919204238304.png)

10. 修改搜索结果页

![image-20230921155113015](assets/image-20230921155113015.png)

### 封装card组件

![image-20230921154447041](assets/image-20230921154447041.png)

![image-20230921160105261](assets/image-20230921160105261.png)

### 登录页

1. 选择组件

![image-20230921104427097](assets/image-20230921104427097.png)

2. 引入组件

![image-20230921104605948](assets/image-20230921104605948.png)

![image-20230921104759490](assets/image-20230921104759490.png)

3. 添加路由

![image-20230921105414644](assets/image-20230921105414644.png)

### 主页

![image-20230921155345911](assets/image-20230921155345911.png)

![image-20230921155757224](assets/image-20230921155757224.png)

### 组队

#### 改造个人页

1. 

![image-20230929192911992](assets/image-20230929192911992.png)

2. 选择组件

![image-20230929182952997](assets/image-20230929182952997.png)

![image-20230929193237288](assets/image-20230929193237288.png)

 #### 创建队伍

1. 选择组件

![image-20230929160137052](assets/image-20230929160137052.png)

![image-20230929160256427](assets/image-20230929160256427.png)

![image-20230929160328303](assets/image-20230929160328303.png)

2. 引入组件

```ts
app.use(Stepper);
app.use(Radio);
app.use(RadioGroup);
app.use(Picker);
app.use(DatetimePicker);
app.use(Popup);
```

3. 修改队伍页

![image-20230929162758781](assets/image-20230929162758781.png)

3. 添加路由

![image-20230929160620727](assets/image-20230929160620727.png)

![image-20230929143419028](assets/image-20230929143419028.png)

5. 开发页面

![image-20230929160808100](assets/image-20230929160808100.png)

![image-20230929161058655](assets/image-20230929161058655.png)

![image-20230929161148642](assets/image-20230929161148642.png)

![image-20230929161230574](assets/image-20230929161230574.png)

![image-20230929161458754](assets/image-20230929161458754.png)

![image-20230929161731336](assets/image-20230929161731336.png)

#### 展示队伍&加入队伍

1. 封装队伍列表组件

![image-20230929162218716](assets/image-20230929162218716.png)

![image-20230929162347552](assets/image-20230929162347552.png)

2. 修改队伍页面

![image-20230929162525542](assets/image-20230929162525542.png)

3. 封装队伍类型

![image-20230929163053782](assets/image-20230929163053782.png)

4. 封装队伍状态枚举对象

![image-20230929163133640](assets/image-20230929163133640.png)

#### 搜索队伍

1. 选择组件

![image-20230929163238705](assets/image-20230929163238705.png)

2. 开发页面

![image-20230929165407905](assets/image-20230929165407905.png)

![image-20230929165531651](assets/image-20230929165531651.png)

![image-20230929165604236](assets/image-20230929165604236.png)

#### 更新队伍

1. 点击更新队伍获取队伍id

![image-20230929184112208](assets/image-20230929184112208.png)

![image-20230929184223485](assets/image-20230929184223485.png)

![image-20230929184306701](assets/image-20230929184306701.png)

2. 更新队伍

![image-20230929184612001](assets/image-20230929184612001.png)

![image-20230929184704538](assets/image-20230929184704538.png)

3. 添加路由

![image-20230929184801831](assets/image-20230929184801831.png)

#### 查看创建的队伍

1. 复制页面

![image-20230929193841009](assets/image-20230929193841009.png)

2. 添加路由

![image-20230929193513004](assets/image-20230929193513004.png)

#### 查看加入的队伍

1. 复制页面

![image-20230929193431796](assets/image-20230929193431796.png)

![image-20230929193446496](assets/image-20230929193446496.png)

2. 添加路由

![image-20230929193658289](assets/image-20230929193658289.png)

#### 解散队伍&退出队伍

![image-20230929200329839](assets/image-20230929200329839.png)

![image-20230929200404531](assets/image-20230929200404531.png)



## 数据库查询慢怎么办？

使用缓存：提前把数据取出来保存好（通常保存到读写更快的介质，比如内存），就可以更快地读写。

### 缓存的实现

- Redis（分布式缓存）（**采用**）
- memcached（分布式）
- Etcd（云原生架构的一个分布式存储，**存储配置**，扩容能力）（**需要学习**）

---

- ehcache（单机）

- 本地缓存（Java 内存 Map）
- Caffeine（Java 内存缓存，高性能）
- Google Guava

### 设计缓存 key

不同用户看到的数据不同

systemId:moduleId:func:options（不要和别人冲突）

yingluo:user:recommed:userId

**redis 内存不能无限增加，一定要设置过期时间！！！**  

### 给主页推荐功能设置缓存

1. Service层

```java
@Override
public List<UserDTO> usersRecommend(long currentPage, long pageSize, HttpServletRequest request) {
    //获取当前登录用户
    UserDTO loginUser = this.getLoginUser(request);
    //拼接key
    String key = RECOMMEND_USUERS_KEY + loginUser.getId();
    //读取缓存
    String userDTOListJson = stringRedisTemplate.opsForValue().get(key);
    //反序列化
    JSONArray objects = JSONUtil.parseArray(userDTOListJson);
    //遍历数组，把user添加到List集合
    Iterator<Object> iterator = objects.stream().iterator();
    List userDTOList = new ArrayList();
    while (iterator.hasNext()) {
        userDTOList.add(iterator.next());
    }
    //如果有缓存直接读缓存
    if (userDTOList != null && !userDTOList.isEmpty()) {
        return userDTOList;
    }
    //如果有没缓存查数据库
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper();
    Page<User> page = this.page(new Page<User>(currentPage, pageSize), wrapper);
    userDTOList = page.getRecords().stream().map(user1 -> {
        return BeanUtil.copyProperties(user1, UserDTO.class);
    }).collect(Collectors.toList());
    //将查询到的数据添加到缓存
    userDTOListJson = JSONUtil.toJsonStr(userDTOList);
    stringRedisTemplate.opsForValue().set(key, userDTOListJson,3, TimeUnit.MINUTES);
    return userDTOList;
}
```

2. Controller层

```java
@GetMapping("/recommend")
public Result usersRecommend(long currentPage, long pageSize, HttpServletRequest request) {
    //返回数据
    return Result.success(userService.usersRecommend(currentPage, pageSize, request));
}
```

### Redis在Java 里的实现方式

#### Spring Data Redis（推荐）

Spring Data：通用的数据访问框架，定义了一组 **增删改查** 的接口

mysql、redis、jpa

[spring-data-redis](https://mvnrepository.com/artifact/org.springframework.data/spring-data-redis)

1）引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2）配置 Redis 地址

```yml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password: 123
    database: 0
```

#### Jedis

独立于 Spring 操作 Redis 的 Java 客户端

要配合 Jedis Pool 使用



#### Lettuce

**高阶** 的操作 Redis 的 Java 客户端

异步、连接池



#### Redisson

分布式操作 Redis 的 Java 客户端，让你像在使用本地的集合一样操作 Redis（分布式 Redis 数据网格）



#### JetCache 



对比

1. 如果你用的是 Spring，并且没有过多的定制化要求，可以用 Spring Data Redis，最方便
2. 如果你用的不是 SPring，并且追求简单，并且没有过高的性能要求，可以用 Jedis + Jedis Pool
3. 如果你的项目不是 Spring，并且追求高性能、高定制化，可以用 Lettuce，支持异步、连接池

---

- 如果你的项目是分布式的，需要用到一些分布式的特性（比如分布式锁、分布式集合），推荐用 redisson



### 缓存预热

优点：

1. 解决第一个用户访问还是很慢的问题，可以让用户始终访问很快，也能一定程度上保护数据库

缺点：

1. 增加开发成本（额外的开发、设计）
2. 预热的时机和时间如果错了，有可能你缓存的数据不对或者太老
3. 需要占用额外空间

#### 怎么缓存预热？

1. 定时（用定时任务，每天刷新所有用户的推荐列表）
2. 模拟触发（手动触发）

注意点：

1. 缓存预热的意义（新增少、总用户多）
2. 缓存的空间不能太大，要预留给其他缓存空间
3. 缓存数据的周期（此处每天一次）

#### 定时任务实现

1. **Spring Scheduler（spring boot 默认整合了）** 
2. Quartz（独立于 Spring 存在的定时任务框架）
3. XXL-Job 之类的分布式任务调度平台（界面 + sdk）

第一种方式：

1. 启动类开启 @EnableScheduling

   

2. 给要定时执行的方法添加 @Scheduling 注解，指定 cron 表达式或者执行频率

```java
/**
 * 缓存预热任务
 */
@Component
public class PreCacheJob {
    @Resource
    private UserService userService;
    @Resource
    private StringRedisTemplate stringRedisTemplate;
    //重点用户
    List<Long> mainUserList = Arrays.asList(2l);

    //每天13：11执行
    @Scheduled(cron = "0 11 13 * * *")//秒 分 时 日 月 年
    public void doCacheRecommendUser() {
        for (Long userId : mainUserList) {
            //拼接key
            String key = RECOMMEND_USUERS_KEY + userId;
            //缓存20条
            LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper();
            Page<User> page = userService.page(new Page<User>(1, 20), wrapper);

            List<UserDTO> userDTOList = page.getRecords().stream().map(user1 -> {
                return BeanUtil.copyProperties(user1, UserDTO.class);
            }).collect(Collectors.toList());
            //将查询到的数据添加到缓存
            String userDTOListJson = JSONUtil.toJsonStr(userDTOList);
            stringRedisTemplate.opsForValue().set(key, userDTOListJson, 3, TimeUnit.MINUTES);
        }
    }

}
```

不要去背 cron 表达式！！！！！

- https://cron.qqe2.com/
- https://www.matools.com/crontab/

---

### 控制定时任务的执行

不进行控制的缺点：

1. 浪费资源，想象 10000 台服务器同时执行定时任务
2. 脏数据，比如重复插入

****

**要控制定时任务在同一时间只有 1 个服务器能执行。怎么做？**

1. 分离定时任务程序和主程序，只在 1 个服务器运行定时任务。成本太大

2. 写死配置，每个服务器都执行定时任务，但是只有 ip 符合配置的服务器才真实执行业务逻辑，其他的直接返回。成本最低；但是我们的 IP 可能是不固定的，把 IP 写的太死了

3. 动态配置，配置是可以轻松的、很方便地更新的（**代码无需重启**），但是只有 ip 符合配置的服务器才真实执行业务逻辑。

   - 数据库
   - Redis
   - 配置中心（Nacos、Apollo、Spring Cloud Config）

   问题：服务器多了、IP 不可控还是很麻烦，还是要人工修改

4. 分布式锁，只有抢到锁的服务器才能执行业务逻辑。坏处：增加成本；好处：不用手动配置，多少个服务器都一样。

#### 锁

有限资源的情况下，控制同一时间（段）只有某些线程（用户 / 服务器）能访问到资源。

Java 实现锁：synchronized 关键字、并发包的类

问题：只对单个 JVM 有效

####  分布式锁

为啥需要分布式锁？

1. 有限资源的情况下，控制同一时间（段）只有某些线程（用户 / 服务器）能访问到资源。
2. 单个锁只对单个 JVM 有效

#### 分布式锁实现的关键

##### 抢锁机制

保证同一时间只有 1 个服务器能抢到锁

核心思想 就是：先来的人先把数据改成自己的标识（服务器 ip），后来的人发现标识已存在，就抢锁失败，继续等待。

等先来的人执行方法结束，把标识清空，其他的人继续抢锁。

MySQL 数据库：select for update 行级锁（最简单）

（乐观锁）

✔ Redis 实现：内存数据库，读写速度快 。支持 setnx、lua 脚本，比较方便我们实现分布式锁。

setnx：set if not exists 如果不存在，则设置；只有设置成功才会返回 true，否则返回 false

##### 注意事项

1. 用完锁要释放（腾地方）√

2. 锁一定要加过期时间 √

3. 如果方法执行时间过长，锁提前过期了？

   问题：

   1. 连锁效应：释放掉别人的锁
   2. 这样还是会存在多个方法同时执行的情况

​	解决方案：续期

```java
boolean end = false;

new Thread(() -> {
    if (!end)}{
    续期
})

end = true;

```

4. 释放锁的时候，有可能先判断出是自己的锁，但这时锁过期了，最后还是释放了别人的锁

   ```java
   // 原子操作
   if(get lock == A) {
       // set lock B
       del lock
   }
   ```

   Redis + lua 脚本实现

5. Redis 如果是集群（而不是只有一个 Redis），如果分布式锁的数据不同步怎么办？

https://blog.csdn.net/feiying0canglang/article/details/113258494

#### Redisson 实现分布式锁

Redisson 是一个 java 操作 Redis 的客户端，实现了很多 Java 里支持的接口和数据结构，**提供了大量的分布式数据集来简化对 Redis 的操作和使用，可以让开发者像使用本地集合一样使用 Redis，完全感知不到 Redis 的存在。**

**2 种引入方式**

1. spring boot starter 引入（不推荐，版本迭代太快，容易冲突）https://github.com/redisson/redisson/tree/master/redisson-spring-boot-starter
2. 直接引入：https://github.com/redisson/redisson#quick-start

##### 定时任务  + 锁

1. waitTime 设置为 0，只抢一次，抢不到就放弃
2. 注意释放锁要写在 finally 中

##### 实现代码

```java
/**
 * 缓存预热任务
 */
@Component
@Slf4j
public class PreCacheJob {
    @Resource
    private UserService userService;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Resource
    private RedissonClient redissonClient;
    //重点用户
    List<Long> mainUserList = Arrays.asList(2l);

    //每天执行
    @Scheduled(cron = "0 37 16 * * *")
    public void doCacheRecommendUser() {
        RLock lock = redissonClient.getLock(PRECACHEJOB_DOCACHE_LOCK);
        try {
            // 将waittime设置为0，那么意味着不会等待任何时间，直接尝试获取锁，如果获取到了锁，那么方法会返回true，否则返回false。
            // 只有一个线程能获取锁
            if (lock.tryLock(0,-1, TimeUnit.SECONDS)) {
                log.info("getLock");
                for (Long userId : mainUserList) {
                    //拼接key
                    String key = RECOMMEND_USUERS_KEY + userId;
                    //缓存20条
                    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper();
                    Page<User> page = userService.page(new Page<User>(1, 20), wrapper);

                    List<UserDTO> userDTOList = page.getRecords().stream().map(user1 -> {
                        return BeanUtil.copyProperties(user1, UserDTO.class);
                    }).collect(Collectors.toList());
                    //将查询到的数据添加到缓存
                    String userDTOListJson = JSONUtil.toJsonStr(userDTOList);
                    stringRedisTemplate.opsForValue().set(key, userDTOListJson, RECOMMEND_USUERS_KEY_TTL, TimeUnit.MINUTES);
                }
                //只能释放自己加的锁

            }
        } catch (InterruptedException e) {
            log.error(e.getMessage());
        } finally {
            if (lock.isHeldByCurrentThread()){
                log.info("getLock");
                lock.unlock();
            }
        }

    }

}
```

##### 看门狗机制

> redisson 中提供的续期机制

开一个监听线程，如果方法还没执行完，就帮你重置 redis 锁的过期时间。

原理：

1. 监听当前线程，默认过期时间是 30 秒，每 10 秒续期一次（补到 30 秒），续的时间不太长是为了防止服务器宕机，锁无法释放
2. 如果线程挂掉（注意 debug 模式也会被它当成服务器宕机），则不会续期

https://blog.csdn.net/qq_26222859/article/details/79645203

---

Zookeeper 实现（不推荐）

## 随机匹配



**前端不同页面怎么传递数据**？

1. **url querystring（xxx?id=1）** 比较适用于页面跳转
2. **url（/team/:id，xxx/1）**
3. hash (/team#1)
4. localStorage
5. **context（全局变量，同页面或整个项目要访问公共变量）**



### 随机匹配后端

> 为了帮大家更快地发现和自己兴趣相同的朋友

匹配 1 个还是匹配多个？

答：匹配多个，并且按照匹配的相似度从高到低排序

怎么匹配？（根据什么匹配）

答：标签 tags

> 还可以根据 user_team 匹配加入相同队伍的用户

本质：找到有相似标签的用户

#### 1. 怎么匹配

1. 找到有共同标签最多的用户（TopN）
2. 共同标签越多，分数越高，越排在前面
3. 如果没有匹配的用户，随机推荐几个（降级方案）

编辑距离算法：https://blog.csdn.net/DBC_121/article/details/104198838

> 最小编辑距离：字符串 1 通过最少多少次增删改字符的操作可以变成字符串 2

```java
/**
 * 编辑距离算法（用于计算最相似的两个标签列表）标签列表的每个标签字符串，相当于算法修改前字符串的每个字符
 * 原理: https://blog.csdn.net/DBC_121/article/details/104198838
 *
 * @param tagList1
 * @param tagList2
 * @return
 */
public static int minDistance1(List<String> tagList1, List<String> tagList2) {
    int n = tagList1.size();
    int m = tagList2.size();
    if (n * m == 0)
        return n + m;
    int[][] d = new int[n + 1][m + 1];
    for (int i = 0; i < n + 1; i++) {
        d[i][0] = i;
    }
    for (int j = 0; j < m + 1; j++) {
        d[0][j] = j;
    }
    for (int i = 1; i < n + 1; i++) {
        for (int j = 1; j < m + 1; j++) {
            int left = d[i - 1][j] + 1;
            int down = d[i][j - 1] + 1;
            int left_down = d[i - 1][j - 1];
            if (!tagList1.get(i - 1).equals(tagList2.get(j - 1))) {
                left_down += 1;
            }
            d[i][j] = Math.min(left, Math.min(down, left_down));
        }
    }
    return d[n][m];
}
```

余弦相似度算法：https://blog.csdn.net/m0_55613022/article/details/125683937（如果需要带权重计算，比如学什么方向最重要，性别相对次要）



#### 2. 怎么对所有用户匹配，取 TOP

直接取出所有用户，依次和当前用户计算分数，取 TOP N（54 秒）

优化方法：

1. 切忌不要在数据量大的时候循环输出日志（取消掉日志后 20 秒）

2. Map 存了所有的分数信息，占用内存

   解决：维护一个固定长度的有序集合（sortedSet），只保留分数最高的几个用户（时间换空间）（暂未使用）

   e.g.【3, 4, 5, 6, 7】取 TOP 5，分数为 1 的用户就不用放进去了

3. 细节：剔除自己 √

4. 尽量只查需要的数据：

   1. 过滤掉标签为空的用户 √
   2. 根据部分标签取用户（前提是能区分出来哪个标签比较重要）
   3. 只查需要的数据（比如 id 和 tags） √（7.0s）

5. 提前查？（定时任务）

   1. 提前把所有用户给缓存（不适用于经常更新的数据）
   2. 提前运算出来结果，缓存（针对一些重点用户，提前缓存）

```java
/**
 * 获取与当前用户相似度最高的用户
 * @param num 需要匹配用户的个数
 * @param request
 * @return
 */
@GetMapping("/match")
public Result usersMatch(long num, HttpServletRequest request) {
    if (num<=0 || num >20){
        throw new BusinessException(ErrorCode.PARAMS_ERROR);
    }
    //获取登录用户
    UserVO loginUser = userService.getLoginUser(request);
    //封装返回匹配的用户
    return Result.success(userService.usersMatch(num,loginUser));
}
```

```java
/**
 * 获取与当前用户相似度最高的用户
 * @param num
 * @param loginUser
 * @return
 */
List<UserVO> usersMatch(long num, UserVO loginUser);
```

```java
@Override
public List<UserVO> usersMatch(long num, UserVO loginUser) {
    //获取当前用户的标签列表
    String loginUserTags = loginUser.getTags();
    Gson gson = new Gson();
    //把当前用户的标签列表转换成List集合
    List<String> loginUserTagList = gson.fromJson(loginUserTags, new TypeToken<List<String>>() {
    }.getType());
    //查询所有标签字段不为空的用户，且只查用id和tags字段
    LambdaQueryWrapper<User> userWrapper = new LambdaQueryWrapper<>();
    userWrapper.select(User::getId, User::getTags);
    userWrapper.isNotNull(User::getTags);
    List<User> users = this.list(userWrapper);
    //用户==>相似度
    List<Pair<User, Long>> list = new ArrayList<>();
    //遍历查到的每个用户
    for (User user : users) {
        //获取用户的标签
        String userTags = user.getTags();
        //过滤掉无标签的用户和当前用户
        if (StringUtils.isBlank(userTags) || user.getId().equals(loginUser.getId())) {
            continue;
        }
        //把用户的标签列表转换成List集合
        List<String> userTagList = gson.fromJson(userTags, new TypeToken<List<String>>() {
        }.getType());
        //使用编辑距离算法，计算出每个用户的标签和当前用户的标签的编辑距离
        long distance = AlgorithmUtil.minDistance1(loginUserTagList, userTagList);
        //存到list集合中
        list.add(new Pair<>(user, distance));
    }
    //按编辑距离升序排序
    List<Pair<User, Long>> topUserPairList = list.stream()
            .sorted((p1, p2) -> (int) (p1.getValue() - p2.getValue()))
            .limit(num).collect(Collectors.toList());
    //从topUserPairList取出userId
    List<Long> userIdList = topUserPairList.stream()
            .map(pair -> pair.getKey().getId())
            .collect(Collectors.toList());
    userWrapper = new LambdaQueryWrapper<>();
    userWrapper.in(User::getId,userIdList);
    //根据userId查询用户，然后返回
    Map<Long, List<UserVO>> userIdUserMap = this.list(userWrapper).stream()
            .map(user -> BeanUtil.copyProperties(user, UserVO.class))
            .collect(Collectors.groupingBy(UserVO::getId));
    List<UserVO> finalUserList=new ArrayList<>();
    for (Long userId : userIdList) {
        finalUserList.add(userIdUserMap.get(userId).get(0));
    }
    return finalUserList;
}
```

检索 => 召回 => 粗排 => 精排 => 重排序等等

检索：尽可能多地查符合要求的数据（比如按记录查）

召回：查询可能要用到的数据（不做运算）

粗排：粗略排序，简单地运算（运算相对轻量）

精排：精细排序，确定固定排位

### 随机匹配前端

1. 选择组件

![image-20230930141546873](assets/image-20230930141546873.png)

2. 引入组件

![image-20230930141610817](assets/image-20230930141610817.png)

3. 修改主页

![image-20230930141738953](assets/image-20230930141738953.png)

![image-20230930141959562](assets/image-20230930141959562.png)

![image-20230930142029568](assets/image-20230930142029568.png)

![image-20230930142221483](assets/image-20230930142221483.png)

### 分表学习建议

mycat、sharding sphere 框架

一致性 hash



## 优化

### 加载骨架屏特效 ✔

解决：van-skeleton 组件

1. 选择组件

![image-20230930144636062](assets/image-20230930144636062.png)

2. 引入组件

![image-20230930144702137](assets/image-20230930144702137.png)

3. 修改UserCardList组件

![image-20230930145054310](assets/image-20230930145054310.png)

4. 修改主页

![image-20230930145249504](assets/image-20230930145249504.png)

![image-20230930145403562](assets/image-20230930145403562.png)

### 前端导航栏死【标题】问题 ✔

解决：使用 router.beforeEach，根据要跳转页面的 url 路径 匹配 config/routes 配置的 title 字段。

![image-20230930161750439](assets/image-20230930161750439.png)

![image-20230930161915241](assets/image-20230930161915241.png)

### 队伍操作权限控制✔

加入队伍： 仅非队伍创建人、且未加入队伍的人可见

更新队伍：仅创建人可见

解散队伍：仅创建人可见

退出队伍：仅创建人已加入队伍的人可见

![image-20230930171308929](assets/image-20230930171308929.png)

### 强制登录，自动跳转到登录页✔

解决：axios 全局配置响应拦截、并且添加重定向

![image-20230930175044992](assets/image-20230930175044992.png)

![image-20230930175157861](assets/image-20230930175157861.png)

![image-20230930175315930](assets/image-20230930175315930.png)

### 区分公开和加密房间&加入有密码的房间，要指定密码✔

1. 选择组件

![image-20230930190059022](assets/image-20230930190059022.png)

![image-20230930190200201](assets/image-20230930190200201.png)

2. 引入组件

![image-20230930190245632](assets/image-20230930190245632.png)

3. 区分公开和加密

![image-20230930190448572](assets/image-20230930190448572.png)

![image-20230930190520995](assets/image-20230930190520995.png)

![image-20230930190735821](assets/image-20230930190735821.png)

![image-20230930190853797](assets/image-20230930190853797.png)

4. 加入有密码的房间，要指定密码

![image-20230930191259709](assets/image-20230930191259709.png)

![image-20230930191321307](assets/image-20230930191321307.png)

![image-20230930191455572](assets/image-20230930191455572.png)

![image-20230930191527762](assets/image-20230930191527762.png)

### 展示已加入队伍人数✔

1. 后端在service层中的listTeams方法末尾加入这段逻辑，来设置队伍人数

```java
//查询已加入队伍的所有用户
userTeamQueryWrapper = new LambdaQueryWrapper<>();
userTeamQueryWrapper.in(UserTeam::getTeamId, teamIdList);
List<UserTeam> userTeamList = userTeamService.list(userTeamQueryWrapper);
// 队伍 id => 加入这个队伍的用户列表
Map<Long, List<UserTeam>> teamIdUserTeamMap = userTeamList.stream().collect(Collectors.groupingBy(UserTeam::getTeamId));
//给每个队伍设置队伍人数
//getOrDefault是map的一个方法，如果这个键teamId的vlaue为null没有队员，就给个空集合
teamUserVOList.forEach(team -> team.setHasJoinNum(teamIdUserTeamMap.getOrDefault(team.getId(), new ArrayList<>()).size()));
```

2. 前端

![image-20230930194309992](assets/image-20230930194309992.png)

![image-20230930194330716](assets/image-20230930194330716.png)

### 并发请求重复加入队伍可能出现问题（加锁、分布式锁）✔

使用Redisson的分布式锁，锁是teamId

相同的用户线程并发请求加入相同的队伍时，同时只有一个用户线程能够拿到锁、加入不相同的队伍时，并行执行；不同用户线程，加入相同队伍时，只有一个用户线程能拿到锁，加入不同队伍时，不同用户线程能够并行执行

```java
@Override
@Transactional(rollbackFor = Exception.class)
public long addTeam(Team team, UserVO loginUser) {
    //1. 请求参数是否为空？
    if (team == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "team空值");
    }
    // 2. 是否登录，未登录不允许创建
    if (loginUser == null) {
        throw new BusinessException(ErrorCode.NO_LOGIN, "未登录");
    }
    // 3. 校验信息
    //    1. 队伍人数 > 1 且 <= 10
    Integer maxNum = Optional.ofNullable(team.getMaxNum()).orElse(0);
    if (maxNum <= 1 || maxNum > 10) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍人数不满足要求");
    }
    //    2. 队伍标题 <= 20
    String name = team.getName();
    if (StringUtils.isBlank(name) || name.length() > 20) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍标题不满足要求");
    }
    //    3. 描述 <= 512
    String description = team.getDescription();
    if (description.length() > 512) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍描述过长");
    }
    //    4. status 是否公开（int）不传默认为 0（公开）
    Integer status = Optional.ofNullable(team.getStatus()).orElse(0);
    TeamStatusEnum statusEnum = TeamStatusEnum.getEnumByValue(status);
    if (statusEnum == null) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍状态不满足要求");
    }
    //    5. 如果 status 是加密状态，一定要有密码，且密码 <= 32
    String password = team.getPassword();
    if (TeamStatusEnum.SECRET.equals(statusEnum) && (StringUtils.isBlank(password) || password.length() > 32)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍密码设置不满足要求");
    }
    //    6. 超时时间 > 当前时间
    Date expireTime = team.getExpireTime();
    if (new Date().after(expireTime)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "队伍已过期");
    }
    Long teamId = team.getId();
    // 只有一个线程能获取锁
    RLock lock = redissonClient.getLock(JOINTEAM_DOJOIN_LOCK+":"+teamId);
    //    7. 校验用户最多创建 5 个队伍
    try {
        while (true){
            if (lock.tryLock(0,-1, TimeUnit.SECONDS)) {
                log.info("getLock");
                LambdaQueryWrapper<Team> queryWrapper = new LambdaQueryWrapper<>();
                Long userId = loginUser.getId();
                queryWrapper.eq(Team::getUserId, userId);
                long hasTeamNum = this.count(queryWrapper);
                if (hasTeamNum >= 5) {
                    throw new BusinessException(ErrorCode.PARAMS_ERROR, "单个用户最多创建5个用户");
                }
                // 4. 插入队伍信息到队伍表
                team.setUserId(userId);
                boolean result = this.save(team);
                if (!result || teamId == null) {
                    throw new BusinessException(ErrorCode.SYSTEM_ERROR, "创建队伍失败");
                }
                // 5. 插入用户 -队伍关系到关系表
                UserTeam userTeam = new UserTeam();
                userTeam.setUserId(userId);
                userTeam.setTeamId(teamId);
                userTeam.setJoinTime(new Date());
                result = userTeamService.save(userTeam);
                if (!result) {
                    throw new BusinessException(ErrorCode.SYSTEM_ERROR, "创建用户-队伍失败");
                }
            }
        }
    } catch (InterruptedException e) {
        log.error(e.getMessage());
    } finally {
        if (lock.isHeldByCurrentThread()){
            log.info("getLock");
            lock.unlock();
        }
    }
    return teamId;
}
```

## 上线

区分多环境：前端区分开发和线上接口，后端 prod 改为用线上公网可访问的数据库

### 前端

![image-20231001164544429](assets/image-20231001164544429.png)

前端：Vercel（免费）

https://vercel.com/

1.  打包

![image-20231001145908035](assets/image-20231001145908035.png)

2. 进入打包好的目录，使用serve命令运行

```sh
#没有serve,安装
npm i -g serve
```

![image-20231001150219848](assets/image-20231001150219848.png)

3. 使用Vercel部署项目

```sh
#安装vercel，以管理员身份打开命令提示符
npm i -g vercel
```

![image-20231001152901451](assets/image-20231001152901451.png)

4. 注册https://vercel.com/

5. 登录 

![image-20231001154018478](assets/image-20231001154018478.png)

6. 部署

```
vercel --prod
```

![image-20231001154424703](assets/image-20231001154424703.png)

![image-20231001154605927](assets/image-20231001154605927.png)

![image-20231001164724302](assets/image-20231001164724302.png)

![image-20231001164755245](assets/image-20231001164755245.png)

![image-20231001154746030](assets/image-20231001154746030.png)

### 后端：微信云托管（部署容器的平台，付费）

https://cloud.weixin.qq.com/cloudrun/service

1. 注册公众号
2. 新建服务

![image-20231001161303850](assets/image-20231001161303850.png)

![image-20231001161421174](assets/image-20231001161421174.png)

![image-20231001161504823](assets/image-20231001161504823.png)

3. 编写DockerFile

```dockerfile
#指定基础镜像
FROM maven:3.5-jdk-8-alpine as builder
#指定镜像的工作目录
WORKDIR /app
#把需要的本地文件复制到容器镜像/app工作目录中
COPY pom.xml .
COPY src ./src
#用RUN执行maven的打包命令,至此镜像制作完成
RUN mvn package -DskipTests
#之后使用该镜像运行容器时,会自动执行以下命令,启动
CMD ["java","-jar","/app/target/userCenter-backend-0.0.1-SNAPSHOT.jar","--spring.profiles.active=prod"]
```

![image-20231001162429133](assets/image-20231001162429133.png)

4. 把后端项目打成压缩包

![image-20231001162148886](assets/image-20231001162148886.png)

5. 上传以及设置

![image-20231001162601817](assets/image-20231001162601817.png)

![image-20231001162622691](assets/image-20231001162622691.png)

![image-20231001174605474](assets/image-20231001174605474.png)

**（免备案！！！）**

这个项目需要redis，微信云托管上没有redis所以发布会失败

## todo

队伍过期倒计时

队伍分享

## 如何改造成小程序？

**cordova、跨端开发框架 taro、uniapp**


