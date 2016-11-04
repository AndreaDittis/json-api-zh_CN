---
layout: page
title: "推荐规范"
---


本节介绍实现JSON API的推荐规范。这些推荐规范用于对基本JSON API规范的补充，以保证基本规范以外部分的一致性。

##命名<a href="#naming" id="naming" class="headerlink"></a>

用于键的URL安全命名的合法字符和推荐字符已经在格式说明部分定义。同样为了规范键命名方式，推荐以下(更严格的)规则：

* 键名称应该以字符"a-z"(U+0061 to U+007A)开始和结束.
* 键名称应该只包含字符“a-z” (U+0061 to U+007A)和 “0-9” (U+0030 to U+0039)，并且以减号连字符(U+002D HYPHEN-MINUS, “-“) 作为单词之间的分隔符.

##URL设计<a href="#url-design" id="url-design" class="headerlink"></a>

###相关文档<a href="#url-design-reference-document" id="url-design-reference-document" class="headerlink"></a>

在一个"reference document"中，每个资源都可通过一个特有路径寻址。考虑"reference document"中的所有资源对于设计一个API的URL结构很有帮助。资源在这个文档的top-level中以类型分组。每个资源在这些类型集合中以ID标识。每个资源中的属性和链接可以通过前文所述的资源对象结构寻址。

相关文档的概念可用于为资源和它们的关联设计URL。由于目标和规则不同，此处的相关文档与用于传输文档有少许不同之处。例如，群组在相关文档中以集合形式表现，因为群组的成员必须通过ID寻址；但是在传输文档中以数组形式表现，因为它是有序的。

###资源组URL<a href="#url-design-urls-for-resource-collection" id="url-design-urls-for-resource-collection" class="headerlink"></a>

推荐利用资源类型构成资源组的URL。

例如，一组`photos`类型的资源可以通过以下URL定义:

```javascript
/photos
```

###资源个体URL<a href="#url-design-urls-for-individual-resources" id="url-design-urls-for-individual-resources" class="headerlink"></a>

将资源组看做以资源ID标识的集合。每个资源个体的URL可以通过在资源组URL后附加资源ID构成。

例如，ID为`"1"`的照片可以通过以下URL定义：

```javascript
/photos/1
```

###关联URL和相关资源URL<a href="#url-design-relationship-urls-and-related-urls" id="url-design-relationship-urls-and-related-urls" class="headerlink"></a>

如基本规范所述，对于每个关联，可以有两个URL:

* "relationship URL": 表述关联本身的URL，用关联的`links`对象中的`self`键定义。客户端可以通过这个URL直接操作关联。例如，客户端可以通过这个URL利用`POST`请求移除`author`，而不需要删除`people`资源本省。

* “related resource URL” : 指向相关资源的URL，用关联的`links`对象中的`related`键定义。当通过这个URL获取数据，将会回复将相关资源对象作为主数据的响应。

推荐通过在资源URL后附加`/relationships/`和关联名称构成关联URL。

例如，一张照片的`comments`关联可以通过以下URL定义:

```javascript
/photos/1/relationships/comments
```

一张照片的`photographer`关联可以通过以下URL定义:

```javascript
/photos/1/relationships/photographer
```

推荐通过在资源URL后附加关联名称构成相关资源URL。

例如，一张照片的`comments`可以通过以下URL定义:

```javascript
/photos/1/comments
```

一张照片的`photographer`可以通过以下URL定义:

```javascript
/photos/1/photographer
```

由于这些URL表示关联中的资源，因此他们不能用于资源本省的`self`链接。当构成`self`链接时，资源个体URL推荐规范仍然使用。



## Filtering <a href="#filtering" id="filtering" class="headerlink"></a>

基础规范并未规定服务器支持的过滤规则。查询参数`filter`是用于任何查询策略的保留字。

如果服务器希望支持基于associations的资源集合的过滤,推荐允许查询参数将`filter`与association名称相结合。

比如,下面是一个对与一个post相关的所有评论的请求:

```javascript
GET /comments?filter[post]=1 HTTP/1.1
```
多个过滤值可以用逗号分隔开。比如:

```javascript
GET /comments?filter[post]=1,2 HTTP/1.1
```

一个请求可以包括多个过滤:

```javascript
GET /comments?filter[post]=1,2&filter[author]=12 HTTP/1.1
```

## Top-level, 资源层链接以及关联链接 <a href="#including-links" id="including-links" class="headerlink"></a>

基础规范并未规定包括带有资源响应的连接。然而,推荐将以下链接包括在响应文档里。


* **Top-level链接** 整个响应的自链接以及相关的分页链接.
* **资源层链接** 每个资源的自链接(如果资源是集合的一部分，此链接不同于top-level链接).
* **关联链接** 用于表述资源中的所有可用关联. 

比如,一个对评论集合的请求可以引起如下响应:

```javascript
GET /comments HTTP/1.1

{
  "data": [{
      "type": "comments",
      "id": "1",
      "attributes": {
          "text": "HATEOS are the thing!"
      },
      "links": {
          "self": "/comments/1"
      },
      "relationships": {
        "author": {
          "links": {
            "self": "/comments/1/relationships/author",
            "related": "/comments/1/author"
          }
        },
        "articles": {
          "links": {
            "self": "/comments/1/relationships/articles",
            "related": "/comments/1/articles"
          }
        }
      }
  }],
  "links": {
      "self": "/comments"
  }
}
```

## 支持缺少PATCH的客户端 <a href="#supporting-client-lacking-patch" id="supporting-client-lacking-patch" class="headerlink"></a>

一些客户端,如IE8,并不支持HTTP`PATCH`方法。如果API服务器希望支持这些客户端的,
建议当客户端报头包括`X-HTTP-Method-Override: PATCH`时,把`POST`请求当做`PATCH`请求。
这样,只需通过添加报头,缺少`PATCH`支持的客户端就可以拥有自己的更新请求。

## 日期和时间字段的格式 <a href="#formatting-date-and-time-fields" id="formatting-date-and-time-fields" class="headerlink"></a>

虽然JSON API并未指定日期和时间字段的格式,但推荐服务器使用与ISO 8601相符的格式。
这个页面[[W3C NOTE](https://www.w3.org/TR/NOTE-datetime)]提供了推荐格式的概述。

## 异步处理 <a href="#asynchronous-processing" id="asynchronous-processing" class="headerlink"></a>

考虑当你需要创建一个资源时操作需要很长时间的情况。

```javascript
POST /photos HTTP/1.1
```

这个请求应该返回一个`202 Accepted`的状态码,其报头中`Content-Location`有一个链接。

```javascript
HTTP/1.1 202 Accepted
Content-Type: application/vnd.api+json
Content-Location: https://example.com/photos/queue-jobs/5234

{
  "data": {
    "type": "queue-jobs",
    "id": "5234",
    "attributes": {
      "status": "Pending request, waiting other process"
    },
    "links": {
      "self": "/photos/queue-jobs/5234"
    }
  }
}
```

如需检查处理的状态,客户端可以向所给地址更早发出请求。

```javascript
GET /photos/queue-jobs/5234 HTTP/1.1
Accept: application/vnd.api+json
```

当处理完成时,请求应该返回一个`303 See other`的状态码,其报头中`Location`有一个链接。

```javascript
HTTP/1.1 303 See other
Content-Type: application/vnd.api+json
Location: https://example.com/photos/4577
```