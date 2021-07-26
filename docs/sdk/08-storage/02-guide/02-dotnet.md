---
id: dotnet
title: 数据存储指南 · .NET
sidebar_label: .NET 指南
---



数据存储是云服务提供的核心功能之一，可用于存放和查询应用数据。下面的代码展示了如何创建一个对象并将其存入云端：


```cs
// 构建对象
LCObject todo = new LCObject("Todo");
// 为属性赋值
todo["title"] = "工程师周会";
todo["content"] = "周二两点，全体成员";
// 将对象保存到云端
await todo.Save();
```

我们为各个平台或者语言开发的 SDK 在底层都是通过 HTTPS 协议调用统一的 REST API，提供完整的接口对数据进行各类操作。

## SDK 安装与初始化

请阅读 [数据存储、即时通讯 .NET SDK 配置指南](/sdk/storage/guide/setup-dotnet)。

> C#/.Net/Unity SDK 老版本迁移
>
> 如果你还在在使用我们老版本的 C#/.Net/Unity SDK（版本号为 `YYYYMMDD.N` 格式，类名以 `AV` 开头），请参考 [.Net SDK 迁移指南] 迁移到新版 SDK。

[.Net SDK 迁移指南]: https://github.com/leancloud/csharp-sdk/wiki/.Net-Standard-SDK-%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97

## 对象

### `LCObject`

`LCObject` 是云服务对复杂对象的封装，每个 `LCObject` 包含若干与 JSON 格式兼容的属性值对（也称键值对，key-value pairs）。这个数据是无模式化的（schema free），意味着你不需要提前标注每个 `LCObject` 上有哪些 key，你只需要随意设置键值对就可以，云端会保存它。

比如说，一个保存着单个 Todo 的 `LCObject` 可能包含如下数据：

```json
title:      "给小林发邮件确认会议时间",
isComplete: false,
priority:   2,
tags:       ["工作", "销售"]
```

### 数据类型

`LCObject` 支持的数据类型包括 `String`、`Number`、`Boolean`、`Object`、`Array`、`Date` 等等。你可以通过嵌套的方式在 `Object` 或 `Array` 里面存储更加结构化的数据。

`LCObject` 还支持两种特殊的数据类型 `Pointer` 和 `File`，可以分别用来存储指向其他 `LCObject` 的指针以及二进制数据。

`LCObject` 同时支持 `GeoPoint`，可以用来存储地理位置信息。参见 [GeoPoint](#geopoint)。

以下是一些示例：


```cs
// 基本类型
int numberValue = 2018;
bool boolValue = true;
string stringValue = "hello, world";
DateTime now = DateTime.Now;
List<int> intList = new List<int> { 1, 2, 3 };
Dictionary<string, object> dict = new Dictionary<string, object> {
  { "year", 1780 },
  { "first", "partridge" },
  { "second", "turtledoves" },
  { "fifth", "golden rings" }
};

// 构建对象
LCObject object = new LCObject("Hello");
object["numberValue"] = numberValue;
object["boolValue"] = boolValue;
object["stringValue"] = stringValue;
object["time"] = now;
object["intList"] = intList;
object["dictValue"] = dict;
```

我们不推荐通过 `byte[]` 在 `LCObject` 里面存储图片、文档等大型二进制数据。每个 `LCObject` 的大小不应超过 **128 KB**。如需存储大型文件，可创建 `LCFile` 实例并将将其关联到 `LCObject` 的某个属性上。参见 [文件](#文件)。

注意：时间类型在云端将会以 UTC 时间格式存储，但是客户端在读取之后会转化成本地时间。

**云服务控制台 > 数据存储 > 结构化数据** 中展示的日期数据也会依据操作系统的时区进行转换。一个例外是当你通过 REST API 获得数据时，这些数据将以 UTC 呈现。你可以手动对它们进行转换。

若想了解云服务是如何保护应用数据的，请阅读[数据和安全](/sdk/storage/guide/security)。

### 构建对象

下面的代码构建了一个 class 为 `Todo` 的 `LCObject`：

```cs
LCObject object = new LCObject("Todo");
```

在构建对象时，为了使云端知道对象属于哪个 class，需要将 class 的名字作为参数传入。你可以将云服务里面的 class 比作关系型数据库里面的表。一个 class 的名字必须以字母开头，且只能包含数字、字母和下划线。

### 保存对象

下面的代码将一个 class 为 `Todo` 的对象存入云端：


```cs
// 构建对象
LCObject todo = new LCObject("Todo");
// 为属性赋值
todo["title"] = "马拉松报名";
todo["priority"] = 2;
// 将对象保存到云端
await todo.Save();
```

为了确认对象已经保存成功，我们可以到 **云服务控制台 > 数据存储 > 结构化数据 > `Todo`** 里面看一下，应该会有一行新的数据产生。点一下这个数据的 `objectId`，应该能看到类似这样的内容：

```json
{
  "title":     "马拉松报名",
  "priority":  2,
  "ACL": {
    "*": {
      "read":  true,
      "write": true
    }
  },
  "objectId":  "582570f38ac247004f39c24b",
  "createdAt": "2017-11-11T07:19:15.549Z",
  "updatedAt": "2017-11-11T07:19:15.549Z"
}
```

注意，无需在 **云服务控制台 > 数据存储 > 结构化数据** 里面创建新的 `Todo` class 即可运行前面的代码。如果 class 不存在，它将自动创建。

以下是一些对象的内置属性，会在对象保存时自动创建，无需手动指定：

内置属性 | 类型 | 描述
--- | --- | ---
`objectId` | string | 该对象唯一的 ID 标识。
`ACL` | LCACL | 该对象的权限控制，实际上是一个 JSON 对象，控制台做了展现优化。
`createdAt` | DateTime | 该对象被创建的时间。
`updatedAt` | DateTime | 该对象最后一次被修改的时间。

这些属性的值会在对象被存入云端时自动填入，代码中尚未保存的 `LCObject` 不存在这些属性。

属性名（**keys**）只能包含字母、数字和下划线。自定义属性不得以双下划线（`__`）开头或与任何系统保留字段和内置属性（`ACL`、`className`、`createdAt`、`objectId` 和 `updatedAt`）重名，无论大小写。

属性值（**values**）可以是字符串、数字、布尔值、数组或字典（任何能以 JSON 编码的数据）。参见 [数据类型](#数据类型)。

我们推荐使用驼峰式命名法（CamelCase）为类和属性来取名。类，采用大驼峰法，如 `CustomData`。属性，采用小驼峰法，如 `imageUrl`。

### 获取对象

对于已经保存到云端的 `LCObject`，可以通过它的 `objectId` 将其取回：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
LCObject todo = await query.Get("582570f38ac247004f39c24b");
// todo 就是 ObjectId 为 582570f38ac247004f39c24b 的 Todo 实例
string title = todo["title"] as string;
int priority = (int)(todo["priority"]);

// 获取内置属性
string objectId = todo.ObjectId;
DateTime updatedAt = todo.UpdatedAt;
DateTime createdAt = todo.CreatedAt;
```

对象拿到之后，可以通过 `get` 方法来获取各个属性的值。注意 `objectId`、`updatedAt` 和 `createdAt` 这三个内置属性不能通过 `get` 获取或通过 `set` 修改，只能由云端自动进行填充。尚未保存的 `LCObject` 不存在这些属性。

如果你试图获取一个不存在的属性，SDK 不会报错，而是会返回 null。
#### 同步对象

当云端数据发生更改时，你可以调用 `Fetch` 方法来刷新对象，使之与云端数据同步：

```cs
LCObject todo = LCObject.CreateWithoutData("Todo", "582570f38ac247004f39c24b");
await todo.Fetch();
// todo 已刷新
```

刷新操作会强行使用云端的属性值覆盖本地的属性。因此如果本地有属性修改，**`Fetch` 操作会丢弃这些修改**。为避免这种情况，你可以在刷新时指定 **需要刷新的属性**，这样只有指定的属性会被刷新（包括内置属性 `objectId`、`createdAt` 和 `updatedAt`），其他属性不受影响。

```cs
LCObject todo = LCObject.CreateWithoutData("Todo", "582570f38ac247004f39c24b");
await todo.Fetch(includes: new string[] { "priority", "location" });
// 只有 priority 和 location 会被获取和刷新
```
### 更新对象

要更新一个对象，只需指定需要更新的属性名和属性值，然后调用 `Save` 方法。例如：

```cs
LCObject todo = LCObject.CreateWithoutData("Todo", "582570f38ac247004f39c24b");
todo["content"] = "这周周会改到周三下午三点。";
await todo.Save();
```

云服务会自动识别需要更新的属性并将对应的数据发往云端，未更新的属性会保持原样。

#### 有条件更新对象

通过传入 `query` 选项，可以按照指定条件去更新对象——当条件满足时，执行更新；条件不满足时，不执行更新并返回 `305` 错误。

例如，用户的账户表 `Account` 有一个余额字段 `balance`，同时有多个请求要修改该字段值。为避免余额出现负值，只有当金额小于或等于余额的时候才能接受请求：

```cs
try {
  LCObject account = LCObject.CreateWithoutData("Account", "5745557f71cfe40068c6abe0");
  // 对 balance 原子减少 100
  int amount = -100;
  account.Increment("balance", amount);
  // 设置条件
  LCQuery<LCObject> query = new LCQuery<LCObject>("Account");
  query.WhereGreaterThanOrEqualTo("balance", -amount);
  // 操作结束后，返回最新数据。
  // 如果是新对象，则所有属性都会被返回，
  // 否则只有更新的属性会被返回。
  await account.Save(fetchWhenSave: true, query: query);
  print($"当前余额为：{account["balance"]}");
} catch (LCException e) {
  if (e.code == 305) {
    print("余额不足，操作失败！");
  }
}
```

**`query` 选项只对已存在的对象有效**，不适用于尚未存入云端的对象。

`query` 选项在有多个客户端需要更新同一属性的时候非常有用。相比于通过 `LCQuery` 查询 `LCObject` 再对其进行更新的方法，这样做更加简洁，并且能够避免出现差错。

#### 更新计数器

设想我们正在开发一个微博，需要统计一条微博有多少个赞和多少次转发。由于赞和转发的操作可能由多个客户端同时进行，直接在本地更新数字并保存到云端的做法极有可能导致差错。为保证计数的准确性，可以通过 **原子操作** 来增加或减少一个属性内保存的数字：

```cs
post.Increment("likes", 1);
```

可以指定需要增加或减少的值。若未指定，则默认使用 `1`。

注意，虽然原子增减支持浮点数，但因为底层数据库的浮点数存储格式限制，会有舍入误差。
因此，需要原子增减的字段建议使用整数以避免误差，例如 `3.14` 可以存储为 `314`，然后在客户端进行相应的转换。
否则，以比较大小为条件查询对象的时候，需要特殊处理，
`< a` 需改查 `< a + e`，`> a` 需改查 `> a - e`，`== a` 需改查 `> a - e` 且 `< a + e`，其中 `e` 为误差范围，据所需精度取值，比如 `0.0001`。

#### 更新数组

更新数组也是原子操作。使用以下方法可以方便地维护数组类型的数据：

- `Add(key, value)` 将指定对象附加到数组末尾。
- `AddAll(key, values)` 将一组对象附加到数组末尾。
- `AddUnique(key, value)` 将指定对象附加到数组末尾，确保对象唯一。
- `AddAllUnique(key, values)` 将指定对象数组附加到数组末尾，确保对象唯一。
- `Remove(key, value)` 从数组字段中删除指定对象的所有实例。
- `RemoveAll(key, values)` 从数组字段中删除指定的对象数组。


例如，`Todo` 用一个 `alarms` 属性保存所有闹钟的时间。下面的代码将多个时间加入这个属性：

```cs
DateTime alarm1 = DateTime.Parse("2018-04-30 07:10:00Z");
DateTime alarm2 = DateTime.Parse("2018-04-30 07:20:00Z");
DateTime alarm3 = DateTime.Parse("2018-04-30 07:30:00Z");

LCObject todo = new LCObject("Todo");
todo.AddAllUnique("alarms", new object[] { alarm1, alarm2, alarm3 });
await todo.Save();
```

### 删除对象

下面的代码从云端删除一个 `Todo` 对象；

```cs
LCObject todo = LCObject.CreateWithoutData("Todo", "582570f38ac247004f39c24b");
await todo.Delete();
```

如果只需删除对象的一个属性，可以用 `Unset`：

```cs
LCObject todo = LCObject.CreateWithoutData("Todo", "582570f38ac247004f39c24b");

// priority 属性会被删除
todo.Unset("priority");

// 保存对象
await todo.Save();
```

注意，删除对象是一个较为敏感的操作，我们建议你阅读[《ACL 权限管理开发指南》](https://leancloud.cn/docs/acl-guide.html) 来了解潜在的风险。熟悉 class 级别、对象级别和字段级别的权限可以帮助你有效阻止未经授权的操作。

### 批量操作

```cs
// 批量构建和更新
LCObject.SaveAll();

// 批量删除
LCObject.DeleteAll();

// 批量同步
LCObject.FetchAll();
```

下面的代码将所有 `Todo` 的 `isComplete` 设为 `true`：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
ReadOnlyCollection<LCObject> results = await query.Find();
// 获取需要更新的 todo
foreach (LCObject todo in results) {
  // 更新属性值
  todo["isComplete"] = true;
}
await LCObject.SaveAll(results);
```

虽然上述方法可以在一次请求中包含多个操作，每一个分别的保存或同步操作在计费时依然会被算作一次请求，而所有的删除操作则会被合并为一次请求。
### 后台运行

细心的开发者已经发现，在所有的示例代码中几乎都是用了异步来访问云端，形如的用法都是提供给开发者在主线程调用用以实现后台运行的方法，因此开发者在主线程可以放心地调用这种命名方式的函数。

### 数据模型

对象之间可以产生关联。拿一个博客应用来说，一个 `Post` 对象可以与许多个 `Comment` 对象产生关联。云服务支持三种关系：一对一、一对多、多对多。

#### 一对一、一对多关系

一对一、一对多关系可以通过将 `LCObject` 保存为另一个对象的属性值的方式产生。比如说，让博客应用中的一个 `Comment` 指向一个 `Post`。

下面的代码会创建一个含有单个 `Comment` 的 `Post`：

```cs
// 创建 post
LCObject post = new LCObject("Post");
post["title"] = "饿了……";
post["content"] = "中午去哪吃呢 ?";

// 创建 comment
LCObject comment = new LCObject("Comment");
comment["content"] = "当然是肯德基啦 !";

// 将 post 设为 comment 的一个属性值
comment["parent"] = post;

// 保存 comment 会同时保存 post
await comment.Save();
```

云端存储时，会将被指向的对象用 `Pointer` 的形式存起来。你也可以用 `objectId` 来指向一个对象：

```cs
LCObject post = LCObject.CreateWithoutData("Post", "57328ca079bc44005c2472d0");
comment["post"] = post;
```

请参阅 [关系查询](#关系查询) 来了解如何获取关联的对象。

#### 多对多关系

想要建立多对多关系，最简单的办法就是使用 **数组**。在大多数情况下，使用数组可以有效减少查询的次数，提升程序的运行效率。但如果有额外的属性需要附着于两个 class 之间的关联，那么使用 **中间表** 可能是更好的方式。注意这里说到的额外的属性是用来描述 class 之间的关系的，而不是任何单一的 class 的。

我们建议你在任何一个 class 的对象数量超出 100 的时候考虑使用中间表。

### 序列化和反序列化

在实际的开发中，把 `LCObject` 当作参数传递的时候，会涉及到复杂对象的拷贝的问题，因此 `LCObject` 也提供了序列化和反序列化的方法。

序列化：

```cs
LCObject object = new LCObject("Hello");
object["title"] = "马拉松报名";
object["priority"] = 2;
object["owner"] = await LCUser.GetCurrent();
string serializedString = object.ToString();
```

反序列化：

```cs
LCObject newObject = LCObject.ParseObject(json);
await newObject.Save();
```

## 查询

我们已经了解到如何从云端获取单个 `LCObject`，但你可能还会有一次性获取多个符合特定条件的 `LCObject` 的需求，这时候就需要用到 `LCQuery` 了。

### 基础查询

执行一次基础查询通常包括这些步骤：

1. 构建 `LCQuery`；
2. 向其添加查询条件；
3. 执行查询并获取包含满足条件的对象的数组。

下面的代码获取所有 `lastName` 为 `Smith` 的 `Student`：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Student");
query.WhereEqualTo("lastName", "Smith");
// students 是包含满足条件的 Student 对象的数组
ReadOnlyCollection<LCObject> students = await query.Find();
```
### 查询条件

可以给 `LCObject` 添加不同的条件来改变获取到的结果。

下面的代码查询所有 `firstName` 不为 `Jack` 的对象：


```cs
query.WhereNotEqualTo("firstName", "Jack");
```

对于能够排序的属性（比如数字、字符串），可以进行比较查询：


```cs
// 限制 age < 18
query.WhereLessThan("age", 18);

// 限制 age <= 18
query.WhereLessThanOrEqualTo("age", 18);

// 限制 age > 18
query.WhereGreaterThan("age", 18);

// 限制 age >= 18
query.WhereGreaterThanOrEqualTo("age", 18);
```

可以在同一个查询中设置多个条件，这样可以获取满足所有条件的结果。可以理解为所有的条件是 `AND` 的关系：


```cs
query.WhereEqualTo("firstName", "Jack");
query.WhereGreaterThan("age", 18);
```


可以通过指定 `limit` 限制返回结果的数量（默认为 `100`）：


```cs
// 最多获取 10 条结果
query.Limit(10);
```


由于性能原因，`limit` 最大只能设为 `1000`。即使将其设为大于 `1000` 的数，云端也只会返回 1,000 条结果。

如果只需要一条结果，可以直接用 `First`：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
query.WhereEqualTo("priority", 2);
// todo 是第一个满足条件的 Todo 对象
LCObject todo = await query.First();
```

可以通过设置 `skip` 来跳过一定数量的结果：

```cs
query.Skip(20);
```

把 `skip` 和 `limit` 结合起来，就能实现翻页功能：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
query.WhereEqualTo("priority", 2);
query.Limit(10);
query.Skip(20);
```

需要注意的是，`skip` 的值越高，查询所需的时间就越长。作为替代方案，可以通过设置 `createdAt` 或 `updatedAt` 的范围来实现更高效的翻页，因为它们都自带索引。
同理，也可以通过设置自增字段的范围来实现翻页。

对于能够排序的属性，可以指定结果的排序规则：

```cs
// 按 createdAt 升序排列
query.OrderByAscending("createdAt");

// 按 createdAt 降序排列
query.OrderByDescending("createdAt");
```

还可以为同一个查询添加多个排序规则；

```cs
query.AddAscendingOrder("priority");
query.AddDescendingOrder("createdAt");
```

下面的代码可用于查找包含或不包含某一属性的对象：


可以通过 `Select` 指定需要返回的属性。下面的代码只获取每个对象的 `title` 和 `content`（包括内置属性 `objectId`、`createdAt` 和 `updatedAt`）：


```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
query.Select("title");
query.Select("content");
LCObject todo = await query.First();

string title = todo["title"] as string; // √
string content = todo["content"] as string; // √
string notes = todo["notes"] as string; // null
```

`Select` 支持点号（`author.firstName`），详见[《点号使用指南》](https://leancloud.cn/docs/dot-notation.html)。

另外，字段名前添加减号前缀表示反向选择，例如 `-author` 表示不返回 `author` 字段。
反向选择同样适用于内置字段，比如 `-objectId`，也可以和点号组合使用，比如 `-pubUser.createdAt`。

对于未获取的属性，可以通过对结果中的对象进行 `Fetch` 操作来获取。参见 [同步对象](#同步对象)。
### 字符串查询

可以用 `WhereStartsWith` 来查找某一属性值以特定字符串开头的对象。和 SQL 中的 `LIKE` 一样，你可以利用索引带来的优势：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
// 相当于 SQL 中的 title LIKE 'lunch%'
query.WhereStartsWith("title", "lunch");
```

可以用 `WhereContains` 来查找某一属性值包含特定字符串的对象：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
// 相当于 SQL 中的 title LIKE '%lunch%'
query.WhereContains("title", "lunch");
```

和 `WhereStartsWith` 不同，`WhereContains` 无法利用索引，因此不建议用于大型数据集。

注意 `WhereStartsWith` 和 `WhereContains` 都是 **区分大小写** 的，所以上述查询会忽略 `Lunch`、`LUNCH` 等字符串。

如果想查找某一属性值不包含特定字符串的对象，可以使用 `WhereMatches` 进行基于正则表达式的查询：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
// 'title' 不包含 "ticket"（不区分大小写）
query.WhereMatches("title", "^((?!ticket).)*\$", modifiers: "i");
```

不过我们并不推荐大量使用这类查询，尤其是对于包含超过 100,000 个对象的 class，因为这类查询无法利用索引，实际操作中云端会遍历所有对象来获取结果。如果有进行全文搜索的需求，可以使用全文搜索服务。

使用查询时如果遇到性能问题，可参阅 [查询性能优化](#查询性能优化)。

### 数组查询

下面的代码查找所有数组属性 `tags` 包含 ` 工作 ` 的对象：

```cs
query.WhereEqualTo("tags", "工作");
```


下面的代码查询数组属性长度为 3 （正好包含 3 个标签）的对象：


```cs
query.WhereSizeEqualTo("tags", 3);
```


下面的代码查找所有数组属性 `tags` **同时包含** ` 工作 `、` 销售 ` 和 ` 会议 ` 的对象：


```cs
query.WhereContainsAll("tags", new string[] { "工作", "销售", "会议" });
```


如需获取某一属性值包含一列值中任意一个值的对象，可以直接用 `WhereContainedIn` 而无需执行多次查询。下面的代码构建的查询会查找所有 `priority` 为 `1` **或** `2` 的 todo 对象：


```cs
// 单个查询
LCQuery<LCObject> priorityOneOrTwo = new LCQuery<LCObject>("Todo");
priorityOneOrTwo.WhereContainedIn("priority", new string[] { 1, 2 });
// 这样就可以了 :)

// ---------------
//       vs.
// ---------------

// 多个查询
LCQuery<LCObject> priorityOne = new LCQuery<LCObject>("Todo");
priorityOne.WhereEqualTo("priority", 1);

LCQuery<LCObject> priorityTwo = new LCQuery<LCObject>("Todo");
priorityTwo.WhereEqualTo("priority", 2);

LCQuery<LCObject> priorityOneOrTwo = LCQuery.Or(new LCQuery<LCObject>[] { priorityOne, priorityTwo });
ReadOnlyCollection<LCObject> results = await priorityOneOrTwo.Find();
// 好像有些繁琐 :(
```


反过来，还可以用 `WhereNotContainedIn` 来获取某一属性值不包含一列值中任何一个的对象。

### 关系查询

查询关联数据有很多种方式，常见的一种是查询某一属性值为特定 `LCObject` 的对象，这时可以像其他查询一样直接用 `WhereEqualTo`。比如说，如果每一条博客评论 `Comment` 都有一个 `post` 属性用来存放原文 `Post`，则可以用下面的方法获取所有与某一 `Post` 相关联的评论：

```cs
LCObject post = LCObject.CreateWithoutData("Post", "57328ca079bc44005c2472d0");
LCQuery<LCObject> query = new LCQuery<LCObject>("Comment");
query.WhereEqualTo("post", post);
// comments 包含与 post 相关联的评论
ReadOnlyCollection<LCObject> comments = await query.Find();
```

如需获取某一属性值为另一查询结果中任一 `LCObject` 的对象，可以用 `WhereMatchesQuery`。下面的代码构建的查询可以找到所有包含图片的博客文章的评论：

```cs
LCQuery<LCObject> innerQuery = new LCQuery<LCObject>("Post");
innerQuery.WhereExists("image");

LCQuery<LCObject> query = new LCQuery<LCObject>("Comment");
query.WhereMatchesQuery("post", innerQuery);
```

如需获取某一属性值不是另一查询结果中任一 `LCObject` 的对象，则使用 `WhereDoesNotMatchQuery`。

有时候可能需要获取来自另一个 class 的数据而不想进行额外的查询，此时可以在同一个查询上使用 `Include`。下面的代码查找最新发布的 10 条评论，并包含各自对应的博客文章：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Comment");

// 获取最新发布的
query.OrderByDescending("createdAt");

// 只获取 10 条
query.Limit(10);

// 同时包含博客文章
query.Include("post");

// comments 包含最新发布的 10 条评论，包含各自对应的博客文章
ReadOnlyCollection<LCObject> comments = await query.Find();
foreach (LCObject comment in comments) {
// 该操作无需网络连接
  LCObject post = comment["post"] as LCObject;
}
```


可以用 dot 符号（`.`）来获取多级关系，例如 `post.author`，详见[《点号使用指南》](https://leancloud.cn/docs/dot-notation.html)的《在查询对象时使用点号》一节。

可以在同一查询上应用多次 `Include` 以包含多个属性。

通过 `Include` 进行多级查询的方式不适用于数组属性内部的 `LCObject`，只能包含到数组本身。

#### 关系查询的注意事项

云端使用的并非关系型数据库，无法做到真正的联表查询，所以实际的处理方式是：先执行内嵌 / 子查询（和普通查询一样，`limit` 默认为 `100`，最大 `1000`），然后将子查询的结果填入主查询的对应位置，再执行主查询。如果子查询匹配到的记录数量超出 `limit`，且主查询有其他查询条件，那么可能会出现没有结果或结果不全的情况，因为只有 `limit` 数量以内的结果会被填入主查询。

我们建议采用以下方案进行改进：

- 确保子查询的结果在 100 条以下，如果在 100 至 1,000 条之间的话请将子查询的 `limit` 设为 `1000`。
- 将需要查询的字段冗余到主查询所在的表上。
- 进行多次查询，每次在子查询上设置不同的 `skip` 值来遍历所有记录（注意 `skip` 的值较大时可能会引发性能问题，因此不是很推荐）。

### 统计总数量

如果只需知道有多少对象匹配查询条件而无需获取对象本身，可使用 `Count` 来代替 `Find`。比如说，查询有多少个已完成的 todo：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
query.WhereEqualTo("isComplete", true);
int count = await query.Count();
print($"{count} 个 todo 已完成");
```

### 组合查询

组合查询就是把诸多查询条件用一定逻辑合并到一起（`OR` 或 `AND`）再交给云端去查询。

组合查询不支持在子查询中包含 `GeoPoint` 或其他非过滤性的限制（例如 `near`、`withinGeoBox`、`limit`、`skip`、`ascending`、`descending`、`include`）。

#### OR 查询

OR 操作表示多个查询条件符合其中任意一个即可。 例如，查询优先级大于等于 `3` 或者已经完成了的 todo：


```cs
LCQuery<LCObject> priorityQuery = new LCQuery<LCObject>("Todo");
priorityQuery.WhereGreaterThanOrEqualTo("priority", 3);

LCQuery<LCObject> isCompleteQuery = new LCQuery<LCObject>("Todo");
isCompleteQuery.WhereEqualTo("isComplete", true);

LCQuery<LCObject> priorityOneOrTwo = LCQuery.Or(new LCQuery<LCObject>[] { priorityQuery, isCompleteQuery });
ReadOnlyCollection<LCObject> results = await priorityOneOrTwo.Find();
```

使用 OR 查询时，子查询中不能包含 `GeoPoint` 相关的查询。

#### AND 查询

使用 AND 查询的效果等同于往 `LCQuery` 添加多个条件。下面的代码构建的查询会查找创建时间在 `2016-11-13` 和 `2016-12-02` 之间的 todo：

```cs
LCQuery<LCObject> startDateQuery = new LCQuery<LCObject>("Todo");
startDateQuery.WhereGreaterThanOrEqualTo("createdAt", DateTime.Parse("2016-11-13 00:00:00Z"));

LCQuery<LCObject> endDateQuery = new LCQuery<LCObject>("Todo");
endDateQuery.WhereLessThan("createdAt", DateTime.Parse("2016-12-03 00:00:00Z"));

LCQuery<LCObject> query = LCQuery<LCObject>.And(new LCQuery<LCObject>[] { startDateQuery, endDateQuery });
ReadOnlyCollection<LCObject> results = await query.Find();
```

单独使用 AND 查询跟使用基础查询相比并没有什么不同，不过当查询条件中包含不止一个 OR 查询时，就必须使用 AND 查询：


```cs
LCQuery<LCObject> createdAtQuery = new LCQuery<LCObject>("Todo");
createdAtQuery.WhereGreaterThanOrEqualTo("createdAt", DateTime.Parse("2018-04-30 00:00:00Z"));
createdAtQuery.WhereLessThan("createdAt", DateTime.Parse("2018-05-01 00:00:00Z"));

LCQuery<LCObject> locationQuery = new LCQuery<LCObject>("Todo");
locationQuery.WhereDoesNotExist("location");

LCQuery<LCObject> priority2Query = new LCQuery<LCObject>("Todo");
priorityQuery.WhereEqualTo("priority", 2);

LCQuery<LCObject> priority3Query = new LCQuery<LCObject>("Todo");
priorityQuery.WhereEqualTo("priority", 3);

LCQuery<LCObject> priorityQuery = LCQuery<LCObject>.Or(new LCQuery<LCObject>[] { priority2Query, priority3Query });
LCQuery<LCObject> timeLocationQuery = LCQuery<LCObject>.Or(new LCQuery<LCObject>[] { locationQuery, createdAtQuery });
LCQuery<LCObject> query = LCQuery<LCObject>.And(new LCQuery<LCObject>[] { priorityQuery, timeLocationQuery });
```

### 查询性能优化

影响查询性能的因素很多。特别是当查询结果的数量超过 10 万，查询性能可能会显著下降或出现瓶颈。以下列举一些容易降低性能的查询方式，开发者可以据此进行有针对性的调整和优化，或尽量避免使用。

- 不等于和不包含查询（无法使用索引）
- 通配符在前面的字符串查询（无法使用索引）
- 有条件的 `count`（需要扫描所有数据）
- `skip` 跳过较多的行数（相当于需要先查出被跳过的那些行）
- 无索引的排序（另外除非复合索引同时覆盖了查询和排序，否则只有其中一个能使用索引）
- 无索引的查询（另外除非复合索引同时覆盖了所有条件，否则未覆盖到的条件无法使用索引，如果未覆盖的条件区分度较低将会扫描较多的数据）

## LiveQuery

LiveQuery 衍生于 [`LCQuery`](#查询)，并为其带来了更强大的功能。它可以让你无需编写复杂的逻辑便可在客户端之间同步数据，这对于有实时数据同步需求的应用来说很有帮助。

设想你正在开发一个多人协作同时编辑一份文档的应用，单纯地使用 `LCQuery` 并不是最好的做法，因为它只具备主动拉取的功能，而应用并不知道什么时候该去拉取。

想要解决这个问题，就要用到 LiveQuery 了。借助 LiveQuery，你可以订阅所有需要保持同步的 `LCQuery`。订阅成功后，一旦有符合 `LCQuery` 的 `LCObject` 发生变化，云端就会主动、实时地将信息通知到客户端。

LiveQuery 使用 WebSocket 在客户端和云端之间建立连接。WebSocket 的处理会比较复杂，而我们将其封装成了一个简单的 API 供你直接使用，无需关注背后的原理。

### 启用 LiveQuery

进入 **云服务控制台 > 数据存储 > 设置**，在 **安全设置** 里面勾选 **启用 LiveQuery**

```cs
using LeanCloud.LiveQuery;
```

### Demo

下面是在使用了 LiveQuery 的网页应用和手机应用中分别操作，数据保持同步的效果：

<video src="https://capacity-files.lncld.net/1496988080458" controls autoplay muted preload="auto" width="100%" height="100%" >
HTML5 Video is required for this demo, which your browser doesn't support.
</video>

使用我们的「LeanTodo」微信小程序和网页应用，可以实际体验以上视频所演示的效果，步骤如下：

1. 微信扫码，添加小程序「LeanTodo」；

    ![LeanTodo mini program](/img/leantodo-weapp-qr.jpg)

2. 进入小程序，点击首页左下角 **设置** > **账户设置**，输入便于记忆的用户名和密码；

3. 使用浏览器访问 <https://leancloud.github.io/leantodo-vue/>，输入刚刚在小程序中更新好的账户信息，点击 **Login**；

4. 随意添加更改数据，查看两端的同步状态。

注意按以上顺序操作。在网页应用中使用 **Signup** 注册的账户无法与小程序创建的账户相关联，所以如果颠倒以上操作顺序，则无法观测到数据同步效果。

[LiveQuery 公开课](http://www.bilibili.com/video/av11291992/) 涵盖了许多开发者关心的问题和解答。

### 构建订阅

首先创建一个普通的 `LCQuery` 对象，添加查询条件（如有），然后进行订阅操作：


```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
query.WhereEqualTo("isComplete", true);
await query.Subscribe();
// 订阅成功
```


LiveQuery 不支持内嵌查询，也不支持返回指定属性。

订阅成功后，就可以接收到和 `LCObject` 相关的更新了。假如在另一个客户端上创建了一个 `Todo` 对象，对象的 `title` 设为 ` 更新作品集 `，那么下面的代码可以获取到这个新的 `Todo`：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
LCLiveQuery liveQuery = await query.Subscribe();
liveQuery.OnCreate = (obj) => {
  print(obj["title"]); // 更新作品集
};
```


此时如果有人把 `Todo` 的 `content` 改为 ` 把我最近画的插画放上去 `，那么下面的代码可以获取到本次更新：

```cs
liveQuery.OnUpdate = (updatedTodo, updatedKeys) => {
  print(updatedTodo["content"]); // 把我最近画的插画放上去
};
```

### 事件处理

订阅成功后，可以选择监听如下几种数据变化：

- `create`
- `update`
- `enter`
- `leave`
- `delete`

#### `create` 事件

当有新的满足 `LCQuery` 查询条件的 `LCObject` 被创建时，`create` 事件会被触发。下面的 `object` 就是新建的 `LCObject`：

```cs
liveQuery.OnCreate = (obj) => {
  print("对象被创建。");
};
```

#### `update` 事件

当有满足 `LCQuery` 查询条件的 `LCObject` 被更新时，`update` 事件会被触发。下面的 `object` 就是有更新的 `LCObject`：

```cs
liveQuery.OnUpdate = (obj, updatedKeys) => {
  print("对象被更新。");
};
```

#### `enter` 事件

当一个已存在的、原本不符合 `LCQuery` 查询条件的 `LCObject` 发生更新，且更新后符合查询条件，`enter` 事件会被触发。下面的 `object` 就是进入 `LCQuery` 的 `LCObject`，其内容为该对象最新的值：

```cs
liveQuery.OnEnter = (obj, updatedKeys) => {
  print("对象进入。");
};
```

注意区分 `create` 和 `enter` 的不同行为。如果一个对象已经存在，在更新之前不符合查询条件，而在更新之后符合查询条件，那么 `enter` 事件会被触发。如果一个对象原本不存在，后来被构建了出来，那么 `create` 事件会被触发。

#### `leave` 事件

当一个已存在的、原本符合 `LCQuery` 查询条件的 `LCObject` 发生更新，且更新后不符合查询条件，`leave` 事件会被触发。下面的 `object` 就是离开 `LCQuery` 的 `LCObject`，其内容为该对象最新的值：

```cs
liveQuery.OnLeave = (obj, updatedKeys) => {
  print("对象离开。");
};
```

#### `delete` 事件

当一个已存在的、原本符合 `LCQuery` 查询条件的 `LCObject` 被删除，`delete` 事件会被触发。下面的 `object` 就是被删除的 `LCObject` 的 `objectId`：

```cs
liveQuery.OnDelete = (objId) => {
  print("对象被删除。");
};
```

### 取消订阅

如果不再需要接收有关 `LCQuery` 的更新，可以取消订阅。

```cs
await liveQuery.Unsubscribe();
// 成功取消订阅
```

### 断开连接

断开连接有几种情况：

1. 网络异常或者网络切换，非预期性断开。
2. 退出应用、关机或者打开飞行模式等，用户在应用外的操作导致断开。

如上几种情况开发者无需做额外的操作，只要切回应用，SDK 会自动重新订阅，数据变更会继续推送到客户端。

而另外一种极端情况——**当用户在移动端使用手机的进程管理工具，杀死了进程或者直接关闭了网页的情况下**，SDK 无法自动重新订阅，此时需要开发者根据实际情况实现重新订阅。

### LiveQuery 的注意事项

因为 LiveQuery 的实时性，很多用户会陷入一个误区，试着用 LiveQuery 来实现一个简单的聊天功能。
我们不建议这样做，因为使用 LiveQuery 构建聊天服务会承担额外的存储成本，产生的费用会增加，后期维护的难度非常大（聊天记录、对话维护之类的代码会很混乱），并且云服务已经提供了即时通讯的服务。
LiveQuery 的核心还是提供一个针对查询的推拉结合的用法，脱离设计初衷容易造成前端的模块混乱。

## 文件

有时候应用需要存储尺寸较大或结构较为复杂的数据，这类数据不适合用 `LCObject` 保存，此时文件对象 `LCFile` 便成为了更好的选择。文件对象最常见的用途是保存图片，不过也可以用来保存文档、视频、音乐等其他二进制数据。

### 构建文件

可以通过字符串构建文件：
```cs
LCFile file = new LCFile("resume.txt", UTF8.GetBytes("LeanCloud"));
```

除此之外，还可以通过 URL 构建文件：

```cs
LCFile file = new LCFile("logo.png", new Uri("https://leancloud.cn/assets/imgs/press/Logo%20-%20Blue%20Padding.a60eb2fa.png"));
```

通过 URL 构建文件时，SDK 并不会将原本的文件转储到云端，而是会将文件的物理地址存储为字符串，这样也就不会产生任何文件上传流量。使用其他方式构建的文件会被保存在云端。

云端会根据文件扩展名自动检测文件类型。如果需要的话，也可以手动指定 `Content-Type`（一般称为 MIME 类型）：

```cs
LCFile file = new LCFile("resume.txt", UTF8.GetBytes("LeanCloud"));
file.MimeType = "application/json";
```

与前面提到的方式相比，一个更常见的文件构建方式是从本地路径上传。

```cs
LCFile file = new LCFile("avatar.jpg", "./avatar.jpg");
```

这里上传的文件名字叫做 `avatar.jpg`。需要注意：

- 每个文件会被分配到一个独一无二的 `objectId`，所以在一个应用内是允许多个文件重名的。
- 文件必须有扩展名才能被云端正确地识别出类型。比如说要用 `LCFile` 保存一个 PNG 格式的图像，那么扩展名应为 `.png`。
- 如果文件没有扩展名，且没有手动指定类型，那么云服务将默认使用 `application/octet-stream`。

### 保存文件

将文件保存到云端后，便可获得一个永久指向该文件的 URL：

```cs
await file.Save();
print(file.Url);
```

文件上传后，可以在 `_File` class 中找到。已上传的文件无法再被修改。如果需要修改文件，只能重新上传修改过的文件并取得新的 `objectId` 和 URL。


已经保存到云端的文件可以关联到 `LCObject`：

```cs
LCObject todo = new LCObject("Todo");
todo["title"] = "买蛋糕";
// attachments 是一个 File 类型
todo.Add("attachments", file);
await todo.Save();
```

也可以通过构建 `LCQuery` 进行[查询](#查询)：

```cs
LCQuery<LCObject> query = LCFile.GetQuery();
```

需要注意的是，内部文件（上传到文件服务的文件）的 `url` 字段是由云端动态生成的，其中涉及切换自定义域名的相关处理逻辑。
因此，通过 url 字段查询文件仅适用于外部文件（直接保存外部 URL 到 `_File` 表创建的文件），内部文件请改用 key 字段（URL 中的路径）查询。

注意，如果文件被保存到了 `LCObject` 的一个数组属性中，那么在查询 `LCObject` 时如果需要包含文件，则要用到 `LCQuery` 的 `Include` 方法。比如说，在获取所有标题为 ` 买蛋糕 ` 的 todo 的同时获取附件中的文件：

```cs
// 获取同一标题且包含附件的 todo
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
query.WhereEqualTo("title", "买蛋糕");
query.WhereExists("attachments");

// 同时获取附件中的文件
query.Include("attachments");
ReadOnlyCollection<LCObject> todos = await query.Find();
foreach (LCObject todo in todos) {
  // 获取每个 todo 的 attachments 数组
  List<object> attachments = todo["attachments"] as List<object>;
}
```

### 上传进度监听

上传过程中可以实时向用户展示进度：

```cs
await file.Save((count, total) => {
  print($"{count}/{total}");
  if (count == total) {
    print("done");
  }
});
```

### 文件元数据

上传文件时，可以用 `metaData` 添加额外的属性。文件一旦保存，`metaData` 便不可再修改。

```cs
file.AddMetaData("size", 1024);
file.AddMetaData("width", 128);
file.AddMetaData("height", 256);
file.MimeType = "image/jpg";
await file.Save();
```

### 图像缩略图

成功保存图像后，除了可以获取指向该文件的 URL 外，还可以获取图像的缩略图 URL，并且可以指定缩略图的宽度和高度：

```cs
// 获得宽度为 100 像素，高度为 200 像素的缩略图 url
string url = file.GetThumbnailUrl(100, 200);
```

图片最大不超过 **20 MB** 才可以获取缩略图。

国际版不支持图片缩略图。

### 删除文件

下面的代码从云端删除一个文件：

```cs
LCFile file = LCObject.CreateWithoutData("_File", "552e0a27e4b0643b709e891e");
await file.Delete();
```

默认情况下，文件的删除权限是关闭的，需要进入 **云服务控制台 > 数据存储 > 结构化数据 > `_File`**，选择 **其他** > **权限设置** > **`delete`** 来开启。

## GeoPoint

云服务允许你通过将 `LCGeoPoint` 关联到 `LCObject` 的方式存储折射真实世界地理位置的经纬坐标，这样做可以让你查询包含一个点附近的坐标的对象。常见的使用场景有「查找附近的用户」和「查找附近的地点」。

要构建一个包含地理位置的对象，首先要构建一个地理位置。下面的代码构建了一个 `{{geoPointObjectName}}` 并将其纬度（`latitude`）设为 `39.9`，经度（`longitude`）设为 `116.4`：

```cs
LCGeoPoint point = new LCGeoPoint(39.9, 116.4);
```

现在可以将这个地理位置存储为一个对象的属性：

```cs
todo["location"] = point;
```

### 地理位置查询

给定一些含有地理位置的对象，可以从中找出离某一点最近的几个，或者处于某一范围内的几个。要执行这样的查询，可以向普通的 `LCQuery` 添加 `WhereNear` 条件。下面的代码查找 `location` 属性值离某一点最近的 `Todo` 对象：

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
LCGeoPoint point = new LCGeoPoint(39.9, 116.4);
query.WhereNear("location", point);

// 限制为 10 条结果
query.Limit(10);
// todos 是包含满足条件的 Todo 对象的数组
ReadOnlyCollection<LCObject> todos = await query.Find();
```

像 `OrderByAscending` 和 `OrderByDescending` 这样额外的排序条件会获得比默认的距离排序更高的优先级。

若要限制结果和给定地点之间的距离，可以参考 API 文档中的  参数。

若要查询在某一矩形范围内的对象，可以用 `WhereWithinGeoBox`：

![withinGeoBox](/img/geopoint-withingeobox.svg)

```cs
LCQuery<LCObject> query = new LCQuery<LCObject>("Todo");
LCGeoPoint southwest = new LCGeoPoint(30, 115);
LCGeoPoint northeast = new LCGeoPoint(40, 118);
query.WhereWithinGeoBox("location", southwest, northeast);
```

### GeoPoint 的注意事项

GeoPoint 的经纬度的类型是数字，且经度需在 -180.0 到 180.0 之间，纬度需在 -90.0 到 90.0 之间。
另外，每个对象最多只能有一个类型为 GeoPoint 的属性。

## 用户

我们在[登录功能的开发指南](/sdk/taptap-login/guide/start) 中简单介绍了内建账户系统（TDSUser）的基本用法。
这里再详细介绍 TDSUser 的用法。

### TDSUser 和 LCUser

TDSUser 类继承自 LCUser 类。
LCUser 是 LeanCloud 提供的账户系统，TDSUser 基本沿用了其功能和接口，并针对 TDS 的需求进行了细微调整，所以我们推荐大家使用 TDSUser 类来构建玩家账户系统。

### 当前用户

用户登录后，SDK 会自动将会话信息存储到客户端，这样用户在下次打开客户端时无需再次登录。下面的代码检查是否有已经登录的用户：

```cs
TDSUser currentUser = await TDSUser.GetCurrent();
if (currentUser != null) {
  // 跳到首页
} else {
  // 显示注册或登录页面
}
```

会话信息会长期有效，直到用户主动登出：

```cs
await TDSUser.Logout();

// currentUser 变为 null
TDSUser currentUser = await TDSUser.GetCurrent();
```

### 设置当前用户

用户登录后，云端会返回一个 **session token** 给客户端，它会由 SDK 缓存起来并用于日后同一 `TDSUser` 的鉴权请求。session token 会被包含在每个客户端发起的 HTTP 请求的 header 里面，这样云端就知道是哪个 `TDSUser` 发起的请求了。

以下是一些应用可能需要用到 session token 的场景：

- 应用根据以前缓存的 session token 登录（可以通过 `SessionToken` 属性获取到当前用户的 session token，在服务端等受信任的环境下，可以通过 `Master Key` （即 `Server Secret`）读取任意用户的 `sessionToken` 字段以获取 session token）。
- 应用内的某个 WebView 需要知道当前登录的用户。
- 在服务端登录后，返回 session token 给客户端，客户端根据返回的 session token 登录。

下面的代码使用 session token 登录一个用户（云端会验证 session token 是否有效）：

```cs
await TDSUser.BecomeWithSessionToken("anmlwi96s381m6ca7o7266pzf");
```

请避免在外部浏览器使用 URL 来传递 session token，以防范信息泄露风险。

如果在 **控制台 > 数据存储 > 设置** 中勾选了 **密码修改后，强制客户端重新登录**，那么当一个用户修改密码后，该用户的 session token 会被重置。此时需要让用户重新登录，否则会遇到 `403 (Forbidden)` 错误。

下面的代码检查 session token 是否有效：

```cs
TDSUser currentUser = await TDSUser.GetCurrent();
bool isAuthenticated = await currentUser.IsAuthenticated();
if (isAuthenticated) {
  // session token 有效
} else {
  // session token 无效
}
```

### 用户的查询

可以直接构建一个针对 `_User` 的 `LCQuery` 来查询用户：

```cs
LCQuery<TDSUser> userQuery = TDSUser.GetQuery();
```

为了安全起见，**新创建的应用的 `_User` 表默认关闭了 `find` 权限**，这样每位用户登录后只能查询到自己在 `_User` 表中的数据，无法查询其他用户的数据。如果需要让其查询其他用户的数据，建议单独创建一张表来保存这类数据，并开放这张表的 `find` 查询权限。除此之外，还可以在云引擎里封装用户查询相关的方法。

可以参见 [用户对象的安全](#用户对象的安全) 来了解 `_User` 表的一些限制，还可以阅读[数据和安全](/sdk/storage/guide/security)来了解更多 class 级权限设置的方法。

### 关联用户对象

关联 `TDSUser` 的方法和 `LCObject` 是一样的。下面的代码为一名作者保存了一本书，然后获取所有该作者写的书：

```cs
LCObject book = new LCObject("Book");
TDSUser author = await LCUser.GetCurrent();
book["title"] = "我的第五本书";
book["author"] = author;
await book.Save();

LCQuery<LCObject> query = new LCQuery<LCObject>("Book");
query.WhereEqualTo("author", author);
// books 是包含同一作者所有 Book 对象的数组
ReadOnlyCollection<LCObject> books = await query.Find();
```

### 用户对象的安全

`TDSUser` 类自带安全保障，只有通过登录等经过鉴权的方法获取到的 `TDSUser` 才能进行保存或删除相关的操作，保证每个用户只能修改自己的数据。

这样设计是因为 `TDSUser` 中存储的大多数数据都比较敏感，包括手机号、社交网络账号等等。为了用户的隐私安全，即使是应用的开发者也应避免直接接触这些数据。

下面的代码展现了这种安全措施：

```cs
try {
  TDSUser tdsUser = await TDSUser.LoginWithTapTap();
  // 试图修改用户名
  user["username"] = "Jerry";
  // 可以执行，因为用户已鉴权
  await user.Save();

  // 绕过鉴权直接获取用户
  LCQuery<TDSUser> userQuery = TDSUser.GetQuery();
  TDSUser unauthenticatedUser = await userQuery.Get(user.ObjectId);
  unauthenticatedUser["username"] = "Toodle";

  // 会出错，因为用户未鉴权
  unauthenticatedUser.Save();
} catch (LCException e) {
  print($"{e.code} : {e.message}");
}
```

通过 `TDSUser.GetCurrent()` 获取的 `LCUser` 总是经过鉴权的。

要查看一个 `TDSUser` 是否经过鉴权，可以调用 `IsAuthenticated` 方法。通过经过鉴权的方法获取到的 `TDSUser` 无需进行该检查。

### 其他对象的安全

对于给定的一个对象，可以指定哪些用户有权限读取或修改它。为实现该功能，每个对象都有一个由 `ACL` 对象组成的访问控制表。请参阅[《ACL 权限管理开发指南》](https://leancloud.cn/docs/acl-guide.html)。

### 第三方账户登录

我们在登录功能的开发指南中已经介绍了如何[使用 TapTap OAuth 授权结果直接登录账户系统](/sdk/taptap-login/guide/start#用-taptap-oauth-授权结果直接登录账户系统)。

其实除了 TapTap 外，我们也支持直接使用第三方社交平台（例如微信、微博、QQ 等）的账户信息来创建自己的账户体系并完成登录，也允许将既有账户与第三方账户绑定起来，这样终端用户后续可以直接用第三方账户信息来便捷登录。

例如以下的代码展示了终端用户使用微信登录的处理流程：

```cs
Dictionary<string, object> thirdPartyData = new Dictionary<string, object> {
  // 必须
  { "openid", "OPENID" },
  { "access_token", "ACCESS_TOKEN" },
  { "expires_in", 7200 },

  // 可选
  { "refresh_token", "REFRESH_TOKEN" },
  { "scope", "SCOPE" }
};
TDSUser currentUser = await TDSUser.LoginWithAuthData(thirdPartyData, "weixin");
```

`loginWithAuthData` 系列方法需要两个参数来唯一确定一个账户：

- 第三方平台的名字，就是前例中的 `weixin`，该名字由应用层自己决定。
- 第三方平台的授权信息，就是前例中的 `thirdPartyData`（一般包括 `uid`、`token`、`expires` 等信息，与具体的第三方平台有关）。

云端会使用第三方平台的鉴权信息来查询是否已经存在与之关联的账户。如果存在的话，则返回 `200 OK` 状态码，同时附上用户的信息（包括 [`sessionToken`](#设置当前用户)）。如果第三方平台的信息没有和任何账户关联，客户端会收到 `201 Created` 状态码，意味着新账户被创建，同时附上用户的 `objectId`、`createdAt`、`sessionToken` 和一个自动生成的 `username`，例如：

```json
{
  "username":     "k9mjnl7zq9mjbc7expspsxlls",
  "objectId":     "5b029266fb4ffe005d6c7c2e",
  "createdAt":    "2018-05-21T09:33:26.406Z",
  "updatedAt":    "2018-05-21T09:33:26.575Z",
  "sessionToken": "…",
  // authData 通常不会返回，继续阅读以了解其中原因
  "authData": {
    "weixin": {
      "openid":        "OPENID",
      "access_token":  "ACCESS_TOKEN",
      "expires_in":    7200,
      "refresh_token": "REFRESH_TOKEN",
      "scope":         "SCOPE"
    }
  }
  // …
}
```

这时候我们会看到 `_User` 表中出现了一条新的账户记录，账户中有一个名为 `authData` 的列，保存了第三方平台的授权信息。出于安全考虑，`authData` 不会被返回给客户端，除非它属于当前用户。

开发者需要自己完成第三方平台的鉴权流程（一般通过 OAuth 1.0 或 2.0），以获取鉴权信息，继而到云端来登录。

#### Signin With Apple

如果你需要开发 [Sigin With Apple](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api)，云服务可以帮你校验 `identityToken`，并获取 Apple 的 `access_token`。Apple Sign In 的 `authData` 结构如下：

```
{
  "lc_apple": {
    "uid" : "从 Apple 获取到的 User Identifier",
    "identity_token" : "从苹果获取到的 identityToken"
    "code" : "从苹果获取到的 Authorization Code"
  }
}
```
`authData` 中的 key 的作用：

* **`lc_apple`**：只有 platform 为 `lc_apple` 时，云服务才会执行 `identity_token` 和 `code` 的逻辑。
* **`uid`**：必填。云服务通过 `uid` 判断是否存在用户。
* **`identity_token`**：可选。`authData` 中有 `identity_token` 时云端会自动校验 `identity_token` 的有效性。开发者需要在云服务控制台「存储」-「用户」-「设置」-「第三方集成」中填写 Apple 的相关信息。
* **`code`**：可选。`authData` 中有 `code` 时云端会自动用该 `code` 向 Apple 换取 `access_token` 和 `refresh_token`。开发者需要在云服务控制台「存储」-「用户」-「设置」-「第三方集成」中填写 Apple 的相关信息。

##### 获取 Client ID

Client ID 用于校验 `identity_token` 及获取 `access_token`，指的是 Apple 应用的 identifier，也就是 `AppID` 或 `serviceID`。对于原生应用来说，指的是 Xcode 中的 Bundle Identifier，例如 `com.mytest.app`。详情请参考 [Apple 的文档](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)。

##### 获取 Private Key 及 Private Key ID

Private Key 用于获取 `access_token`。登录 Apple 开发者平台，在左侧的 「Certificates, Identifiers & Profiles」 中选择 「Keys」，添加一个用于 Apple Sign In 的 Private Key，下载 XXXXX.p8 文件，同时在下载 Key 的页面获得 Private Key ID。详情请参考[ Apple 的文档](https://help.apple.com/developer-account/#/dev77c875b7e)。

将 Key ID 填写到控制台，将下载下来的 Private Key 文件上传到控制台。控制台只能上传 Private Key 文件，无法查看及下载其内容。

##### 获取 Team ID

Team ID 用于获取 `access_token`。登录 Apple 开发者平台，在右上角或 Membership 页面即可看到自己所属开发团队的 Team ID。注意选择 Bundle ID 对应的 Team。

##### 使用 Apple Sign In 登录云服务

在控制台填写完成所有信息后，使用以下代码登录。

```cs
Dictionary<string, object> appleAuthData = new Dictionary<string, object> {
  // 必须
  { "uid", "USER IDENTIFIER" },

  // 可选
  { "identity_token", "IDENTITY TOKEN" },
  { "code", "AUTHORIZATION CODE" }
};
TDSUser currentUser = await TDSUser.LoginWithAuthData(appleAuthData, "lc_apple");
```

#### 鉴权数据的保存

`_User` class 中的 `authData` 是一个以平台名为键名，鉴权信息为键值的 JSON 对象。

一个关联了微信账户的用户应该会有下列对象作为 `authData`：

```json
{
  "weixin": {
    "openid":        "…",
    "access_token":  "…",
    "expires_in":    7200,
    "refresh_token": "…",
    "scope":         "…"
  }
}
```

而一个关联了微博账户的用户，则会有如下的 `authData`：

```json
{
    "weibo": {
      "refresh_token": "2.0xxx",
      "uid": "271XFEFEW273",
      "expires_in": 115057,
      "access_token": "2.00xxx",
    }
}
```

我们允许一个账户绑定多个第三方平台的鉴权数据，这样如果某个用户同时关联了微信和微博账户，则其 `authData` 可能会是这样的：

```json
{
  "weixin": {
    "openid":        "…",
    "access_token":  "…",
    "expires_in":    7200,
    "refresh_token": "…",
    "scope":         "…"
  }
  "weibo": {
    "refresh_token": "2.0xxx",
    "uid": "271XFEFEW273",
    "expires_in": 115057,
    "access_token": "2.00xxx",
  }
}
```

理解 `authData` 的数据结构至关重要。一个终端用户通过如下的鉴权信息来登录的时候，

```json
"weixin": {
  "openid":        "OPENID",
  "access_token":  "ACCESS_TOKEN",
  "expires_in":    7200,
  "refresh_token": "REFRESH_TOKEN",
  "scope":         "SCOPE"
}
```

云端首先会查找账户系统（_User 表），看看是否存在 authData.weixin.openid = “ OPENID ” 的账户，如果存在，则返回现有账户，如果不存在那么就创建一个新账户，同时将上面的鉴权信息写入新账户的 `authData` 属性中，并将新账户的数据当成结果返回。

云端会自动为 `_User` class 中每个用户的 `authData.<PLATFORM>.<uid>` 创建唯一索引，从而避免重复数据。
`<uid>` 在微信等部分云服务内建支持的第三方平台上为 `openid` 字段，在其他第三方平台（包括部分云服务专门支持的第三方平台和所有云服务没有专门支持的第三方平台）上为 `uid` 字段。

#### 自动验证第三方平台授权信息

为了确保账户数据的有效性，云端还支持对部分平台的 `Access Token` 的有效性进行自动验证，以防止伪造账户数据。如果有效性验证不通过，云端会返回 `invalid authData` 错误，关联不会被建立。对于云端无法识别的服务，开发者需要自己去验证 `Access Token` 的有效性。
比如，注册、登录时分别通过云引擎的 `beforeSave hook`、`beforeUpdate hook` 来验证 `Access Token` 有效性。

如果希望使用这一功能，则在开始使用前，需要在 **云服务控制台 > 数据存储 > 用户 > 设置** 配置相应平台的 **应用 ID** 和 **应用 Secret Key**。

如果不希望云端自动验证 `Access Token`，可以在 **云服务控制台 > 数据存储 > 设置** 里面取消勾选 **第三方登录时，验证用户 AccessToken 合法性**。

配置平台账号的目的在于创建 `TDSUser` 时，云端会使用相关信息去校验请求参数 `thirdPartyData` 的合法性，确保 `TDSUser` 实际对应着一个合法真实的用户，确保平台安全性。

#### 绑定第三方账户

用户已经有了 LCUser 并登录成功后，可以绑定新的第三方账号信息。
绑定成功后，新的第三方账户信息会被添加到 LCUser 的 authData 字段里。

例如，下面的代码可以关联微信账户：

```cs
await currentUser.AssociateAuthData(weixinData, "weixin");
```

为节省篇幅，上面的代码示例中没有给出具体的微信平台授权信息，相关内容请参考上面的[「第三方账户登录」](#第三方账户登录)一节。

#### 解除与第三方账户的关联

类似地，可以解绑第三方账户。

例如，下面的代码可以解除用户和微信账户的关联：

```cs
TDSUser currentUser = await TDSUser.GetCurrent();
await currentUser.DisassociateWithAuthData("weixin");
```

#### 扩展：接入 UnionID 体系，打通不同子产品的账号系统

随着第三方平台的账户体系变得日渐复杂，它们的第三方鉴权信息出现了一些较大的变化。下面我们以最典型的微信开放平台为例来进行说明。

当一个用户在移动应用内登录微信账号时，会被分配一个 OpenID；在微信小程序内登录账号时，又会被分配另一个不同的 OpenID。这样的架构会导致的问题是，使用同一个微信号的用户，也无法在微信开发平台下的移动应用和小程序之间互通。

微信官方为了解决这个问题，引入了 `UnionID` 的体系，以下为其官方说明：

> 通过获取用户基本信息接口，开发者可通过 OpenID 来获取用户基本信息，而如果开发者拥有多个公众号，可使用以下办法通过 UnionID 机制来在多公众号之间进行用户帐号互通。只要是同一个微信开放平台帐号下的公众号，用户的 UnionID 是唯一的。换句话说，同一用户，对同一个微信开放平台帐号下的不同应用，UnionID 是相同的。

其他平台，如 QQ 和微博，与微信的设计也基本一致。

云服务支持 `UnionID` 体系。你只需要给 `loginWithauthData` 和 `associateWithauthData` 接口传入更多的第三方鉴权信息，即可完成新 UnionID 体系的集成。新增加的第三方鉴权登录选项包括：

- unionId，指第三方平台返回的 UnionId 字符串。
- unionId platform，指 unionId 对应的 platform 字符串，由应用层自己指定，[后面](#该如何指定 -unionIdPlatform)会详述。
- asMainAccount，指示是否把当前平台的鉴权信息作为主账号来使用。如果作为主账号，那么就由当前用户唯一占有该 unionId，以后其他平台使用同样的 unionId 登录的话，会绑定到当前的用户记录上来；否则，当前应用的鉴权信息会被绑定到其他账号上去。

下面让我们通过一个例子来说明如何使用这些参数完成 UnionID 登录。

假设云服务在微信开放平台上有两个应用，一个是「云服务通讯」，一个是「云服务技术支持」，这两个应用在接入第三方鉴权的时候，分别使用了 `wxleanoffice` 和 `wxleansupport` 作为 platform 来进行登录。现在我们开启 UnionID 的用户体系，希望同一个微信用户在这两个应用中都能对应到同一个账户系统（_User 表中的同一条记录），同时我们决定将 `wxleanoffice` 平台作为主账号平台。

假设对于用户 A，微信给 ta 为 云服务分配的 UnionId 为 `unionid4a`，而对两个应用的授权信息分别为：

```json
"wxleanoffice": {
  "access_token": "officetoken",
  "openid": "officeopenid",
  "expires_in": 1384686496
},
"wxleansupport": {
  "openid": "supportopenid",
  "access_token": "supporttoken",
  "expires_in": 1384686496
}
```

现在，用户 A 在「云服务通讯」中通过微信登录，其调用请求为：

```cs
Dictionary<string, object> thirdPartyData = new Dictionary<string, object> {
  // 必须
  { "uid", "officeopenid" },
  { "access_token", "officetoken" },
  { "expires_in", 1384686496 },
  { "unionId", "unionid4a" },   // 新增属性

  // 可选
  { "refresh_token", "..." },
  { "scope", "SCOPE" }
};
LCUserAuthDataLoginOption option = new LCUserAuthDataLoginOption();
option.AsMainAccount = true;
option.UnionIdPlatform = "weixin";
TDSUser currentUser = await TDSUser.LoginWithAuthDataAndUnionId(
    thirdPartyData, "wxleanoffice", "unionid4a",
    option: option);
```

> 注意代码中将微信传回来的 openid 属性改为了 uid，这是因为云端要求对于自定义的 platform，只能使用 uid 这样的属性名，才能保证自动建立 `authData.<PLATFORM>.uid` 的唯一索引，具体可以参考[数据存储 REST API 使用详解](/sdk/storage/guide/rest)的《连接用户账户和第三方平台》一节。

如果用户 A 是第一次在「云服务通讯」中通过微信登录，那么 _User 表中会增加一个新用户（假设其 objectId 为 `ThisIsUserA`），其 `authData` 的结果如下：

```json
{
  "wxleanoffice": {
    "platform": "weixin",
    "uid": "officeopenid",
    "expires_in": 1384686496,
    "main_account": true,
    "access_token": "officetoken",
    "unionid": "unionid4a"
  },
  "_weixin_unionid": {   // 新增键值对
    "uid": "unionid4a"
  }
}
```

可以看到，与之前的第三方登录 API 相比，这里由于登录时指定了 `asMainAccount` 为 true，所以 authData 的第一级子目录中增加了 `_weixin_unionid` 的键值对，这里的 `weixin` 就是我们指定的 `unionIdPlatform` 的值。`_weixin_unionid` 这个增加的键值对非常重要，以后我们判断是否存在同样 UnionID 的账户就是依靠它来查找的，而是否增加这个键值对，则是由登录时指定的 `asMainAccount` 的值决定的：

- 当 `asMainAccount` 为 true 时，云端会在 `authData` 下面增加名为 `_{unionIdPlatform}_unionid` 的键值对，当前账号就会作为这一个 UnionID 对应的主账号被唯一确定。
- 当 `asMainAccount` 为 false 时，云端不会在 `authData` 下面增加名为 `_{unionIdPlatform}_unionid` 的键值对，此时如果通过提供的 UnionID 可以找到主账号，则会将当前的鉴权信息合并进主账号的 `authData` 属性里，同时返回主账号对应的 _User 表记录；如果通过提供的 UnionID 可以找不到主账号，则会根据平台的 `openid` 去查找账户，找到匹配的账户就返回匹配的，找不到就新建一个账户，此时的处理逻辑与不使用 UnionID 时的逻辑完全一致。

接下来，用户 A 继续在「云服务技术支持」中进行微信登录，其登录逻辑为：

```cs
Dictionary<string, object> thirdPartyData = new Dictionary<string, object> {
  // 必须
  { "uid", "supportopenid" },
  { "access_token", "supporttoken" },
  { "expires_in", 1384686496 },
  { "unionId", "unionid4a" },

  // 可选
  { "refresh_token", "..." },
  { "scope", "SCOPE" }
};
LCUserAuthDataLoginOption option = new LCUserAuthDataLoginOption();
option.AsMainAccount = false;
option.UnionIdPlatform = "weixin";     // 这里指定 unionIdPlatform，使用「weixin」来指代微信平台。
TDSUser currentUser = await TDSUser.LoginWithAuthDataAndUnionId(
    thirdPartyData, "wxleansupport", "unionid4a",
    option: option);
```

与「云服务通讯」中的登录过程相比，在「云服务技术支持」这个应用中，我们在登录时只是将 `asMainAccount` 设为了 false。 这时我们看到，本次登录得到的还是 objectId 为 `ThisIsUserA` 的 _User 表记录（同一个账户），同时该账户的 `authData` 属性中发生了变化，多了 `wxleansupport` 的数据，如下：

```json
{
  "wxleanoffice": {
    "platform": "weixin",
    "uid": "officeopenid",
    "expires_in": 1384686496,
    "main_account": true,
    "access_token": "officetoken",
    "unionid": "unionid4a"
  },
  "_weixin_unionid": {
    "uid": "unionid4a"
  },
  "wxleansupport": {
    "platform": "weixin",
    "uid": "supportopenid",
    "expires_in": 1384686496,
    "main_account": false,
    "access_token": "supporttoken",
    "unionid": "unionid4a"
  }
}
```

在新的登录方式中，当一个用户以「平台名为 `wxleanoffice`、uid 为 `officeopenid`、UnionID 为 `unionid4a`」的第三方鉴权信息登录得到新的 `TDSUser` 后，接下来这个用户以「平台名为 `wxleansupport`、uid 为 `supportopenid`、UnionID 为 `unionid4a`」的第三方鉴权信息登录时，云端判定是同样的 UnionID，就直接把来自 `wxleansupport` 的新用户数据加入到已有账户的 `authData` 里了，不会再创建新的账户。

这样一来，云端通过识别平台性的用户唯一标识 UnionID，让来自同一个 UnionID 体系内的应用程序、小程序等不同平台的用户都绑定到了一个 `TDSUser` 上，实现互通。

##### 为 UnionID 建立索引

云端会为 UnionID 自动建立索引，不过因为自动创建基于请求的抽样统计，可能会滞后。
因此，我们推荐自行创建相关索引，特别是用户量（`_User`　表记录数）很大的应用，更需要预先创建索引，否则用户使用 UnionID 账号登录时可能超时失败。
以上面的微信 UnionID 为例，建议在控制台预先创建下列唯一索引（允许缺失值）：

- `authData.wxleanoffice.uid`
- `authData.wxleansupport.uid`
- `authData._weixin_unionid.uid`

##### 该如何指定 unionIdPlatform

从上面的例子可以看出，使用 UnionID 登录的时候，需要指定 `unionIdPlatform` 的主要目的，就是为了便于查找已经存在的唯一主账号。云端会在主账号对应账户的 `authData` 属性中增加一个 `_{unionIdPlatform}_unionid` 键值对来标记唯一性，终端用户在其他应用中登录的时候，云端会根据参数中提供的 `uniondId` + `unionIdPlatform` 的组合，在 `_User` 表中进行查找，这样来确定唯一的既存主账号。

本来 `unionIdPlatform` 的取值，应该是开发者可以自行决定的，但是 Javascript SDK 基于易用性的目的，在 `loginWithAuthDataAndUnionId` 之外，还额外提供了两个接口：

- `AV.User.loginWithQQAppWithUnionId`，这里默认使用 `qq` 作为 `unionIdPlatform`。
- `AV.User.loginWithWeappWithUnionId`，这里默认使用 `weixin` 作为 `unionIdPlatform`。

从我们的统计来看，这两个接口已经被很多开发者接受，在大量线上产品中产生着实际数据。所以为了避免数据在不同平台（例如 Android 和 iOS 应用）间发生冲突，建议大家统一按照 Javascript SDK 的默认值来设置 `unionIdPlatform`，即：

- 微信平台的多个应用，统一使用 `weixin` 作为 `unionIdPlatform`；
- QQ 平台的多个应用，统一使用 `qq` 作为 `unionIdPlatform`；
- 微博平台的多个应用，统一使用 `weibo` 作为 `unionIdPlatform`；
- 除此之外的其他平台，开发者可以自行定义 `unionIdPlatform` 的名字，只要自己保证多个应用间统一即可。

##### 主副应用不同登录顺序出现的不同结果

上面的流程是用户先登录了「云服务通讯」这个主应用，然后再登录「云服务技术支持」这个副应用，所以账号都被通过 UnionID 有效关联起来了。可能有人会想到另外一个问题，如果用户 B 先登录副应用，后登录主应用，这时候会发生什么事情呢？

用户 B 首先登录副应用的时候，提供了「平台名为 `wxleansupport`、uid 为 `supportopenid`、UnionID 为 `unionid4a`」的第三方鉴权信息，并且指定「UnionIDPlatform 为 `weixin`、`asMainAccount` 为 false」（与上面的调用完全一致），此时云端由于找不到存在的 UnionID，会新建一个 `TDSUser` 对象，该账户 `authData` 结果为：

```json
{
  "wxleansupport": {
    "platform": "weixin",
    "uid": "supportopenid",
    "expires_in": 1384686496,
    "main_account": false,
    "access_token": "supporttoken",
    "unionid": "unionid4a"
  }
}
```

用户 B 接着又使用了主应用，ta 再次通过微信登录，此时以「平台名为 `wxleanoffice`、uid 为 `officeopenid`、UnionID 为 `unionid4a`」的第三方鉴权信息，以及「UnionIDPlatform 为 `weixin`、`asMainAccount` 为 true」的参数进行登录，此时云端由于找不到存在的 UnionID，会再次新建一个 `TDSUser` 对象，该账户 `authData` 结果为：

```json
{
  "wxleanoffice": {
    "platform": "weixin",
    "uid": "officeopenid",
    "expires_in": 1384686496,
    "main_account": true,
    "access_token": "officetoken",
    "unionid": "unionid4a"
  },
  "_weixin_unionid": {
    "uid": "unionid4a"
  }
}
```

还有更复杂的情况。如果某公司的产品之前就接入了微信登录，产生了很多存量用户，并且分散在不同的子产品中，这时候怎么办？我们接下来专门讨论此时的解决方案。

##### 存量账户如何通过 UnionID 实现关联

还是以我们的两个子产品「云服务通讯」（后续以「产品 1」指代）和「云服务技术支持为例」（后续以「产品 2」指代），在接入 UnionID 之前，我们就接入了之前版本的微信平台登录，这时候账户系统内可能存在多种账户：

- 只使用产品 1 的微信用户 A
- 只使用产品 2 的微信用户 B
- 同时使用两个产品的微信用户 C

此时的存量账户表如下所示：

objectId | 微信用户 | authData.{platform} | authData._{platform}_unionid
------ | ------ | ------ | ------
1 | UserA | openid1（对应产品 1） | N/A
2 | UserB | openid2（对应产品 2） | N/A
3 | UserC | openid3（对应产品 1） | N/A
4 | UserC | openid4（对应产品 2） | N/A

现在我们对两个子产品进行升级，接入 UnionID 体系。这时因为已经有同一个微信用户在不同子产品中创建了不同的账户（例如 objectId 为 3 和 4 的账户），我们需要确定以哪个平台的账号为主。比如决定使用「云服务通讯」上生成的账号为主账号，则在该应用程序更新版本时，使用 `asMainAccount=true` 参数。这个应用带着 UnionID 登录匹配或创建的账号将作为主账号，之后所有这个 UnionID 的登录都会匹配到这个账号。请注意这时 `_User` 表里会剩下一些用户数据，也就是没有被选为主账号的、其他平台的同一个用户的旧账号数据（例如 objectId 为 2 和 4 的账户）。这部分数据会继续服务于已经发布的但仍然使用 OpenID 登录的旧版应用。

接下来我们看一下，如果以产品 1 的账户作为「主账户」，按照前述的方式同时提供 openid/unionid 完成登录，则最后达到的结果是：

1. 使用老版本的用户，不管在哪个产品里面，都可以和往常一样通过 openid 登录到正确的账户；
2. 使用产品 1 的新版本的老用户，通过 openid/unionid 组合，还是绑定到原来的账户。例如 UserC 在产品 1 中通过 openid3/unionId3 还是会绑定到 objectId=3 的账户（会增加 uniondId 记录）；而 UserC 在产品 2 的新版本中，通过 openid4/unionId3 的组合则会绑定到 objectId=3 的账户，而不再是原来的 objectId=4 的账户。
3. 使用产品 1 的新版本的新用户，通过 openid/unionid 组合，会创建新的账户；之后该用户再使用产品 2 的新版本，也会绑定到刚才创建的新账户上。

以上面的三个用户为例，他们分别升级到两个产品的最新版，且最终都会启用两个产品，则账户表的最终结果如下：

objectId | 微信用户 | authData.{platform} | authData._{platform}_unionid
------ | ------ | ------ | ------
1 | UserA | openid1（对应产品 1）/openid6（对应产品 2） | unionId_user_A
2 | UserB | openid2（对应产品 2） | N/A
3 | UserC | openid3（对应产品 1）/openid4（对应产品 2） | unionId_user_C
4 | UserC | openid4（对应产品 2） | N/A
5 | UserB | openid5（对应产品 1）/openid2（对应产品 2） | unionId_user_B

之后有新的用户 D，分别在两个产品的新版本中登录，则账户表中会增加一条新的 objectId=6 的记录，结果如下：

objectId | 微信用户 | authData.{platform} | authData._{platform}_unionid
------ | ------ | ------ | ------
1 | UserA | openid1（对应产品 1）/openid6（对应产品 2） | unionId_user_A
2 | UserB | openid2（对应产品 2） | N/A
3 | UserC | openid3（对应产品 1）/openid4（对应产品 2） | unionId_user_C
4 | UserC | openid4（对应产品 2） | N/A
5 | UserB | openid5（对应产品 1）/openid2（对应产品 2） | unionId_user_B
6 | UserD | openid7（对应产品 1）/openid8（对应产品 2） | unionId_user_D

如果之后我们增加了新的子产品 3，这些用户在子产品 3 中也进行微信登录的话，那么四个用户还是会继续绑定到 objectId 为 1/3/5/6 的主账户。此时账户表的结果会变为：

objectId | 微信用户 | authData.{platform} | authData._{platform}_unionid
------ | ------ | ------ | ------
1 | UserA | openid1（对应产品 1）/openid6（对应产品 2）/openid9（对应产品 3） | unionId_user_A
2 | UserB | openid2（对应产品 2） | N/A
3 | UserC | openid3（对应产品 1）/openid4（对应产品 2）/openid10（对应产品 3） | unionId_user_C
4 | UserC | openid4（对应产品 2） | N/A
5 | UserB | openid5（对应产品 1）/openid2（对应产品 2）/openid11（对应产品 3） | unionId_user_B
6 | UserD | openid7（对应产品 1）/openid8（对应产品 2）/openid12（对应产品 3） | unionId_user_D

## 角色

随着用户量的增长，你可能会发现相比于为每一名用户单独设置权限，将预先设定好的权限直接分配给一部分用户是更好的选择。为了迎合这种需求，云服务支持基于角色的权限管理。请参阅[《ACL 权限管理开发指南》](https://leancloud.cn/docs/acl-guide.html)。

## 全文搜索

全文搜索是一个针对应用数据进行全局搜索的接口，它基于搜索引擎构建，提供更强大的搜索功能。要深入了解其用法和阅读示例代码，请阅读[全文搜索指南](/sdk/storage/guide/fulltext-search)。