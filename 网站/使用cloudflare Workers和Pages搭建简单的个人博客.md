<<<<<<< HEAD
<<<<<<< HEAD
# 使用cloudflare Workers和Pages搭建简单的个人博客

## 写在前面

首先这两个是啥，官方的话如下：

[Workers]([Cloudflare 员工](https://developers.cloudflare.com/workers/))

> Cloudflare Workers 提供了一个[无服务器](https://www.cloudflare.com/learning/serverless/what-is-serverless/)执行环境，允许您创建新应用程序或增强现有应用程序，而无需配置或维护基础设施。

[Pages]([Cloudflare Pages documentation](https://developers.cloudflare.com/pages/))

> 创建可立即部署到 Cloudflare 全球网络的全栈应用程序

我的理解是，Workers可以运行一些简单的脚本。Pages类似于可以提供一个简单的前端静态页面，当然功能不仅于此。

下面列出free计划中这两个的使用限制。

| Workers |      |
| ------ | ---- |
| 请求    | 10万 / 天 |
| 期间    | 期间不收费 |
| CPU时间 | 每次调用 10 毫秒的 CPU 时间 |

Pages的free计划有一些限制，这里不方便列表，请自行查看源文链接[使用限制 |Cloudflare Pages 文档](https://developers.cloudflare.com/pages/platform/limits/)





顺带一提，在cloudflare（以下简称cf）的free计划中，帮我们提供了免费的D1数据库和KV命名空间。

> [D1 SQL Database]([Cloudflare D1](https://developers.cloudflare.com/d1/)):	从 Workers 和 Pages 项目创建新的无服务器 SQL 数据库来进行查询。
>
> [KV]([Cloudflare Workers KV | Cloudflare Workers KV](https://developers.cloudflare.com/kv/)):	在 Cloudflare 网络中存储应用程序数据，并从 Workers 访问键值对。

下面列出free计划中这两个的使用限制。

| D1数据库             | workers Free |      | KV       | Free      |
| -------------------- | ------------ | ---- | :------- | --------- |
| 读取的行数           | 500万 / 天   |      | 读取请求 | 10万 / 天 |
| 写入的行数           | 10万 / 天    |      | 写入请求 | 1000 / 天 |
| 存储空间（每GB存储） | 5 GB (共)    |      | 删除请求 | 1000 / 天 |
|                      |              |      | 列出请求 | 1000 / 天 |
|                      |              |      | 存储数据 | 1GB       |

根据上面的条件，对于个人简单博客网站的搭建其实完全够了，每日的请求数量限制对于一般小型的也完全够用。Workers只提供一个脚本的运行，目前官方提供的有typescript与javasrcipt，还有python。对于本人的博客后端仅仅是查查数据库这种简单的完全可以实现。

**根据官方文档的介绍，在实际上利用[Pages]([Cloudflare Pages documentation](https://developers.cloudflare.com/pages/))与[Functions | Cloudflare Pages docs](https://developers.cloudflare.com/pages/functions/)实际上就可以完成地搭建一个人网站，在不使用额外服务器的条件上（为什么没有用而是选择了Workers而额外来个脚本呢，好吧，其实是最初没看到~~）。**



## 开始

于是，话不多说，我们准备开始。

需要的工具有：

- 一个cf账号
- 灵巧的手

### Workers

1. 登录Cloudflare,没有账号的请自行注册。然后选择**Free**计划即可，找不到图了。





2. 先创建Workers

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/c66aed16f5694beda6670524e8559cb9.png)





3. 如果方便的可以使用下面的模版，我这里就不使用了，后面需要什么再加

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/527b6330a9654a139d21c166aadee84a.png)





4. 随便命一个名字后点击部署即可

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/fa76abe1ea7646199ac0c262b249076c.png)





5. 之后应该是这个页面,可以点击其他的，不过我们暂时不管，点击“继续处理项目”就行

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/8b7a837c55484d63871764eb0e8ab86a.png)





6. 现在我们让为其添加数据库

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/1c7a99cbb8ba4672a7d0a92718e84cc6.png)





7. 随便输入名字后创建即可，只要与现存的不重名，点击创建

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/fd9ed9817872466aab4cb3920eac7415.png)





8. 成功后应该是这个页面

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/2ee7226b6c1049f08c999d13ec54c84e.png)





9. 现在我们去绑定这两个

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/bfe760b1a292409ca99dfa93aed2785a.png)





10. 如图

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/460feb438db94690bc20fbed7183b0bc.png)





11. 变量名称随便，然后选择你刚才的数据库后点击部署

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/f5de18573ab949c192b86841f9024095.png)





12. 然后去，右上角编辑代码

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/369c7e50358941b593c63d59d9a43b09.png)





13. 最后，index.js代码中那个函数体里面添加即可

```javascript
const { DATABASE } = env;

//执行方式大概像这样
const stmt = DATABASE.prepare("SELECT * FROM TABLENAME");
const { results } = await stmt.all();
console.log(results)
```



绑定KV类似，不过在12步的加的信息有点不同。

类似于

```
[[kv_namespaces]]
binding = "MY_KV_NAMESPACE"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

如果你在里面直接编写代码，你会发现，额，没有终端和一些东西，写代码有点不方便。好吧，这个时候其实就要轮到我们前面第5步的那个**wrangler**出场了。我的理解是，大概可以是这么一个流程，在本地使用wrangler建立一个项目（项目类型和编程语言在创建时候可以选择），我们可以在这个wrangler创建的本地项目写代码。写完后，可以直接通过命令发布到自己的Workers，这样就可以实现在本地开发，可以同步啥的。与git有点类似的味道。

[wranger的官方链接](https://developers.cloudflare.com/workers/wrangler/)

至此，Worker初步设置结束

---

### Pages

下面是我们的Pages的设置。

这个比较简单，自己选择方式上传网页的文件即可，然后根据提示继续。

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/8f55ae73fd4848788dde68f7ef3705ed.png)

到了这里就行了

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/5085bcf1abf14fd28255ed91fff73173.png)



---

## 最后

如果您在Cloudflare中有受到保护的域名，完全可以设置这些东西的“自定义域”，使其访问更加友好。

本网站便是利用此方法（~~穷~~）

至此，结束，感谢
=======
<<<<<<< HEAD
# 使用cloudflare Workers和Pages搭建简单的个人博客

## 写在前面

首先这两个是啥，官方的话如下：

[Workers]([Cloudflare Workers](https://developers.cloudflare.com/workers/))

> Cloudflare Workers 提供了一个[无服务器](https://www.cloudflare.com/learning/serverless/what-is-serverless/)执行环境，允许您创建新应用程序或增强现有应用程序，而无需配置或维护基础设施。

[Pages]([Cloudflare Pages documentation](https://developers.cloudflare.com/pages/))

> 创建可立即部署到 Cloudflare 全球网络的全栈应用程序

我的理解是，Workers可以运行一些简单的脚本。Pages类似于可以提供一个完整的前后端（本网站只使用了作为托管静态网页的作用），当然功能不仅于此。

下面列出free计划中这两个的使用限制。

| Workers |      |
| ------ | ---- |
| 请求    | 10万 / 天 |
| 期间    | 期间不收费 |
| CPU时间 | 每次调用 10 毫秒的 CPU 时间 |

Pages的free计划有一些限制，这里不方便列表，请自行查看源文链接[使用限制 |Cloudflare Pages 文档](https://developers.cloudflare.com/pages/platform/limits/)





顺带一提，在cloudflare（以下简称cf）的free计划中，帮我们提供了免费的D1数据库和KV命名空间。

> [D1 SQL Database]([Cloudflare D1](https://developers.cloudflare.com/d1/)):	从 Workers 和 Pages 项目创建新的无服务器 SQL 数据库来进行查询。
>
> [KV]([Cloudflare Workers KV | Cloudflare Workers KV](https://developers.cloudflare.com/kv/)):	在 Cloudflare 网络中存储应用程序数据，并从 Workers 访问键值对。

下面列出free计划中这两个的使用限制。

| D1数据库             | workers Free |      | KV       | Free      |
| -------------------- | ------------ | ---- | :------- | --------- |
| 读取的行数           | 500万 / 天   |      | 读取请求 | 10万 / 天 |
| 写入的行数           | 10万 / 天    |      | 写入请求 | 1000 / 天 |
| 存储空间（每GB存储） | 5 GB (共)    |      | 删除请求 | 1000 / 天 |
|                      |              |      | 列出请求 | 1000 / 天 |
|                      |              |      | 存储数据 | 1GB       |

根据上面的条件，对于个人简单博客网站的搭建其实完全够了，每日的请求数量限制对于一般小型的也完全够用。Workers只提供一个脚本的运行，目前官方提供的有typescript与javasrcipt，还有python。对于本人的博客后端仅仅是查查数据库这种简单的完全可以实现。

**根据官方文档的介绍，在实际上利用[Pages]([Cloudflare Pages documentation](https://developers.cloudflare.com/pages/))与[Functions | Cloudflare Pages docs](https://developers.cloudflare.com/pages/functions/)实际上就可以完成地搭建一个人网站，在不使用额外服务器的条件上（为什么没有用而是选择了Workers而额外来个脚本呢，好吧，其实是最初没看到~~）。**



## 开始

于是，话不多说，我们准备开始。

需要的工具有：

- 一个cf账号
- 灵巧的手

### Workers

1. 登录Cloudflare,没有账号的请自行注册。然后选择**Free**计划即可，找不到图了。





2. 先创建Workers

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/c66aed16f5694beda6670524e8559cb9.png)





3. 如果方便的可以使用下面的模版，我这里就不使用了，后面需要什么再加

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/527b6330a9654a139d21c166aadee84a.png)





4. 随便命一个名字后点击部署即可

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/fa76abe1ea7646199ac0c262b249076c.png)





5. 之后应该是这个页面,可以点击其他的，不过我们暂时不管，点击“继续处理项目”就行

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/8b7a837c55484d63871764eb0e8ab86a.png)





6. 现在我们让为其添加数据库

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/1c7a99cbb8ba4672a7d0a92718e84cc6.png)





7. 随便输入名字后创建即可，只要与现存的不重名，点击创建

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/fd9ed9817872466aab4cb3920eac7415.png)





8. 成功后应该是这个页面

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/2ee7226b6c1049f08c999d13ec54c84e.png)





9. 现在我们去绑定这两个

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/bfe760b1a292409ca99dfa93aed2785a.png)





10. 如图

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/460feb438db94690bc20fbed7183b0bc.png)





11. 变量名称随便，然后选择你刚才的数据库后点击部署

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/f5de18573ab949c192b86841f9024095.png)





12. 然后去，右上角编辑代码

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/369c7e50358941b593c63d59d9a43b09.png)





13. 最后，index.js代码中那个函数体里面添加即可

```javascript
const { DATABASE } = env;

//执行方式大概像这样
const stmt = DATABASE.prepare("SELECT * FROM TABLENAME");
const { results } = await stmt.all();
console.log(results)
```



绑定KV类似，不过在12步的加的信息有点不同。

类似于

```
[[kv_namespaces]]
binding = "MY_KV_NAMESPACE"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

如果你在里面直接编写代码，你会发现，额，没有终端和一些东西，写代码有点不方便。好吧，这个时候其实就要轮到我们前面第5步的那个**wrangler**出场了。我的理解是，大概可以是这么一个流程，在本地使用wrangler建立一个项目（项目类型和编程语言在创建时候可以选择），我们可以在这个wrangler创建的本地项目写代码。写完后，可以直接通过命令发布到自己的Workers，这样就可以实现在本地开发，可以同步啥的。与git有点类似的味道。

[wranger的官方链接](https://developers.cloudflare.com/workers/wrangler/)

至此，Worker初步设置结束

---

### Pages

下面是我们的Pages的设置。

这个比较简单，自己选择方式上传网页的文件即可，然后根据提示继续。

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/8f55ae73fd4848788dde68f7ef3705ed.png)

到了这里就行了

![img](https://raw.githubusercontent.com/tbthub/cf-md/main/images/5085bcf1abf14fd28255ed91fff73173.png)



---

## 最后

如果您在Cloudflare中有受到保护的域名，完全可以设置这些东西的“自定义域”，使其访问更加友好。

本网站便是利用此方法（~~穷~~）

至此，结束，感谢
>>>>>>> 028546e4a9fe639a745614d5141dd67c460ec5c4
