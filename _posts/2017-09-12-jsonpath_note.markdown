layout: post
title:  "JsonPath基本功能使用"
date:   2017-09-12 15:03:25
categories: original



JsonPath 提供了强大的 json 解析功能。

项目文档中提供了 jsonpath 对 json 的基本解析功能。详情参考 https://github.com/json-path/JsonPath。

测试 demo 数据
``` json
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            },
            {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99
            },
            {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10
}
```

配置Option.AS_PATH_LIST参数，返回 匹配jsonpath的所有 json 节点。

``` java
Configuration conf = Configuration.builder().options(Option.AS_PATH_LIST).build();
List<String> pathList = JsonPath.using(conf).parse(jsonStr).read("$..title");

```
配置 MappingProvider，返回特定 PoJo 类型

``` java
Configuration configuration = Configuration.builder().jsonProvider(new GsonJsonProvider()).mappingProvider(new GsonMappingProvider()).build();
        List<String> titles = JsonPath.using(configuration).parse(jsonStr).read("$.store.book[*].title", new TypeRef<List<String>>(){});
```

配置Option.DEFAULT_PATH_LEAF_TO_NULL，找不到节点值时返回 null。避免抛出PathNotFoundException

``` java
Configuration configuration = Configuration.defaultConfiguration().addOptions(Option.DEFAULT_PATH_LEAF_TO_NULL);
String author4 = JsonPath.using(configuration).parse(jsonStr).read("$.store.book[0].author1");
```
配置Option.ALWAYS_RETURN_LIST，返回list

``` java
        configuration = Configuration.defaultConfiguration().addOptions(Option.ALWAYS_RETURN_LIST);
        List<String> author0List = JsonPath.using(configuration).parse(jsonStr).read("$.store.book[0].author");
```

通过JsonPath新增和修改节点值。
set(jsonPath,value)方法用于修改 jsonpath 的 value 值。
put(jsonPath, key, value)方法在 jsonpath层次新增一个节点key，值为 value。
add(jsonPath, object)方法用于Array 类型的 jsonpath 新增一个item。object 为 item 的完整 jsonObject。

``` java
 Object json = new JsonParser().parse(jsonStr).getAsJsonObject();
        Configuration configuration =
                Configuration.builder().jsonProvider(new GsonJsonProvider()).mappingProvider(new GsonMappingProvider()).build();
        DocumentContext context = JsonPath.using(configuration).parse(json);
        context.set(JsonPath.compile("$.store.book[0].title"), "test_title");
        context.put(JsonPath.compile("$.store.bicycle"), "newNode", "newNode_value");
        context.add(JsonPath.compile("$.store.book"), new JsonParser().parse("{\"category\":\"test1\",\"title\":\"test_title_1\"}").getAsJsonObject());
```



