---
title: 工具JsonPath
layout: post
tags:
  - java
---

今天在看一个开源项目源码的时候，看到两个好东西，备份一下以后应该用得着

## JsonPath
类似[XPath](http://www.w3school.com.cn/xpath/)，关于这个工具类的使用场景，举个例子，一个 json api 调用返回了一个复杂的对象，而我们只需要其中的一些信息，直接的做法是将字符串解析成 JSONObject 对象，然后根据对应的 key 一层一层剥开

{%highlight json%}
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            // ...省略...
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10
}
{%endhighlight%}

假设我们只需要第一本书的价格，直接的做法

{%highlight java%}
String json = IOUtils.toString(getClass().getResourceAsStream("json.txt"));
JSONObject j = JSONObject.fromObject(json);
JSONObject jStore = j.getJSONObject("store");
JSONArray jBooks = jStore.getJSONArray("book");
System.out.println(jBooks.getJSONObject(0).get("price"));
{%endhighlight%}

用 JsonPath 的方式，只需要一行代码（完整语法参考[https://github.com/jayway/JsonPath](https://github.com/jayway/JsonPath))

{%highlight java%}
double price = JsonPath.read(json, "$.store.book[0].price");
System.out.println(price);

List<String> books = JsonPath.read(json, "$.store.book[?(@.title=='Moby Dick')]");
System.out.println(books);
{%endhighlight%}

## jsonschema2pojo
* 工具在线地址 [http://www.jsonschema2pojo.org/](http://www.jsonschema2pojo.org/)
* Github地址 [https://github.com/joelittlejohn/jsonschema2pojo](https://github.com/joelittlejohn/jsonschema2pojo)

这个工具是将 json 类型的数据结构转换成 Java pojo，比如将上一个例子的 json 数据丢进去，生成出来的 pojo 代码如下（省略了getter/setter）

{%highlight java%}

    -----------------------------------com.example.Bicycle.java-----------------------------------
    package com.example;

    public class Bicycle {
        public String color;
        public double price;
    }
    -----------------------------------com.example.Book.java-----------------------------------
    package com.example;

    public class Book {
        public String category;
        public String author;
        public String title;
        public double price;
        public String isbn;
    }
    -----------------------------------com.example.Store.java-----------------------------------
    package com.example;

    import java.util.ArrayList;
    import java.util.List;

    public class Store {
        public List<Book> book = new ArrayList<Book>();
        public Bicycle bicycle;
    }
    -----------------------------------com.example.Example.java-----------------------------------
    package com.example;

    public class Example {
        public Store store;
        public int expensive;
    }
{%endhighlight%}

### 参考资料
* [https://github.com/jayway/JsonPath](https://github.com/jayway/JsonPath)
* [http://www.baeldung.com/guide-to-jayway-jsonpath](http://www.baeldung.com/guide-to-jayway-jsonpath)
* [http://www.jsonschema2pojo.org/](http://www.jsonschema2pojo.org/)
* [https://github.com/joelittlejohn/jsonschema2pojo](https://github.com/joelittlejohn/jsonschema2pojo)








