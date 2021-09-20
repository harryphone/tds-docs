---
id: java
title: 数据存储开发指南 · Java 
sidebar_label: Java 指南
---



数据存储是云服务提供的核心功能之一，可用于存放和查询应用数据。下面的代码展示了如何创建一个对象并将其存入云端：

```java
// 构建对象
LCObject todo = new LCObject("Todo");

// 为属性赋值
todo.put("title",   "工程师周会");
todo.put("content", "周二两点，全体成员");

// 将对象保存到云端
todo.saveInBackground().subscribe(new Observer<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject todo) {
        // 成功保存之后，执行其他逻辑
        System.out.println("保存成功。objectId：" + todo.getObjectId());
    }
    public void onError(Throwable throwable) {
        // 异常处理
    }
    public void onComplete() {}
});
```

我们为各个平台或者语言开发的 SDK 在底层都是通过 HTTPS 协议调用统一的 REST API，提供完整的接口对数据进行各类操作。

## SDK 安装与初始化

请阅读[数据存储、即时通讯 Java SDK 配置指南](/sdk/storage/guide/setup-java/)。

> Android SDK 老版本迁移
>
> 如果你还在在使用我们老版本 Android SDK（所有版本号低于 `5.0.0`，`groupId` 为 `cn.leancloud.android` 的 libraries），要迁移到新版的 Unified SDK，具体迁移方法见 [Java SDK 的 README](https://github.com/leancloud/java-unified-sdk#migration-to-8x)。

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


```java
// 基本类型
boolean bool = true;
int number = 2018;
String string = number + " 流行音乐榜单";
Date date = new Date();
byte[] data = "Hello world!".getBytes();
ArrayList<Object> arrayList = new ArrayList<>();
arrayList.add(number);
arrayList.add(string);
HashMap<Object, Object> hashMap = new HashMap<>();
hashMap.put("number", number);
hashMap.put("string", string);

// 构建对象
LCObject testObject = new LCObject("TestObject");
testObject.put("testBoolean",   bool);
testObject.put("testInteger",   number);
testObject.put("testDate",      date);
testObject.put("testData",      data);
testObject.put("testArrayList", arrayList);
testObject.put("testHashMap",   hashMap);
testObject.save();
```


我们不推荐通过 `byte[]` 在 `LCObject` 里面存储图片、文档等大型二进制数据。每个 `LCObject` 的大小不应超过 **128 KB**。如需存储大型文件，可创建 `LCFile` 实例并将将其关联到 `LCObject` 的某个属性上。参见 [文件](#文件)。

注意：时间类型在云端将会以 UTC 时间格式存储，但是客户端在读取之后会转化成本地时间。

**云服务控制台 > 数据存储 > 结构化数据** 中展示的日期数据也会依据操作系统的时区进行转换。一个例外是当你通过 REST API 获得数据时，这些数据将以 UTC 呈现。你可以手动对它们进行转换。

若想了解云服务是如何保护应用数据的，请阅读[数据和安全](/sdk/storage/guide/security/)。

### 构建对象

下面的代码构建了一个 class 为 `Todo` 的 `LCObject`：

```java
LCObject todo = new LCObject("Todo");
```


在构建对象时，为了使云端知道对象属于哪个 class，需要将 class 的名字作为参数传入。你可以将云服务里面的 class 比作关系型数据库里面的表。一个 class 的名字必须以字母开头，且只能包含数字、字母和下划线。

### 保存对象

下面的代码将一个 class 为 `Todo` 的对象存入云端：


```java
// 构建对象
LCObject todo = new LCObject("Todo");

// 为属性赋值
todo.put("title", "马拉松报名");
todo.put("priority", 2);

// 将对象保存到云端
todo.saveInBackground().subscribe(new Observer<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject todo) {
        // 成功保存之后，执行其他逻辑
        System.out.println("保存成功。objectId：" + todo.getObjectId());
    }
    public void onError(Throwable throwable) {
        // 异常处理
    }
    public void onComplete() {}
});
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
`objectId` | `String` | 该对象唯一的 ID 标识。
`ACL` | `LCACL` | 该对象的权限控制，实际上是一个 JSON 对象，控制台做了展现优化。
`createdAt` | `Date` | 该对象被创建的时间。
`updatedAt` | `Date` | 该对象最后一次被修改的时间。


这些属性的值会在对象被存入云端时自动填入，代码中尚未保存的 `LCObject` 不存在这些属性。

属性名（**keys**）只能包含字母、数字和下划线。自定义属性不得以双下划线（`__`）开头或与任何系统保留字段和内置属性（`ACL`、`className`、`createdAt`、`objectId` 和 `updatedAt`）重名，无论大小写。

属性值（**values**）可以是字符串、数字、布尔值、数组或字典（任何能以 JSON 编码的数据）。参见 [数据类型](#数据类型)。

我们推荐使用驼峰式命名法（CamelCase）为类和属性来取名。类，采用大驼峰法，如 `CustomData`。属性，采用小驼峰法，如 `imageUrl`。

### 获取对象

对于已经保存到云端的 `LCObject`，可以通过它的 `objectId` 将其取回：


```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.getInBackground("582570f38ac247004f39c24b").subscribe(new Observer<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject todo) {
        // todo 就是 objectId 为 582570f38ac247004f39c24b 的 Todo 实例
        String title    = todo.getString("title");
        int priority    = todo.getInt("priority");

        // 获取内置属性
        String objectId = todo.getObjectId();
        Date updatedAt  = todo.getUpdatedAt();
        Date createdAt  = todo.getCreatedAt();
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

对象拿到之后，可以通过 `get` 方法来获取各个属性的值。注意 `objectId`、`updatedAt` 和 `createdAt` 这三个内置属性不能通过 `get` 获取或通过 `set` 修改，只能由云端自动进行填充。尚未保存的 `LCObject` 不存在这些属性。

如果你试图获取一个不存在的属性，SDK 不会报错，而是会返回 `null`。

#### 同步对象

当云端数据发生更改时，你可以调用 `fetchInBackground` 方法来刷新对象，使之与云端数据同步：


```java
LCObject todo = LCObject.createWithoutData("Todo", "582570f38ac247004f39c24b");
todo.fetchInBackground().subscribe(new Observable<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject todo) {
        // todo 已刷新
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

刷新操作会强行使用云端的属性值覆盖本地的属性。因此如果本地有属性修改，**`fetchInBackground` 操作会丢弃这些修改**。为避免这种情况，你可以在刷新时指定 **需要刷新的属性**，这样只有指定的属性会被刷新（包括内置属性 `objectId`、`createdAt` 和 `updatedAt`），其他属性不受影响。

```java
LCObject todo = LCObject.createWithoutData("Todo", "582570f38ac247004f39c24b");
String keys = "priority, location";
todo.fetchInBackground(keys).subscribe(new Observable<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject todo) {
        // 只有 priority 和 location 会被获取和刷新
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

如果想要确保只刷新已经保存到云端的对象，那么可以用 `fetchIfNeededInBackground` 替换 `fetchInBackground`。
调用 `fetchIfNeededInBackground` 方法时，如果是本地构造尚未保存的对象，那么不会访问云端，onNext 方法中传入的是本地对象。

### 更新对象

要更新一个对象，只需指定需要更新的属性名和属性值，然后调用 `saveInBackground` 方法。例如：

要更新一个对象，只需指定需要更新的属性名和属性值，然后调用 `save` 方法。例如：

```java
LCObject todo = LCObject.createWithoutData("Todo", "582570f38ac247004f39c24b");
todo.put("content", "这周周会改到周三下午三点。");
todo.save();
```

云服务会自动识别需要更新的属性并将对应的数据发往云端，未更新的属性会保持原样。

#### 有条件更新对象

通过传入 `query` 选项，可以按照指定条件去更新对象——当条件满足时，执行更新；条件不满足时，不执行更新并返回 `305` 错误。

例如，用户的账户表 `Account` 有一个余额字段 `balance`，同时有多个请求要修改该字段值。为避免余额出现负值，只有当金额小于或等于余额的时候才能接受请求：

```java
LCObject account = LCObject.createWithoutData("Account", "5745557f71cfe40068c6abe0");
// 对 balance 原子减少 100
final int amount = -100;
account.increment("balance", amount);
// 设置条件
LCSaveOption option = new LCSaveOption();
option.query(new LCQuery<>("Account").whereGreaterThanOrEqualTo("balance", -amount));
// 操作结束后，返回最新数据。
// 如果是新对象，则所有属性都会被返回，
// 否则只有更新的属性会被返回。
option.setFetchWhenSave(true);
account.saveInBackground(option).subscribe(new Observer<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject account) {
        System.out.println("当前余额为：" + account.get("balance"));
    }
    public void onError(Throwable throwable) {
        System.out.println("余额不足，操作失败！");
    }
    public void onComplete() {}
});
```


**`query` 选项只对已存在的对象有效**，不适用于尚未存入云端的对象。

`query` 选项在有多个客户端需要更新同一属性的时候非常有用。相比于通过 `LCQuery` 查询 `LCObject` 再对其进行更新的方法，这样做更加简洁，并且能够避免出现差错。

#### 更新计数器

设想我们正在开发一个微博，需要统计一条微博有多少个赞和多少次转发。由于赞和转发的操作可能由多个客户端同时进行，直接在本地更新数字并保存到云端的做法极有可能导致差错。为保证计数的准确性，可以通过 **原子操作** 来增加或减少一个属性内保存的数字：


```java
post.increment("likes", 1);
```

可以指定需要增加或减少的值。若未指定，则默认使用 `1`。

注意，虽然原子增减支持浮点数，但因为底层数据库的浮点数存储格式限制，会有舍入误差。
因此，需要原子增减的字段建议使用整数以避免误差，例如 `3.14` 可以存储为 `314`，然后在客户端进行相应的转换。
否则，以比较大小为条件查询对象的时候，需要特殊处理，
`< a` 需改查 `< a + e`，`> a` 需改查 `> a - e`，`== a` 需改查 `> a - e` 且 `< a + e`，其中 `e` 为误差范围，据所需精度取值，比如 `0.0001`。

#### 更新数组

更新数组也是原子操作。使用以下方法可以方便地维护数组类型的数据：


- `add()` 将指定对象附加到数组末尾。
- `addUnique()` 如果数组中不包含指定对象，则将该对象加入数组。对象的插入位置是随机的。
- `removeAll()` 从数组字段中删除指定对象的所有实例。

例如，`Todo` 用一个 `alarms` 属性保存所有闹钟的时间。下面的代码将多个时间加入这个属性：


```java
Date getDateWithDateString(String dateString) {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    Date date = dateFormat.parse(dateString);
    return date;
}

Date alarm1 = getDateWithDateString("2018-04-30 07:10:00");
Date alarm2 = getDateWithDateString("2018-04-30 07:20:00");
Date alarm3 = getDateWithDateString("2018-04-30 07:30:00");

LCObject todo = new LCObject("Todo");
todo.addAllUnique("alarms", Arrays.asList(alarm1, alarm2, alarm3));
todo.save();
```

### 删除对象

下面的代码从云端删除一个 `Todo` 对象；

```java
LCObject todo = LCObject.createWithoutData("Todo", "582570f38ac247004f39c24b");
todo.delete();
```

注意，删除对象是一个较为敏感的操作，我们建议你阅读[《ACL 权限管理开发指南》](https://leancloud.cn/docs/acl-guide.html)来了解潜在的风险。熟悉 class 级别、对象级别和字段级别的权限可以帮助你有效阻止未经授权的操作。

### 批量操作


```java
// 批量构建和更新
saveAll()
saveAllInBackground()

// 批量删除
deleteAll()
deleteAllInBackground()

// 批量同步
fetchAll()
fetchAllInBackground()
```


下面的代码将所有 `Todo` 的 `isComplete` 设为 `true`：


```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.findInBackground().subscribe(new Observer<List<LCObject>>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(List<LCObject> todos) {
        // 获取需要更新的 todo
        for (LCObject todo : todos) {
            // 更新属性值
            todo.put("isComplete", true);
        }
        // 批量更新
        LCObject.saveAll(todos);
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

虽然上述方法可以在一次请求中包含多个操作，每一个分别的保存或同步操作在计费时依然会被算作一次请求，而所有的删除操作则会被合并为一次请求。
### 后台运行

细心的开发者已经发现，在所有的示例代码中几乎都是用了异步来访问云端，形如 `xxxxInBackground` 的用法都是提供给开发者在主线程调用用以实现后台运行的方法，因此开发者在主线程可以放心地调用这种命名方式的函数。
### 离线存储对象

大多数保存功能可以立刻执行，并通知应用「保存完毕」。不过若不需要知道保存完成的时间，则可使用 `saveEventually` 来代替。

它的优点在于：如果用户目前尚未接入网络，`saveEventually` 会缓存设备中的数据，并在网络连接恢复后上传。如果应用在网络恢复之前就被关闭了，那么当它下一次打开时，SDK 会再次尝试保存操作。

所有 `saveEventually`（或 `deleteEventually`）的相关调用，将按照调用的顺序依次执行。因此，多次对某一对象使用 `saveEventually` 是安全的。

### 数据模型

对象之间可以产生关联。拿一个博客应用来说，一个 `Post` 对象可以与许多个 `Comment` 对象产生关联。云服务支持三种关系：一对一、一对多、多对多。

#### 一对一、一对多关系

一对一、一对多关系可以通过将 `LCObject` 保存为另一个对象的属性值的方式产生。比如说，让博客应用中的一个 `Comment` 指向一个 `Post`。

下面的代码会创建一个含有单个 `Comment` 的 `Post`：


```java
// 创建 post
LCObject post = new LCObject("Post");
post.put("title", "饿了……");
post.put("content", "中午去哪吃呢？");

// 创建 comment
LCObject comment = new LCObject("Comment");
comment.put("content", "当然是肯德基啦！");

// 将 post 设为 comment 的一个属性值
comment.put("parent", post);

// 保存 comment 会同时保存 post
comment.save();
```


云端存储时，会将被指向的对象用 `Pointer` 的形式存起来。你也可以用 `objectId` 来指向一个对象：


```java
LCObject post = LCObject.createWithoutData("Post", "57328ca079bc44005c2472d0");
comment.put("post", post);
```

请参阅 [关系查询](#关系查询) 来了解如何获取关联的对象。

#### 多对多关系

想要建立多对多关系，最简单的办法就是使用 **数组**。在大多数情况下，使用数组可以有效减少查询的次数，提升程序的运行效率。但如果有额外的属性需要附着于两个 class 之间的关联，那么使用 **中间表** 可能是更好的方式。注意这里说到的额外的属性是用来描述 class 之间的关系的，而不是任何单一的 class 的。

我们建议你在任何一个 class 的对象数量超出 100 的时候考虑使用中间表。

### 序列化和反序列化

在实际的开发中，把 `LCObject` 当作参数传递的时候，会涉及到复杂对象的拷贝的问题，因此 `LCObject` 也提供了序列化和反序列化的方法。

序列化：


```java
LCObject todo = new LCObject("Todo"); // 构建对象
todo.put("title", "马拉松报名"); // 设置名称
todo.put("priority", 2); // 设置优先级
todo.put("owner", LCUser.getCurrentUser()); // 这里就是一个 Pointer 类型，指向当前登录的用户
String serializedString = todo.toString();
```

反序列化：


```java
LCObject deserializedObject = LCObject.parseLCObject(serializedString);
deserializedObject.save(); // 保存到服务端
```

## 查询

我们已经了解到如何从云端获取单个 `LCObject`，但你可能还会有一次性获取多个符合特定条件的 `LCObject` 的需求，这时候就需要用到 `LCQuery` 了。

### 基础查询

执行一次基础查询通常包括这些步骤：

1. 构建 `LCQuery`；
2. 向其添加查询条件；
3. 执行查询并获取包含满足条件的对象的数组。

下面的代码获取所有 `lastName` 为 `Smith` 的 `Student`：

```java
LCQuery<LCObject> query = new LCQuery<>("Student");
query.whereEqualTo("lastName", "Smith");
query.findInBackground().subscribe(new Observer<List<LCObject>>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(List<LCObject> students) {
        // students 是包含满足条件的 Student 对象的数组
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

### 查询条件

可以给 `LCObject` 添加不同的条件来改变获取到的结果。

下面的代码查询所有 `firstName` 不为 `Jack` 的对象：

```java
query.whereNotEqualTo("firstName", "Jack");
```

对于能够排序的属性（比如数字、字符串），可以进行比较查询：

```java
// 限制 age < 18
query.whereLessThan("age", 18);

// 限制 age <= 18
query.whereLessThanOrEqualTo("age", 18);

// 限制 age > 18
query.whereGreaterThan("age", 18);

// 限制 age >= 18
query.whereGreaterThanOrEqualTo("age", 18);
```

可以在同一个查询中设置多个条件，这样可以获取满足所有条件的结果。可以理解为所有的条件是 `AND` 的关系：

```java
query.whereEqualTo("firstName", "Jack");
query.whereGreaterThan("age", 18);
```

可以通过指定 `limit` 限制返回结果的数量（默认为 `100`）：

```java
// 最多获取 10 条结果
query.limit(10);
```

由于性能原因，`limit` 最大只能设为 `1000`。即使将其设为大于 `1000` 的数，云端也只会返回 1,000 条结果。

如果只需要一条结果，可以直接用 `getFirstInBackground`：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.whereEqualTo("priority", 2);
query.getFirstInBackground().subscribe(new Observer<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject todo) {
        // todo 是第一个满足条件的 Todo 对象
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

可以通过设置 `skip` 来跳过一定数量的结果：

```java
// 跳过前 20 条结果
query.skip(20);
```

把 `skip` 和 `limit` 结合起来，就能实现翻页功能：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.whereEqualTo("priority", 2);
query.limit(10);
query.skip(20);
```

需要注意的是，`skip` 的值越高，查询所需的时间就越长。作为替代方案，可以通过设置 `createdAt` 或 `updatedAt` 的范围来实现更高效的翻页，因为它们都自带索引。
同理，也可以通过设置自增字段的范围来实现翻页。

对于能够排序的属性，可以指定结果的排序规则：

```java
// 按 createdAt 升序排列
query.orderByAscending("createdAt");

// 按 createdAt 降序排列
query.orderByDescending("createdAt");
```

还可以为同一个查询添加多个排序规则；

```java
query.addAscendingOrder("priority");
query.addDescendingOrder("createdAt");
```

下面的代码可用于查找包含或不包含某一属性的对象：

```java
// 查找包含 "images" 的对象
query.whereExists("images");

// 查找不包含 "images" 的对象
query.whereDoesNotExist("images");
```

可以通过 `selectKeys` 指定需要返回的属性。下面的代码只获取每个对象的 `title` 和 `content`（包括内置属性 `objectId`、`createdAt` 和 `updatedAt`）：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.selectKeys(Arrays.asList("title", "content"));
query.getFirstInBackground().subscribe(new Observer<LCObject>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCObject todo) {
        String title = todo.getString("title"); // √
        String content = todo.getString("content"); // √
        String notes = todo.getString("notes"); // 会报错
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

`selectKeys`
支持点号（`author.firstName`），详见[《点号使用指南》](https://leancloud.cn/docs/dot-notation.html)。
另外，字段名前添加减号前缀表示反向选择，例如 `-author` 表示不返回 `author` 字段。
反向选择同样适用于内置字段，比如 `-objectId`，也可以和点号组合使用，比如 `-pubUser.createdAt`。

对于未获取的属性，可以通过对结果中的对象进行 `fetchInBackground` 操作来获取。参见 [同步对象](#同步对象)。
### 字符串查询

可以用 `whereStartsWith` 来查找某一属性值以特定字符串开头的对象。和 SQL 中的 `LIKE` 一样，你可以利用索引带来的优势：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
// 相当于 SQL 中的 title LIKE 'lunch%'
query.whereStartsWith("title", "lunch");
```


可以用 `whereContains` 来查找某一属性值包含特定字符串的对象：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
// 相当于 SQL 中的 title LIKE '%lunch%'
query.whereContains("title", "lunch");
```


和 `whereStartsWith` 不同，`whereContains` 无法利用索引，因此不建议用于大型数据集。

注意 `whereStartsWith` 和 `whereContains` 都是 **区分大小写** 的，所以上述查询会忽略 `Lunch`、`LUNCH` 等字符串。

如果想查找某一属性值不包含特定字符串的对象，可以使用 `whereMatches` 进行基于正则表达式的查询：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
// "title" 不包含 "ticket"（不区分大小写）
query.whereMatches("title", "^((?!ticket).)*$", "i");
```

不过我们并不推荐大量使用这类查询，尤其是对于包含超过 100,000 个对象的 class，
因为这类查询无法利用索引，实际操作中云端会遍历所有对象来获取结果。如果有进行全文搜索的需求，可以使用全文搜索服务。

使用查询时如果遇到性能问题，可参阅 [查询性能优化](#查询性能优化)。

### 数组查询

下面的代码查找所有数组属性 `tags` 包含 `工作` 的对象：

```java
query.whereEqualTo("tags", "工作");
```

下面的代码查询数组属性长度为 3 （正好包含 3 个标签）的对象：

```java
query.whereSizeEqual("tags", 3);
```

下面的代码查找所有数组属性 `tags` **同时包含** `工作`、`销售` 和 `会议` 的对象：

```java
query.whereContainsAll("tags", Arrays.asList("工作", "销售", "会议"));
```

如需获取某一属性值包含一列值中任意一个值的对象，可以直接用 `whereContainedIn` 而无需执行多次查询。下面的代码构建的查询会查找所有 `priority` 为 `1` **或** `2` 的 todo 对象：

```java
// 单个查询
LCQuery<LCObject> priorityOneOrTwo = new LCQuery<>("Todo");
priorityOneOrTwo.whereContainedIn("priority", Arrays.asList(1, 2));
// 这样就可以了 :)

// ---------------
//       vs.
// ---------------

// 多个查询
final LCQuery<LCObject> priorityOne = new LCQuery<>("Todo");
priorityOne.whereEqualTo("priority", 1);

final LCQuery<LCObject> priorityTwo = new LCQuery<>("Todo");
priorityTwo.whereEqualTo("priority", 2);

LCQuery<LCObject> priorityOneOrTwo = LCQuery.or(Arrays.asList(priorityOne, priorityTwo));
// 好像有些繁琐 :(
```

反过来，还可以用 `whereNotContainedIn` 来获取某一属性值不包含一列值中任何一个的对象。

### 关系查询

查询关联数据有很多种方式，常见的一种是查询某一属性值为特定 `LCObject` 的对象，这时可以像其他查询一样直接用 `whereEqualTo`。比如说，如果每一条博客评论 `Comment` 都有一个 `post` 属性用来存放原文 `Post`，则可以用下面的方法获取所有与某一 `Post` 相关联的评论：

```java
LCObject post = LCObject.createWithoutData("Post", "57328ca079bc44005c2472d0");
LCQuery<LCObject> query = new LCQuery<>("Comment");
query.whereEqualTo("post", post);
query.findInBackground().subscribe(new Observer<List<LCObject>>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(List<LCObject> comments) {
        // comments 包含与 post 相关联的评论
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

如需获取某一属性值为另一查询结果中任一 `LCObject` 的对象，可以用 `whereMatchesQuery`。下面的代码构建的查询可以找到所有包含图片的博客文章的评论：

```java
LCQuery<LCObject> innerQuery = new LCQuery<>("Post");
innerQuery.whereExists("image");

LCQuery<LCObject> query = new LCQuery<>("Comment");
query.whereMatchesQuery("post", innerQuery);
```

如需获取某一属性值不是另一查询结果中任一 `LCObject` 的对象，则使用 `whereDoesNotMatchQuery`。

有时候可能需要获取来自另一个 class 的数据而不想进行额外的查询，此时可以在同一个查询上使用 `include`。下面的代码查找最新发布的 10 条评论，并包含各自对应的博客文章：

```java
LCQuery<LCObject> query = new LCQuery<>("Comment");

// 获取最新发布的
query.orderByDescending("createdAt");

// 只获取 10 条
query.limit(10);

// 同时包含博客文章
query.include("post");

query.findInBackground().subscribe(new Observer<List<LCObject>>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(List<LCObject> comments) {
        // comments 包含最新发布的 10 条评论，包含各自对应的博客文章
        for (LCObject comment : comments) {
            // 该操作无需网络连接
            LCObject post = comment.getLCObject("post");
        }
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

可以用 dot 符号（`.`）来获取多级关系，例如 `post.author`，详见[《点号使用指南》](https://leancloud.cn/docs/dot-notation.html)的《在查询对象时使用点号》一节。

可以在同一查询上应用多次 `include` 以包含多个属性。通过这种方法获取到的对象同样接受 `getFirst` 等 `LCQuery` 辅助方法。

通过 `include` 进行多级查询的方式不适用于数组属性内部的 `LCObject`，只能包含到数组本身。

#### 关系查询的注意事项

云端使用的并非关系型数据库，无法做到真正的联表查询，所以实际的处理方式是：先执行内嵌／子查询（和普通查询一样，`limit` 默认为 `100`，最大 `1000`），然后将子查询的结果填入主查询的对应位置，再执行主查询。如果子查询匹配到的记录数量超出 `limit`，且主查询有其他查询条件，那么可能会出现没有结果或结果不全的情况，因为只有 `limit` 数量以内的结果会被填入主查询。

我们建议采用以下方案进行改进：

- 确保子查询的结果在 100 条以下，如果在 100 至 1,000 条之间的话请将子查询的 `limit` 设为 `1000`。
- 将需要查询的字段冗余到主查询所在的表上。
- 进行多次查询，每次在子查询上设置不同的 `skip` 值来遍历所有记录（注意 `skip` 的值较大时可能会引发性能问题，因此不是很推荐）。

### 统计总数量

如果只需知道有多少对象匹配查询条件而无需获取对象本身，可使用 `count` 来代替 `findInBackground`。比如说，查询有多少个已完成的 todo：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.whereEqualTo("isComplete", true);
query.countInBackground().subscribe(new Observer<Integer>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(Integer count) {
        System.out.println(count + " 个 todo 已完成。");
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

### 组合查询

组合查询就是把诸多查询条件用一定逻辑合并到一起（`OR` 或 `AND`）再交给云端去查询。

组合查询不支持在子查询中包含 `GeoPoint` 或其他非过滤性的限制（例如 `near`、`withinGeoBox`、`limit`、`skip`、`ascending`、`descending`、`include`）。

#### OR 查询

OR 操作表示多个查询条件符合其中任意一个即可。 例如，查询优先级大于等于 `3` 或者已经完成了的 todo：

```java
final LCQuery<LCObject> priorityQuery = new LCQuery<>("Todo");
priorityQuery.whereGreaterThanOrEqualTo("priority", 3);

final LCQuery<LCObject> isCompleteQuery = new LCQuery<>("Todo");
isCompleteQuery.whereEqualTo("isComplete", true);

LCQuery<LCObject> query = LCQuery.or(Arrays.asList(priorityQuery, isCompleteQuery));
```

使用 OR 查询时，子查询中不能包含 `GeoPoint` 相关的查询。

#### AND 查询

使用 AND 查询的效果等同于往 `LCQuery` 添加多个条件。下面的代码构建的查询会查找创建时间在 `2016-11-13` 和 `2016-12-02` 之间的 todo：

```java
Date getDateWithDateString(String dateString) {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    Date date = dateFormat.parse(dateString);
    return date;
}

final LCQuery<LCObject> startDateQuery = new LCQuery<>("Todo");
startDateQuery.whereGreaterThanOrEqualTo("createdAt", getDateWithDateString("2016-11-13"));

final LCQuery<LCObject> endDateQuery = new LCQuery<>("Todo");
endDateQuery.whereLessThan("createdAt", getDateWithDateString("2016-12-03"));

LCQuery<LCObject> query = LCQuery.and(Arrays.asList(startDateQuery, endDateQuery));
```


单独使用 AND 查询跟使用基础查询相比并没有什么不同，不过当查询条件中包含不止一个 OR 查询时，就必须使用 AND 查询：


```java
Date getDateWithDateString(String dateString) {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    Date date = dateFormat.parse(dateString);
    return date;
}

final LCQuery<LCObject> createdAtQuery = new LCQuery<>("Todo");
createdAtQuery.whereGreaterThanOrEqualTo("createdAt", getDateWithDateString("2018-04-30"));
createdAtQuery.whereLessThan("createdAt", getDateWithDateString("2018-05-01"));

final LCQuery<LCObject> locationQuery = new LCQuery<>("Todo");
locationQuery.whereDoesNotExist("location");

final LCQuery<LCObject> priority2Query = new LCQuery<>("Todo");
priority2Query.whereEqualTo("priority", 2);

final LCQuery<LCObject> priority3Query = new LCQuery<>("Todo");
priority3Query.whereEqualTo("priority", 3);

LCQuery<LCObject> priorityQuery = LCQuery.or(Arrays.asList(priority2Query, priority3Query));
LCQuery<LCObject> timeLocationQuery = LCQuery.or(Arrays.asList(locationQuery, createdAtQuery));
LCQuery<LCObject> query = LCQuery.and(Arrays.asList(priorityQuery, timeLocationQuery));
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

进入 **控制台 > 存储 > 设置**，在 **其他** 里面勾选 **启用 LiveQuery**即可。确保即时通信模块已被添加到 `AndroidManifest.xml`：


```xml
<service android:name="cn.leancloud.push.PushService"/>
<receiver android:name="cn.leancloud.push.LCBroadcastReceiver">
  <intent-filter>
    <action android:name="android.intent.action.BOOT_COMPLETED"/>
    <action android:name="android.intent.action.USER_PRESENT"/>
    <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
  </intent-filter>
</receiver>
```

可以在 [SDK 安装与初始化](#SDK-安装与初始化) 中找到完整设置方法。

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


```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.whereEqualTo("isComplete", true);
LCLiveQuery liveQuery = LCLiveQuery.initWithQuery(query);
liveQuery.subscribeInBackground(new LCLiveQuerySubscribeCallback() {
    @Override
    public void done(LCException e) {
        if (e == null) {
            // 订阅成功
        }
    }
});
```

LiveQuery 不支持内嵌查询，也不支持返回指定属性。

订阅成功后，就可以接收到和 `LCObject` 相关的更新了。假如在另一个客户端上创建了一个 `Todo` 对象，对象的 `title` 设为 `更新作品集`，那么下面的代码可以获取到这个新的 `Todo`：


```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.whereEqualTo("isComplete", true);
LCLiveQuery liveQuery = LCLiveQuery.initWithQuery(query);
liveQuery.setEventHandler(new LCLiveQueryEventHandler() {
    @Override
    public void onObjectCreated(LCObject newTodo) {
        System.out.println(newTodo.getString("title")); // 更新作品集
    }
});
liveQuery.subscribeInBackground(new LCLiveQuerySubscribeCallback() {
    @Override
    public void done(LCException e) {
        if (e == null) {
            // 订阅成功
        }
    }
});
```

此时如果有人把 `Todo` 的 `content` 改为 `把我最近画的插画放上去`，那么下面的代码可以获取到本次更新：


```java
liveQuery.setEventHandler(new LCLiveQueryEventHandler() {
    @Override
    public void onObjectUpdated(LCObject updatedTodo, List<String> updatedKeys) {
        System.out.println(updatedTodo.getString("content")); // 把我最近画的插画放上去
    }
});
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


```java
liveQuery.setEventHandler(new LCLiveQueryEventHandler() {
    @Override
    public void onObjectCreated(LCObject object) {
        System.out.println("对象被创建。");
    }
});
```

#### `update` 事件

当有满足 `LCQuery` 查询条件的 `LCObject` 被更新时，`update` 事件会被触发。下面的 `object` 就是有更新的 `LCObject`：


```java
liveQuery.setEventHandler(new LCLiveQueryEventHandler() {
    @Override
    public void onObjectUpdated(LCObject object, List<String> updatedKeys) {
        System.out.println("对象被更新。");
    }
});
```

#### `enter` 事件

当一个已存在的、原本不符合 `LCQuery` 查询条件的 `LCObject` 发生更新，且更新后符合查询条件，`enter` 事件会被触发。下面的 `object` 就是进入 `LCQuery` 的 `LCObject`，其内容为该对象最新的值：


```java
liveQuery.setEventHandler(new LCLiveQueryEventHandler() {
    @Override
    public void onObjectEnter(LCObject object, List<String> updatedKeys) {
        System.out.println("对象进入。");
    }
});
```

注意区分 `create` 和 `enter` 的不同行为。如果一个对象已经存在，在更新之前不符合查询条件，而在更新之后符合查询条件，那么 `enter` 事件会被触发。如果一个对象原本不存在，后来被构建了出来，那么 `create` 事件会被触发。

#### `leave` 事件

当一个已存在的、原本符合 `LCQuery` 查询条件的 `LCObject` 发生更新，且更新后不符合查询条件，`leave` 事件会被触发。下面的 `object` 就是离开 `LCQuery` 的 `LCObject`，其内容为该对象最新的值：


```java
liveQuery.setEventHandler(new LCLiveQueryEventHandler() {
    @Override
    public void onObjectLeave(LCObject object, List<String> updatedKeys) {
        System.out.println("对象离开。");
    }
});
```

#### `delete` 事件

当一个已存在的、原本符合 `LCQuery` 查询条件的 `LCObject` 被删除，`delete` 事件会被触发。下面的 `object` 就是被删除的 `LCObject` 的 `objectId`：


```java
liveQuery.setEventHandler(new LCLiveQueryEventHandler() {
    @Override
    public void onObjectDeleted(String object) {
        System.out.println("对象被删除。");
    }
});
```


### 取消订阅

如果不再需要接收有关 `LCQuery` 的更新，可以取消订阅。


```java
liveQuery.unsubscribeInBackground(new LCLiveQuerySubscribeCallback() {
    @Override
    public void done(LCException e) {
        if (e == null) {
            // 成功取消订阅
        }
    }
});
```

### 断开连接

断开连接有几种情况：

1. 网络异常或者网络切换，非预期性断开。
2. 退出应用、关机或者打开飞行模式等，用户在应用外的操作导致断开。

如上几种情况开发者无需做额外的操作，只要切回应用，SDK 会自动重新订阅，数据变更会继续推送到客户端。

而另外一种极端情况——**当用户在移动端使用手机的进程管理工具，杀死了进程或者直接关闭了网页的情况下**，SDK 无法自动重新订阅，此时需要开发者根据实际情况实现重新订阅。

### 网络状态响应

可以调用 `setConnectionHandler` 静态方法监听 LiveQuery 连接的建立、断开、异常：

```java
LCLiveQuery.setConnectionHandler(new LCLiveQueryConnectionHandler() {
    @Override
    public void onConnectionOpen() {
        System.out.println("============ LiveQuery Connection opened ============");
    }

    @Override
    public void onConnectionClose() {
        System.out.println("============ LiveQuery Connection closed ============");
    }

    @Override
    public void onConnectionError(int code, String reason) {
        System.out.println("============ LiveQuery Connection error. code:" + code
            + ", reason:" + reason + " ============");
    }
});
```

### LiveQuery 的注意事项

因为 LiveQuery 的实时性，很多用户会陷入一个误区，试着用 LiveQuery 来实现一个简单的聊天功能。
我们不建议这样做，因为使用 LiveQuery 构建聊天服务会承担额外的存储成本，产生的费用会增加，后期维护的难度非常大（聊天记录、对话维护之类的代码会很混乱），并且云服务已经提供了即时通讯的服务。
LiveQuery 的核心还是提供一个针对查询的推拉结合的用法，脱离设计初衷容易造成前端的模块混乱。


如果你的应用只使用 LiveQuery （不使用即时通讯和其他推送服务），那么可以在初始化 SDK 时使用 `PushService` 的静态方法 `startIfRequired` 来创建 WebSocket 连接：

```java
PushService.startIfRequired(android.content.Context context);
```

## 文件

有时候应用需要存储尺寸较大或结构较为复杂的数据，这类数据不适合用 `LCObject` 保存，此时文件对象 `LCFile` 便成为了更好的选择。文件对象最常见的用途是保存图片，不过也可以用来保存文档、视频、音乐等其他二进制数据。

### 构建文件


可以通过字符串构建文件：

```java
// resume.txt 是文件名
LCFile file = new LCFile("resume.txt", "LeanCloud".getBytes());
```

除此之外，还可以通过 URL 构建文件：

```java
LCFile file = new LCFile(
    "logo.png",
    "https://leancloud.cn/assets/imgs/press/Logo%20-%20Blue%20Padding.a60eb2fa.png",
    new HashMap<String, Object>()
);
```

通过 URL 构建文件时，SDK 并不会将原本的文件转储到云端，而是会将文件的物理地址存储为字符串，这样也就不会产生任何文件上传流量。使用其他方式构建的文件会被保存在云端。

云端会根据文件扩展名自动检测文件类型。如果需要的话，也可以手动指定 `Content-Type`（一般称为 MIME 类型）：


```java
Map<String, Object> meta = new HashMap<String, Object>();
meta.put("mime_type", "application/json");
LCFile file = new LCFile("resume.txt", "LeanCloud".getBytes(), meta);
```

与前面提到的方式相比，一个更常见的文件构建方式是从本地路径上传。


```java
LCFile file = LCFile.withAbsoluteLocalPath("avatar.jpg", "/tmp/avatar.jpg");
```

这里上传的文件名字叫做 `avatar.jpg`。需要注意：

- 每个文件会被分配到一个独一无二的 `objectId`，所以在一个应用内是允许多个文件重名的。
- 文件必须有扩展名才能被云端正确地识别出类型。比如说要用 `LCFile` 保存一个 PNG 格式的图像，那么扩展名应为 `.png`。
- 如果文件没有扩展名，且没有手动指定类型，那么云服务将默认使用 `application/octet-stream`。

### 保存文件

将文件保存到云端后，便可获得一个永久指向该文件的 URL：

```java
file.saveInBackground().subscribe(new Observer<LCFile>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCFile file) {
        System.out.println("文件保存完成。URL: " + file.getUrl());
    }
    public void onError(Throwable throwable) {
        // 保存失败，可能是文件无法被读取，或者上传过程中出现问题
    }
    public void onComplete() {}
});
```

文件上传后，可以在 `_File` class 中找到。已上传的文件无法再被修改。如果需要修改文件，只能重新上传修改过的文件并取得新的 `objectId` 和 URL。



```java
file.saveInBackground(true).subscribe(new Observer<LCFile>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCFile file) {
        System.out.println("文件保存完成。objectId：" + file.getObjectId());
    }
    public void onError(Throwable throwable) {
        // 保存失败，可能是文件无法被读取，或者上传过程中出现问题
    }
    public void onComplete() {}
});
```

已经保存到云端的文件可以关联到 `LCObject`：

```java
LCObject todo = new LCObject("Todo");
todo.put("title", "买蛋糕");
// attachments 是一个 Array 属性
todo.add("attachments", file);
todo.save();
```

也可以通过构建 `LCQuery` 进行[查询](#查询)：

```java
LCQuery<LCObject> query = new LCQuery<>("_File");
```


需要注意的是，内部文件（上传到文件服务的文件）的 `url` 字段是由云端动态生成的，其中涉及切换自定义域名的相关处理逻辑。
因此，通过 url 字段查询文件仅适用于外部文件（直接保存外部 URL 到 `_File` 表创建的文件），内部文件请改用 key 字段（URL 中的路径）查询。

注意，如果文件被保存到了 `LCObject` 的一个数组属性中，那么在查询 `LCObject` 时如果需要包含文件，则要用到 `LCQuery` 的 `include` 方法。比如说，在获取所有标题为 `买蛋糕` 的 todo 的同时获取附件中的文件：

```java
// 获取同一标题且包含附件的 todo
LCQuery<LCObject> query = new LCQuery<>("Todo");
query.whereEqualTo("title", "买蛋糕");
query.whereExists("attachments");

// 同时获取附件中的文件
query.include("attachments");

query.findInBackground().subscribe(new Observer<List<LCObject>>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(List<LCObject> todos) {
        for (LCObject todo : todos) {
            // 获取每个 todo 的 attachments 数组
            // {# TODO #}
        }
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```


### 上传进度监听

上传过程中可以实时向用户展示进度：

```java
file.saveInBackground(new ProgressCallback() {
    @Override
    public void done(Integer percent) {
        // percent 是一个 0 到 100 之间的整数，表示上传进度
    }
});
```

### 文件元数据

上传文件时，可以用 `metaData` 添加额外的属性。文件一旦保存，`metaData` 便不可再修改。

```java
// 设置元数据
file.addMetaData("author", "LeanCloud");
file.saveInBackground().subscribe(new Observer<LCFile>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(LCFile file) {
        // 获取 author 属性
        String author = (String) file.getMetaData("author");
        // 获取文件名
        String fileName = file.getName();
        // 获取大小（不适用于通过 base64 编码的字符串或者 URL 保存的文件）
        int size = file.getSize();
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

### 图像缩略图

成功保存图像后，除了可以获取指向该文件的 URL 外，还可以获取图像的缩略图 URL，并且可以指定缩略图的宽度和高度：


```java
LCFile file = new LCFile("test.jpg", "文件-url", new HashMap<String, Object>());
file.getThumbnailUrl(true, 100, 100);
```


图片最大不超过 **20 MB** 才可以获取缩略图。

国际版不支持图片缩略图。

### 下载文件

可以调用 LCFile 的 `getDataInBackground` 或 `getDataStreamInBackground` 方法下载文件：

```java
avFile.getDataInBackground().subscribe(new Observer<byte[]>() {
    @Override
    public void onSubscribe(Disposable d) {}

    @Override
    public void onNext(byte[] bytes) {
        Log.d("LCFile", "file data length: " + bytes.length);
    }

    @Override
    public void onError(Throwable e) {
        Log.d("LCFile", "failed to get data. cause: " + e.getMessage());
    }

    @Override
    public void onComplete() {}
});

avFile.getDataStreamInBackground().subscribe(new Observer<InputStream>() {
    @Override
    public void onSubscribe(Disposable d) {}

    @Override
    public void onNext(InputStream inputStream) {
        try {
            byte[] buffer = new byte[102400];
            int read = inputStream.read(buffer);
            Log.d("LCFile", "file data length: " + read);
            inputStream.close();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    @Override
    public void onError(Throwable e) {
        Log.d("LCFile", "failed to get data. cause: " + e.getMessage());
    }

    @Override
    public void onComplete() {}
});
```

除了使用 SDK 提供的方法外，也可以获取 LCFile 的 url 后调用标准库和第三方库下载文件。


### 删除文件

下面的代码从云端删除一个文件：

```java
LCObject file = LCObject.createWithoutData("_File", "552e0a27e4b0643b709e891e");
file.delete()
```

默认情况下，文件的删除权限是关闭的，需要进入 **云服务控制台 > 数据存储 > 结构化数据 > `_File`**，选择 **其他** > **权限设置** > **`delete`** 来开启。

## GeoPoint

云服务允许你通过将 `LCGeoPoint` 关联到 `LCObject` 的方式存储折射真实世界地理位置的经纬坐标，这样做可以让你查询包含一个点附近的坐标的对象。常见的使用场景有「查找附近的用户」和「查找附近的地点」。

要构建一个包含地理位置的对象，首先要构建一个地理位置。下面的代码构建了一个 `LCGeoPoint` 并将其纬度（`latitude`）设为 `39.9`，经度（`longitude`）设为 `116.4`：

```java
LCGeoPoint point = new LCGeoPoint(39.9, 116.4);
```


现在可以将这个地理位置存储为一个对象的属性：

```java
todo.put("location", point);
```

### 地理位置查询

给定一些含有地理位置的对象，可以从中找出离某一点最近的几个，或者处于某一范围内的几个。要执行这样的查询，可以向普通的 `LCQuery` 添加 `whereNear` 条件。下面的代码查找 `location` 属性值离某一点最近的 `Todo` 对象：

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
LCGeoPoint point = new LCGeoPoint(39.9, 116.4);
query.whereNear("location", point);

// 限制为 10 条结果
query.limit(10);
query.findInBackground().subscribe(new Observer<List<LCObject>>() {
    public void onSubscribe(Disposable disposable) {}
    public void onNext(List<LCObject> todos) {
        // todos 是包含满足条件的 Todo 对象的数组
    }
    public void onError(Throwable throwable) {}
    public void onComplete() {}
});
```

像 `orderByAscending` 和 `orderByDescending` 这样额外的排序条件会获得比默认的距离排序更高的优先级。

若要限制结果和给定地点之间的距离，可以参考 API 文档中的 `whereWithinKilometers`、`whereWithinMiles` 和 `whereWithinRadians` 参数。

若要查询在某一矩形范围内的对象，可以用 `whereWithinGeoBox`：

![withinGeoBox](/img/geopoint-withingeobox.svg)

```java
LCQuery<LCObject> query = new LCQuery<>("Todo");
LCGeoPoint southwest = new LCGeoPoint(30, 115);
LCGeoPoint northeast = new LCGeoPoint(40, 118);
query.whereWithinGeoBox("location", southwest, northeast);
```

### GeoPoint 的注意事项

GeoPoint 的经纬度的类型是数字，且经度需在 -180.0 到 180.0 之间，纬度需在 -90.0 到 90.0 之间。
另外，每个对象最多只能有一个类型为 GeoPoint 的属性。

## 用户

请参阅[内建账户指南](/sdk/authentication/guide/)。

## 角色

随着用户量的增长，你可能会发现相比于为每一名用户单独设置权限，将预先设定好的权限直接分配给一部分用户是更好的选择。为了迎合这种需求，云服务支持基于角色的权限管理。请参阅[《ACL 权限管理开发指南》](https://leancloud.cn/docs/acl-guide.html)。

## 子类化

子类化推荐给进阶的开发者在进行代码重构的时候做参考。你可以用 `LCObject#get` 访问任意字段；你也可以使用子类化的属性来封装获取字段的方法，增强编码体验。
子类化有很多优势，包括减少代码的编写量，具有更好的扩展性，和支持自动补全等等。

### 实现

要实现子类化，需要下面 4 个步骤：

1. 首先声明一个子类继承自 `LCObject`；
2. 添加 `@LCClassName` 注解。它的值必须是一个字符串，也就是你过去传入 `LCObject` 构造函数的类名。这样以来，后续就不需要再在代码中出现这个字符串类名；
3. 确保你的子类有一个 public 的默认（参数个数为 0）的构造函数。切记不要在构造函数里修改任何 `LCObject` 的字段；
4. 在你的应用初始化的地方，在调用 `LeanCloud.initialize()` 之前注册子类 `LCObject.registerSubclass(YourClass.class)`。

下面是实现 `Student` 子类化的例子:

``` java
// Student.java
import cn.leancloud.LCClassName;
import cn.leancloud.LCObject;

@LCClassName("Student")
public class Student extends LCObject {
  // 添加访问器
  public String getContent() {
    return getString("content");
  }
  // 添加修改器
  public void setContent(String value) {
    put("content", value);
  }
  // 添加自定义方法
  public void handleReport() {
    // 处理用户举报，当达到某个条数的时候，自动打上屏蔽标志
    increment("report", 1);
    if (getReport() > 50) {
      setSpam(true);
    }
  }
}

// App.java
import cn.leancloud.LeanCloud;

public class MyLeanCloudApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        LCObject.registerSubclass(Student.class);
        LeanCloud.initialize(this, "your-client-id", "your-client-token", "https://please-replace-with-your-customized.domain.com");
    }
}
```

### 初始化子类

你可以使用你自定义的构造函数来创建你的子类对象。你的子类必须定义一个公开的默认构造函数，并且不修改任何父类 LCObject 中的字段，这个默认构造函数将会被 SDK 使用来创建子类的强类型的对象。

要创建一个到现有对象的引用，可以使用 `LCObject.createWithoutData()`:

```java
Student postReference = LCObject.createWithoutData(Student.class, student.getObjectId());
```

### 子类的序列化与反序列化

如果希望 LCObject 子类也支持 Parcelable，则需要至少满足以下几个要求：
1. 确保子类有一个 public 并且参数为 Parcel 的构造函数，并且在内部调用父类的该构造函数。
2. 内部需要有一个静态变量 CREATOR 实现 `Parcelable.Creator`。

```java
// Stduent.java
@LCClassName("Student")
public class Student extends LCObject {
  public Student(){
    super();
  }

  public Student(Parcel in){
    super(in);
  }
  //此处为我们的默认实现，当然你也可以自行实现
  public static final Creator CREATOR = AVObjectCreator.instance;
}
```

### 查询子类

你可以通过 `LCObject.getQuery()` 或者 `LCQuery.getQuery` 的静态方法获取特定的子类的查询对象。下面的例子就查询了用户发表的所有微博列表：

```java
LCQuery<Student> query = LCObject.getQuery(Student.class);
query.whereEqualTo("pubUser", LCUser.getCurrentUser().getUsername());
List<Student> results = query.find();
for (Student a : results) {
    // ...
}
```

### LCUser 的子类化

LCUser 作为 AVObject 的子类，同样允许子类化，你可以定义自己的 User 对象，不过比起 LCObject 子类化会更简单一些，只要继承 LCUser 就可以了：

```java
import cn.leancloud.LCObject;
import cn.leancloud.LCUser;

public class MyUser extends LCUser {
    public void setNickName(String name) {
        this.put("nickName", name);
    }

    public String getNickName() {
        return this.getString("nickName");
    }
}
```

不需要添加 @LCClassname 注解，所有 LCUser 的子类的类名都是内建的 `_User`。同样也不需要注册 MyUser。

当用户子类化 LCUser 后，如果希望以后查询 LCUser 所得到的对象会自动转化为用户子类化的对象，则需要在调用 LeanCloud.initialize() 之前添加：

```java
LCUser.alwaysUseSubUserClass(subUser.class);
```

注册跟普通的 LCUser 对象没有什么不同，但是登录如果希望返回自定义的子类，必须这样：

```java
MyUser cloudUser = LCUser.logIn(username, password, MyUser.class);
```

## 全文搜索

全文搜索是一个针对应用数据进行全局搜索的接口，它基于搜索引擎构建，提供更强大的搜索功能。要深入了解其用法和阅读示例代码，请阅[全文搜索指南](/sdk/storage/guide/fulltext-search/)。