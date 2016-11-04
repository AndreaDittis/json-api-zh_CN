---
layout: page
title: "格式"
---

{% include status.md %}

## 介绍 <a href="#introduction" id="introduction" class="headerlink"></a>

JSON API 是数据交互规范，用以定义客户端如何获取与修改资源，以及服务器如何响应对应请求。

JSON API设计用来最小化请求的数量，以及客户端与服务器间传输的数据量。在高效实现的同时，无需牺牲可读性、灵活性和可发现性。

JSON API需要使用JSON API媒体类型([`application/vnd.api+json`](http://www.iana.org/assignments/media-types/application/vnd.api+json)) 进行数据交互。

JSON API服务器支持通过GET方法获取资源。而且必须独立实现HTTP POST, PUT和DELETE方法的请求响应，以支持资源的创建、更新和删除。

JSON API服务器也可以选择性支持HTTP PATCH方法 [[RFC5789](http://tools.ietf.org/html/rfc5789)]和JSON Patch格式 [[RFC6902](http://tools.ietf.org/html/rfc6902)]，进行资源修改。JSON Patch支持是可行的，因为理论上来说，JSON API通过单一JSON 文档，反映域下的所有资源，并将JSON文档作为资源操作介质。在文档顶层，依据资源类型分组。每个资源都通过文档下的唯一路径辨识。

## 规则约定 <a href="#conventions" id="conventions" class="headerlink"></a>

文档中的关键字， "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" 依据RFC 2119
[[RFC2119](http://tools.ietf.org/html/rfc2119)]规范解释。

##内容约定 <a href="content-negotiation" id="content-negotiation" class="headerlink"></a>

###客户端职责 <a href="#content-negotiation-client-responsibilities" id="content-negotiation-client-responsibilities" class="headerlink"></a>

客户端必须在包含`Content-Type: application/vnd.api+json`头并且不包含媒体类型参数的请求文档中发送所有JSON API数据。

<u>在`Accept`头中包含JSON API媒体类型并且不包含媒体类型参数的客户端必须在`Accept`头中指定媒体类型至少一次。</u>

客户端必须忽略任何从响应文档的`Content-Type`头中获取的`application/vnd.api+json`媒体类型参数。

###服务器职责 <a href="#content-negotiation-server-responsibilities" id="content-negotiation-server-responsibilities" class="headerlink"></a>

服务器必须在包含`Content-Type: application/vnd.api+json`头并且不包含媒体类型参数的请求文档中发送所有JSON API数据。

如果接收到一个用任何媒体类型参数指定`Content-Type: application/vnd.api+json`头的请求，服务器必须返回一个`415 Unsupported Media Type`状态码响应。

如果接收到一个在`Accept`头中包含任何JSON API媒体类型并且所有实体都以媒体类型参数更改的请求，服务器必须返回一个`406 Not Acceptable`状态码响应。

## 文档结构 <a href="#document-structure" id="document-structure" class="headerlink"></a>

这一章节描述JSON API文档结构，通过媒体类型[`application/vnd.api+json`](http://www.iana.org/assignments/media-types/application/vnd.api+json)标示。JSON API文档使用javascript 对象（JSON）[[RFC4627](http://tools.ietf.org/html/rfc4627)]定义。

尽管同种媒体类型用以请求和响应文档，但某些特性只适用于其中一种。差异在下面呈现。

除非另有说明，根据本规范定义的对象都不应该包含任何其他键。客户端和服务器实现必须忽略本规范未指定的键。

### Top Level <a href="#document-structure-top-level" id="document-structure-top-level" class="headerlink"></a>

JSON 对象必须位于每个JSON API文档的根级。这个对象定义文档的“top level”。

文档必须包含以下至少一种top-level键：

* `data`: 文档的”primary data”.
* `errors`: [错误对象](http://jsonapi.org/format/#errors)列表.
* `meta`: 包含非标准元信息的[元对象](http://jsonapi.org/format/#document-meta).

`data`键和`errors`键不能再一个文档中同时存在。

文档可能包含以下任何top-level键：

* `jsonapi`: 描述服务器实现的对象.
* `links`: 与primary data相关的[链接对象](http://jsonapi.org/format/#document-links).
* `include`: 与primary data或其他资源相关的资源对象("included resources")列表.

如果文档不包含top-level `data`键，`included`键也不应该出现。

文档的top-level [链接对象](http://jsonapi.org/format/#document-links)可能包含以下键：

* `self`: 生成当前响应文档的[链接](http://jsonapi.org/format/#document-links).
* `related`: 当primary data代表资源关系时，表示[相关资源链接](http://jsonapi.org/format/#document-resource-object-related-resource-links).
* Primary data的[分页](http://jsonapi.org/format/#fetching-pagination)链接.

文档中的“primary data”代表一个请求所要求的资源或资源集合。

Primary data 必须是以下列举的一种：

* 如果请求要求单一资源，应该是一个单一[资源对象](http://jsonapi.org/format/#document-resource-objects)，或一个单一[资源标识符](http://jsonapi.org/format/#document-resource-identifier-objects)，或`null`.
* 如果请求要求资源集合，应该是一个[资源对象](http://jsonapi.org/format/#document-resource-objects)列表，或一个空列表(`[]`).

例如，以下primary data表示一个单一资源对象：

```javascript
{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      // ... this article's attributes
    },
    "relationships": {
      // ... this article's relationships
    }
  }
}
```

以下primary data表示一个指向同样资源的单一[资源标识符](http://jsonapi.org/format/#document-resource-identifier-objects):

```javascript
{
  "data": {
    "type": "articles",
    "id": "1"
  }
}
```

即使只包含一个元素或为空，资源的一个逻辑集合也必须表示为一个列表。

### 资源对象 <a href="#document-structure-resource-objects" id="document-structure-resource-objects" class="headerlink"></a>

JSON API 文档中的"Resuorce objects"代表资源。

一个资源对象必须至少包含以下top-level键：

* `id`
* `type'

例外：当资源对象来自客户端并且代表一个将要在服务器创建的新资源时，`id`键不是必须的。

此外，资源对象可能包含以下top-level键：

* 'attribute': [属性对象](http://jsonapi.org/format/#document-resource-object-attributes)代表资源的某些数据.
* `relationshiops`: [关联对象](http://jsonapi.org/format/#document-resource-object-relationships)描述该资源与其他JSON API资源之间的关系.
* `links`: [链接资源](http://jsonapi.org/format/#document-links)包含与资源相关的链接.
* `meta`: [元数据资源](http://jsonapi.org/format/#document-meta)包含与资源对象相关的非标准元信息，这些信息不能被作为属性或关联对象表示.

一篇文献(即一个"文献"类型的资源)在文档中这样表示:

```javascript
// ...
{
  "type": "articles",
  "id": "1",
  "attributes": {
    "title": "Rails is Omakase"
  },
  "relationships": {
    "author": {
      "links": {
        "self": "/articles/1/relationships/author",
        "related": "/articles/1/author"
      },
      "data": { "type": "people", "id": "9" }
    }
  }
}
// ...
```

#### 标识符 <a href="#document-structure-resource-object-identification" id="document-structure-resource-object-identification" class="headerlink"></a>

每个[资源对象](http://jsonapi.org/format/#document-resource-objects)包含一个`id`键和一个`type`键。`id`键和`type`键的值必须是字符串类型。

对于每一个既定API，每个资源对象的`type`和`id`对必须定义一个单独且唯一的资源。(由一个或多个但行为表现为一个服务器的服务器控制的URI集合构成一个API。)

`type`键用于描述共享相同属性和关联的[资源对象](http://jsonapi.org/format/#document-resource-objects)。

`type`键的值必须与遵循[键名称](http://jsonapi.org/format/#document-member-names)相同的约束条件。

####字段<a href="#document-structure-resource-object-fields" id="document-structure-resource-object-fields" class="headerlink"></a>

资源对象的[属性](http://jsonapi.org/format/#document-resource-object-attributes)和[关联](http://jsonapi.org/format/#document-resource-object-relationships)被统称为"[fields](http://jsonapi.org/format/#document-resource-object-fields)"。

一个[资源对象](http://jsonapi.org/format/#document-resource-objects)的所有字段必须与`type`和`id`在同一命名空间中。即一个资源不能拥有名字相同的属性与关联，也不能拥有被命名为`type`或`id`的属性和关联。

####属性<a href="#document-structure-resource-object-attributes" id="document-structure-resource-object-attributes" class="headerlink"></a>

`attribute`键的值必须是一个对象(一个"attributes object")。属性对象的键("attributes")代表与[资源对象](http://jsonapi.org/format/#document-resource-objects)中定义的与其有关的信息。

属性可以包含任何合法JSON值。

JSON对象和列表涉及的复杂数据结构可以作为属性的值。但是一个组成或被包含于属性中的对象不能包含`relationships`或`links`键，因为这些键为此规范未来的用途所预留。

虽然一些has-one关系的外键(例如author_id)被在内部与其他将要在资源对象中表达的信息一起储存，但是这些键不能作为属性出现。

####关联<a href="#document-structure-resource-object-relationships" id="document-structure-resource-object-relationships" class="headerlink"></a>

`relationships`键的值必须是一个对象("relationships object")。

关联对象("relationships")的键表示在[资源对象](http://jsonapi.org/format/#document-resource-objects)中定义的与其相关的其他资源对象。

关联可以是单对象关联或多对象关联。

一个"relationship object"必须包含以下至少一种键：

* `links`: 一个[链接对象](http://jsonapi.org/format/#document-links)至少包含以下一种键：
  + `self`: 指向关联本身的链接("relationship link")。此链接允许客户端直接修改关联。例如，通过一个`articale`的关联URL移除一个`author`将会解除一个人与`article`的关系而不需要删除这个`people`资源本身。获取成功后，这个链接将返回一个相关资源之间的[连接](http://jsonapi.org/format/#document-resource-object-linkage)，将其作为primary data。(见 [获取关联](http://jsonapi.org/format/#fetching-relationships)).
  + `related`: [相关资源链接](http://jsonapi.org/format/#document-resource-object-related-resource-links).
* `data`: [资源连接](http://jsonapi.org/format/#document-resource-object-linkage).
* `meta`: 包含关于此关联的非标准元信息的[元对象](http://jsonapi.org/format/#document-meta).

一个代表多对象关联的关联对象可能在`links`键下也包含[分页](http://jsonapi.org/format/#fetching-pagination)链接，如后文描述。

####相关资源链接<a href="#document-structure-resource-object-related-resource-links" id="document-structure-resource-object-related-resource-links" class="headerlink"></a>

"related resource link"提供 [关联](http://jsonapi.org/format/#document-resource-object-relationships)中[链接](http://jsonapi.org/format/#document-links)的[资源对象](http://jsonapi.org/format/#document-resource-objects)的权限。当获取成功后，相关资源对象将作为响应的primary data返回。

例如，通过一个`GET`请求检索时，一篇`article`的`comments`关联可以指定一个返回评论集合[资源对象](http://jsonapi.org/format/#document-resource-objects)的[链接](http://jsonapi.org/format/#document-links)。

当一个相关资源链接出现时，它必须指向一个合法URL， 即使关联当前不予任何目标资源相关。此外，一个相关资源链接不能因为它的关联内容更改而更改。

####资源连接<a href="#document-structure-resource-object-resource-linkage" id="document-structure-resource-object-resource-linkage" class="headerlink"></a>

[复合文档](http://jsonapi.org/format/#document-compound-documents)中的资源连接允许客户端将所有包含的[资源对象](http://jsonapi.org/format/#document-resource-objects)链接在一起，而不需要通过[链接](http://jsonapi.org/format/#document-links)`GET`任何URL。

资源连接必须通过以下一种形式表述：

* 空的单对象关联用`null`表示.
* 空的多对象关联用一个空列表(`[]`)表示.
* 非空单对象关联用一个[资源标识符对象](http://jsonapi.org/format/#document-resource-identifier-objects)表示.
* 非空多对象关联用一个[资源标识符对象](http://jsonapi.org/format/#document-resource-identifier-objects)列表表示.

例如，以下文献与`author`相关：

```javascripts
// ...
{
  "type": "articles",
  "id": "1",
  "attributes": {
    "title": "Rails is Omakase"
  },
  "relationships": {
    "author": {
      "links": {
        "self": "http://example.com/articles/1/relationships/author",
        "related": "http://example.com/articles/1/author"
      },
      "data": { "type": "people", "id": "9" }
    }
  },
  "links": {
    "self": "http://example.com/articles/1"
  }
}
// ...
```

`author`关联包含一个指向关联本身的链接(这允许客户端直接更改相关作者)、一个用于获取资源对象的相关资源链接和一个连接信息。

####资源链接 <a href="#document-structure-resource-links" id="document-structure-resource-resource-links" class="headerlink"></a>

[资源对象](http://jsonapi.org/format/#document-resource-objects)中的可选的`links`键包含与资源相关的[链接](http://jsonapi.org/format/#document-links)。

当链接对象存在时，它可能包含指向定义了资源对象所表示的资源的`self`[链接](http://jsonapi.org/format/#document-links)。

```javascript
// ...
{
  "type": "articles",
  "id": "1",
  "attributes": {
    "title": "Rails is Omakase"
  },
  "links": {
    "self": "http://example.com/articles/1"
  }
}
// ...
```

服务器收到一个特定URL的`GET`请求后，必须回复将此资源作为promary data的响应。

###资源标识符对象<a href="#document-structure-resource-identifier-objects" id="document-structure-resource-identifier-objects" class="headerlink"></a>

“resource identifier object”是一个定义单独资源的对象。

“resource identifier object”必须包含`type`和`id`键。

“resource identifier object”也可能包含`meta`键，它的值是包含非标准元信息的[meta](http://jsonapi.org/format/#document-meta)对象。

###复合文档<a href="#document-structure-compound-documents" id="document-structure-compound-documents" class="headerlink"></a>

为了减少HTTP请求的数量，服务器可能允许包含相关资源与所请求的主要资源的响应。这种响应被称作"compound documents"。

在复合文档中，所有包含的资源必须以一个top-level中的`included`键中的[资源对象](http://jsonapi.org/format/#document-resource-objects)列表表示。

复合文档要求 “full linkage”，这意味着所有包含其中的资源必须至少被一个在同一文档中的[资源标识符对象](http://jsonapi.org/format/#document-resource-identifier-objects)定义。这些资源标识符对象可以是primary data或与主要或相关资源包含在一起的资源连接。

全连接要求唯一的例外是当包含连接数据的关联字段被通过稀疏字段集合排除时。

一个含有多个包含关联的复杂示例文档如下所示:

```javascript
{
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON API paints my bikeshed!"
    },
    "links": {
      "self": "http://example.com/articles/1"
    },
    "relationships": {
      "author": {
        "links": {
          "self": "http://example.com/articles/1/relationships/author",
          "related": "http://example.com/articles/1/author"
        },
        "data": { "type": "people", "id": "9" }
      },
      "comments": {
        "links": {
          "self": "http://example.com/articles/1/relationships/comments",
          "related": "http://example.com/articles/1/comments"
        },
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    }
  }],
  "included": [{
    "type": "people",
    "id": "9",
    "attributes": {
      "first-name": "Dan",
      "last-name": "Gebhardt",
      "twitter": "dgeb"
    },
    "links": {
      "self": "http://example.com/people/9"
    }
  }, {
    "type": "comments",
    "id": "5",
    "attributes": {
      "body": "First!"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "2" }
      }
    },
    "links": {
      "self": "http://example.com/comments/5"
    }
  }, {
    "type": "comments",
    "id": "12",
    "attributes": {
      "body": "I like XML better"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "9" }
      }
    },
    "links": {
      "self": "http://example.com/comments/12"
    }
  }]
}
```

对于每个`typy`和`id`对，一个[复合文档](http://jsonapi.org/format/#document-compound-documents)不能包含超过一个[资源对象](http://jsonapi.org/format/#document-resource-objects)。

###元信息<a href="#document-structure-meta-information" id="document-structure-meta-information" class="headerlink"></a>

在指定位置，`meta`键可用于包含非标准元信息。每个`meta`键的值必须是一个对象("meta object")。

任何键都可以包含在`meta`对象中。

例如: 

```javascript
{
  "meta": {
    "copyright": "Copyright 2015 Example Corp.",
    "authors": [
      "Yehuda Katz",
      "Steve Klabnik",
      "Dan Gebhardt",
      "Tyler Kellen"
    ]
  },
  "data": {
    // ...
  }
}
```

###链接<a href="#document-structure-links" id="document-structure-links" class="headerlink"></a>

在指定位置，`links`键可用于表示链接。每个`links`键的值必须是一个对象("links object")。

每个链接对象的键是一个"link"。一个链接必须是以下形其中之一：

* 一个包含链接的URL的字符串.
* 一个可以包含以下键的对象("link object"):
  - ​`href`: 一个包含此链接URL的字符串
  - `meta`: 包含与此链接相关非标准元信息的元对象.

以下`self`链接是一个简单的URL:

```javascript
"links": {
  "self": "http://example.com/posts"
}
```

以下`related`链接包含一个URL和与资源集合相关的元信息：

```javascript
"links": {
  "related": {
    "href": "http://example.com/articles/1/comments",
    "meta": {
      "count": 10
    }
  }
}
```

###JSON API 对象<a href="#document-structure-json-api-object" id="document-structure-json-api-object" class="headerlink"></a>

一个JSON API文档可能在top level `jsonapi`键下包含与其实现相关的信息。如果它出现，`jsonapi`键的值必须是一个对象("jsonapi object")。jsonapi对象可能包含`version`键，其值是一个表明被支持的最高JSON API版本的字符串。此对象也可能包含一个`meta`键，其值是一个包含非标准元信息的[meta](http://jsonapi.org/format/#document-meta)对象。

```javascript
{
  "jsonapi": {
    "version": "1.0"
  }
}
```

如果`version`键未出现，客户端则假设服务器是遵循此规范最新版本1.0。

####键名称<a href="#document-structure-member-names" id="document-structure-member-names" class="headerlink"></a>

一个JSON API文档中使用的键名称对于客户端和服务器来说必须是大小写敏感的，并且必须满足以下规则：

* 键名称必须包含至少一个字符.
* 键名称必须只包含以下列举的合法字符.
* 键名称必须以以下定义的“globally allowed character”开始和结束.

为了确保键名称能便利地映射到URL，推荐使用[RFC 3986](https://tools.ietf.org/html/rfc3986#page-13)中指定的非保留URL安全字符。

####合法字符 <a href="#document-structure-member-names-allowed-characters" id="document-structure-member-names-allowed-characters" class="headerlink"></a>

以下“globally allowed characters”可以被用于键名称中:

* U+0061 to U+007A, “a-z”
* U+0041 to U+005A, “A-Z”
* U+0030 to U+0039, “0-9”
* U+0080 and above (非ASCII Unicode 字符; 非URL安全字符，不推荐)

此外，以下字符也在键名称中也是合法的，只是不能出现在首位或末位:

* U+002D HYPHEN-MINUS, “-“
* U+005F LOW LINE, “_”
* U+0020 SPACE, “ “ (非URL安全字符，不推荐)

####保留字符<a href="#document-structure-member-names-reserved-characters" id="document-structure-member-names-reserved-characters" class="headerlink"></a>

以下字符不能被用于键名称中:

* U+002B PLUS SIGN, “+” (用于排序)
* U+002C COMMA, “,” (用于关联路径的分隔符)
* U+002E PERIOD, “.” (用于关联路径中的分隔符)
* U+005B LEFT SQUARE BRACKET, “[” (用于稀疏字段集合)
* U+005D RIGHT SQUARE BRACKET, “]” (用于稀疏字段集合)
* U+0021 EXCLAMATION MARK, “!”
* U+0022 QUOTATION MARK, ‘”’
* U+0023 NUMBER SIGN, “#”
* U+0024 DOLLAR SIGN, “$”
* U+0025 PERCENT SIGN, “%”
* U+0026 AMPERSAND, “&”
* U+0027 APOSTROPHE, “’”
* U+0028 LEFT PARENTHESIS, “(“
* U+0029 RIGHT PARENTHESIS, “)”
* U+002A ASTERISK, “*”
* U+002F SOLIDUS, “/”
* U+003A COLON, “:”
* U+003B SEMICOLON, “;”
* U+003C LESS-THAN SIGN, “<”
* U+003D EQUALS SIGN, “=”
* U+003E GREATER-THAN SIGN, “>”
* U+003F QUESTION MARK, “?”
* U+0040 COMMERCIAL AT, “@”
* U+005C REVERSE SOLIDUS, “"
* U+005E CIRCUMFLEX ACCENT, “^”
* U+0060 GRAVE ACCENT, “`”
* U+007B LEFT CURLY BRACKET, “{“
  * U+007C VERTICAL LINE, “”
* U+007D RIGHT CURLY BRACKET, “}”
* U+007E TILDE, “~”
* U+007F DELETE
* U+0000 to U+001F (C0 Controls)

## 数据获取 <a href="#fetching-data" id="fetching-data" class="headerlink"></a>

数据包括资源和关联，可以通过向后端发送`GET`请求获取数据。

响应可以通过以下描述的可选特征进一步提取。

### 资源获取 <a href="#fetching-data-fetching-resources" id="fetching-data-fetching-resources" class="headerlink"></a>

服务器必须支持通过URL获取资源，URL可以通过以下形式提供：

* top-level链接对象中的`self`链接.
* resource-level链接对象中的`self`链接.
* relationship-level链接对象中的`related`链接.

例如，以下请求获取一个文献集合：

```javascript
GET /articles HTTP/1.1
Accept: application/vnd.api+json
```

以下请求获取一篇文献：

```javascript
GET /articles/1 HTTP/1.1
Accept: application/vnd.api+json
```

以下请求获取一篇文献的作者:

```javascript
GET /articles/1/author HTTP/1.1
Accept: application/vnd.api+json
```

#### 响应 <a href="#fetching-data-fetching-resources-responses" id="fetching-data-fetching-resources-responses" class="headerlink"></a>

##### 200 OK 状态码 <a href="#fetching-data-fetching-resources-responses-200-ok" id="fetching-data-fetching-resources-responses-200-ok" class="headerlink"></a>

如果服务器通过请求成功获取一个单独资源或资源集合，服务器必须以`200 OK`状态码回复。

如果服务器通过请求成功获取一个资源集合，服务器必须将一个[资源对象](http://jsonapi.org/format/#document-resource-objects)的列表或空列表(`[]`)作为响应文档的主数据。

例如，对于一个针对文献集合的`GET`请求可以回复：

```javascript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "http://example.com/articles"
  },
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON API paints my bikeshed!"
    }
  }, {
    "type": "articles",
    "id": "2",
    "attributes": {
      "title": "Rails is Omakase"
    }
  }]
}
```

一个空集合的相似回复可以是以下形式:

```javascript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "http://example.com/articles"
  },
  "data": []
}
```

如果服务器可以通过请求成功获取一个单独资源，服服务器必须将一个[资源对象](http://jsonapi.org/format/#document-resource-objects)或`null`作为响应文档的主数据。

只有当请求URL可能指向、但当前并不指向一个单独资源时，`null`才应当作为响应。

例如，一个针对一篇文献的请求可以通过以下形式回复:

```javascript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "http://example.com/articles/1"
  },
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON API paints my bikeshed!"
    },
    "relationships": {
      "author": {
        "links": {
          "related": "http://example.com/articles/1/author"
        }
      }
    }
  }
}
```

如果以上文献的作者是缺失的，相关资源的`GET`请求可以通过以下形式回复：

```javasript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "http://example.com/articles/1/author"
  },
  "data": null
}
```

##### 404 Not Found 状态码 <a href="#fetching-data-fetching-resources-responses-404-not-found" id="fetching-data-fetching-resources-responses-404-not-found" class="headerlink"></a>

当通过一个请求获取一个不存在的单独资源时，服务器必须以`404 Not Found`状态码回复，除非请求允许`null`作为`200 OK`状态码响应的主数据(见后文描述)。

##### 其他响应 <a href="#fetching-data-fetching-resources-responses-other-responses" id="fetching-data-fetching-resources-responses-other-responses" class="headerlink"></a>

服务器可以回复其他HTTP状态码。

服务器可以在错误响应中包含[错误详情](http://jsonapi.org/format/#errors)。

服务器和客户端必须依据[HTTP语义](https://tools.ietf.org/html/rfc7231)分别生成和解析响应。

### 关联获取 <a href="#fetching-data-fetching-relationships" id="fetching-data-fetching-relationships" class="headerlink"></a>

服务器必须支持通过关联URL获取关联数据，关联URL可以通过关联的`links`对象中的`self`链接提供。

例如，以下请求获取文献的评论数据:

```javascript
GET /articles/1/relationships/comments HTTP/1.1
Accept: application/vnd.api+json
```

以下请求获取文献的作者数据:

```javascript
GET /articles/1/relationships/author HTTP/1.1
Accept: application/vnd.api+json
```

#### 响应 <a href="#fetching-data-fetching-relationships-responses" id="fetching-data-fetching-relationships-responses" class="headerlink"></a>

##### 200 OK 状态码 <a href="#fetching-data-fetching-relationships-responses-200-ok" id="fetching-data-fetching-relationships-responses-200-ok" class="headerlink"></a>

当服务器通过请求成功获取关联时，服务器应该以`200 OK`状态码回复。

响应文档的主数据必须是与[资源连接](http://jsonapi.org/format/#document-resource-object-linkage)相匹配的值，如上文对[关联对象](http://jsonapi.org/format/#document-resource-object-relationships)的描述。

Top-level[链接对象](http://jsonapi.org/format/#document-links)可以包含`self`和`related`链接，如上文对[关联对象](http://jsonapi.org/format/#document-resource-object-relationships)的描述。

例如，一个来自单对象关联链接的`GET`请求可以通过以下形式回复:

```javascript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "/articles/1/relationships/author",
    "related": "/articles/1/author"
  },
  "data": {
    "type": "people",
    "id": "12"
  }
}
```

如果以上关联为空，相同URL的`GET`请求可以通过以下形式回复: 

```javascript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "/articles/1/relationships/author",
    "related": "/articles/1/author"
  },
  "data": null
}
```

一个来自多对象关联链接的`GET`请求可以通过以下形式回复:

```javascript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "/articles/1/relationships/tags",
    "related": "/articles/1/tags"
  },
  "data": [
    { "type": "tags", "id": "2" },
    { "type": "tags", "id": "3" }
  ]
}
```

如果以上关联为空，相同URL的`GET`请求可以通过以下形式回复: 

```javascript
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "/articles/1/relationships/tags",
    "related": "/articles/1/tags"
  },
  "data": []
}
```

##### 404 Not Found 状态码 <a href="#fetching-data-fetching-relationships-responses-404-not-found" id="fetching-data-fetching-relationships-responses-404-not-found" class="headerlink"></a>

当通过一个请求获取一个不存在的关联链接URL时，服务器必须以`404 Not Found`状态码回复。

如果关联链接URL存在但是关联为空，如前文所述，服务器必须以`200 OK`状态码回复。

##### 其他响应 <a href="#fetching-data-fetching-resources-relationships-other-responses" id="fetching-data-fetching-relationships-responses-other-responses" class="headerlink"></a>

服务器可以回复其他HTTP状态码。

服务器可以在错误响应中包含[错误详情](http://jsonapi.org/format/#errors)。

服务器和客户端必须依据[HTTP语义](https://tools.ietf.org/html/rfc7231)分别生成和解析响应。

### 内联资源 <a href="#fetching-data-inclusion-of-related-resources" id="fetching-data-inclusion-of-related-resources" class="headerlink"></a>

后端可以默认回复与主数据相关的资源。

后端也可以支持`include`请求参数以保证客户端可以指定需要回复的相关资源。

如果后端不支持`include`参数，必须以`400 Bad Request`状态码回复任何包含此参数的请求。

如果后端支持`include`参数并且客户端使用了此参数，服务器不能在[复合文档](http://jsonapi.org/format/#document-compound-documents)的`included`部分包含任何未请求的[资源对象](http://jsonapi.org/format/#document-resource-objects)。

`include`参数的值必须是一个由逗号分隔符(U+002C COMMA, “,”) 分割的关联路径列表。关联路径是由点号分隔符(U+002E FULL-STOP, “.”) 分割的[关联](http://jsonapi.org/format/#document-resource-object-relationships)名称。

如果服务器不能识别关联路径或不能通过路径支持内联资源，必须以`400 Bad Request`状态码回复。

例如，可以同时请求评论和文献:

```javascript
GET /articles/1?include=comments HTTP/1.1
Accept: application/vnd.api+json
```

可以通过如下形式为每个关联名称定义逗号分割路径，以请求与其他资源相关的资源：

```javascript
GET /articles/1?include=comments.author HTTP/1.1
Accept: application/vnd.api+json
```

可以通过逗号分割列表请求多个相关资源:

```javascript
GET /articles/1?include=author,comments.author HTTP/1.1
Accept: application/vnd.api+json
```

此外，可以向关联后端请求相关资源:

```javascript
GET /articles/1/relationships/comments?include=comments.author HTTP/1.1
Accept: application/vnd.api+json
```

在这种情况下，主数据应该是代表此文献的评论连接的[资源标识符对象](http://jsonapi.org/format/#document-resource-identifier-objects)集合，所有评论和评论作者将会作为相关数据返回。

### 稀疏字段组 <a href="#fetching-sparse-fieldsets" id="fetching-sparse-fieldsets" class="headerlink"></a>

客户端可能通过`fields[TYPE]`参数,请求后端返回只包含特定字段的响应。

`fields`参数的值必须用逗号分隔开,用来表示需要返回字段的名称。

如果客户端请求了一组给定类型的字段,那么后端不能包括资源对象内此类型的附加字段。

```javascript
GET /articles?include=author&fields[articles]=title,body&fields[people]=name HTTP/1.1
Accept: application/vnd.api+json
```
### 排序 <a href="#fetching-sorting" id="fetching-sorting" class="headerlink"></a>

服务器可以选择性支持，根据一个或多个条件(排序字段)对资源集合排序。

后端可能支持带有`sort`查询参数的请求,来排序主要数据。`sort`的值必须代表排序字段。

```javascript
GET /people?sort=age HTTP/1.1
Accept: application/vnd.api+json
```

后端可能支持带有多个排序字段的请求,允许用逗号分隔排序字段。排序字段应该被按照特定顺序执行。

```javascript
GET /people?sort=age,name HTTP/1.1
Accept: application/vnd.api+json
```

每个排序字段必须按升序排列。除非它带有`-`前缀,这种情况下将按降序排列。

```javascript
GET /articles?sort=-created,title HTTP/1.1
Accept: application/vnd.api+json
```

以上的例子应首先返回最新的条目。在同一天创建的条目讲按照标题的字母升序排列。

如果服务器不支持查询参数`sort`指定的排序,必须返回`400 Bad Request`。

如果服务器支持排序,并且客户端通过查询参数`sort`进行排序,服务器必须返回根据指定条件排序的,
上层`data`数组的元素。

### 分页 <a href="#fetching-paginating" id="fetching-paginating" class="headerlink"></a>

服务器可以选择性限制响应返回的资源数量,为所有可获取资源的子集("page")。

服务器可以提供传送分页后数据集的连接("pagination links")。

`pagination links`必须出现在集合相关的连接对象中。需要在上层`links`对象中提供`pagination links`。

下面的键必须被用于`pagination links`:
* `first`: 第一页数据
* `last`: 最后一页数据
* `prev`: 前一页数据
* `next`: 后一页数据

如需表示特定的连接不可用,这些键必须被省略,或者值为`null`。

分页的排序,必须与JSON API的排序规则一致。

查询参数`page`是分页的保留字,服务器和客户端应该用这个参数进行分页。


### 过滤 <a href="#fetching-filtering" id="fetching-filtering" class="headerlink"></a>

查询参数`filter`是过滤的保留字,服务器和客户端应该用这个参数进行过滤。


## 创建，更新，删除资源 <a href="#crud" id="crud" class="headerlink"></a>

服务器可能支持给定类型资源的创建，服务器也可能允许已存在资源的更新和删除。

服务器允许单次请求，更新多个资源，如下所述。多个资源更新必须完全成功或者失败，不允许部分更新成功。

任何包含内容的请求，必须包含`Content-Type:application/vnd.api+json`请求头。 

### 创建资源 <a href="#crud-creating-resources" id="crud-creating-resources" class="headerlink"></a>


向表示待创建资源所属资源集的URL，发出`POST`请求，创建一个或多个资源。
请求必须包含单一资源对象作为主数据。资源对象必须包含至少一个`type`成员。

例如，新photo可以通过如下请求创建：

```javascript
POST /photos HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": {
    "type": "photos",
    "attributes": {
      "title": "Ember Hamster",
      "src": "http://example.com/images/productivity.png"
    },
    "relationships": {
      "photographer": {
        "data": { "type": "people", "id": "9" }
      }
    }
  }
}
```
如果资源对象在`relationships`提供了关系,它的值必须是一个有`data`成员的关系对象。
这个键的值代表新资源将要有的连接。

#### 客户端生成的ID <a href="#crud-client-generated-ids" id="crud-client-generated-ids" class="headerlink"></a>

服务器可能接受创建资源的请求中有客户端生成的ID。ID必须被`id`键指定,它的值必须是通用唯一识别码。
客户端应该使用 RGC4122 [`RGC4122`](http://tools.ietf.org/html/draft-ietf-
httpbis-p2-semantics-22#section-6.3)中描述的合适的UUID。

比如:

```javascript
POST /photos HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": {
    "type": "photos",
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "attributes": {
      "title": "Ember Hamster",
      "src": "http://example.com/images/productivity.png"
    }
  }
}
```

服务器必须对不支持的带有客户端生成ID的创建请求返回`403 Forbidden`。


#### 响应 <a href="#crud-creating-responses" id="crud-creating-responses" class="headerlink"></a>

##### 201 Created <a href="#crud-creating-responses-201" id="crud-creating-responses-201" class="headerlink"></a>

如果`POST`请求不包括客户端生成的ID,并且请求的资源成功被创建,服务器必须返回`201 Created`状态码。

响应应该包含`Location`头，用以标示请求创建所有资源的位置。

响应必须含有一个文档，用以存储所创建的主要资源。

如果响应返回的资源对象在`links`成员里包含`self`键,并且响应数据头提供了`Location`,
那么`self`的值必须匹配`Location`的值。

```javascript
HTTP/1.1 201 Created
Location: http://example.com/photos/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/vnd.api+json

{
  "data": {
    "type": "photos",
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "attributes": {
      "title": "Ember Hamster",
      "src": "http://example.com/images/productivity.png"
    },
    "links": {
      "self": "http://example.com/photos/550e8400-e29b-41d4-a716-446655440000"
    }
  }
}
```

##### 202 Accepted <a href="#crud-creating-responses-202" id="crud-creating-responses-202" class="headerlink"></a>

如果创建资源的请求被接受处理,但在服务器响应时处理并未完成,那么服务器必须返回`202 Accepted`状态码。

##### 204 No Content <a href="#crud-creating-responses-204" id="crud-creating-responses-204" class="headerlink"></a>

如果`POST`请求包括客户端生成的ID,并且请求的资源成功被创建,那么服务器必须返回`201 Created`状态码和响应文档(如上所述),
,或者只返回`204 No Content`状态码,没有响应文档。

##### 403 Forbidden <a href="#crud-creating-responses-403" id="crud-creating-responses-403" class="headerlink"></a>

服务器可能向不支持的创建资源的请求返回`403 Forbidden`的响应。

##### 404 Not Found <a href="#crud-creating-responses-404" id="crud-creating-responses-404" class="headerlink"></a>

如果创建请求引用的相关资源不存在,服务器必须返回`404 Not Found`的响应。

##### 409 Conflict <a href="#crud-creating-responses-409" id="crud-creating-responses-409" class="headerlink"></a>

如果创建请求中,客户端生成的ID已经存在,服务器必须返回`409 Conflict`的响应。

如果创建请求中,资源对象的`type`不在后端支持的类型里,服务器必须返回`409 Conflict`的响应。

服务器应该在响应中包括错误详情和足够的信息以识别冲突原因。

##### Other Responses <a href="#crud-creating-responses-other" id="crud-creating-responses-other" class="headerlink"></a>

服务器响应可能没有状态码。

服务器可能响应包括错误详情的错误响应。

服务器与客户端必须依照HTTP的语义准备和解译响应。


### 更新资源 <a href="#crud-updating" id="crud-updating" class="headerlink"></a>

向表示资源的URL发出`PATCH`请求，即可进行资源更新。

资源的URL可以从资源对象的`self`的值获得。或者,当`GET`请求返回了一个资源对象作为主资源,
同样的请求URL可被用来更新资源。

`PATCH`请求必须包括一个资源对象作为主资源。资源对象必须包含`type`和`id`成员。

比如:

```javascript
PATCH /articles/1 HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "To TDD or Not"
    }
  }
}
```

#### 更新资源属性 <a href="#crud-updating-resources-attributes" id="crud-updating-resources-attributes" class="headerlink"></a>

资源的任何一个,或者所有属性,可能被包括在`PATCH`请求的资源对象中。

如果请求不包括资源所有的属性,那么服务器解译请求时必须添加这些属性,并赋予当前的值。
服务器不能给缺失的属性赋值为null。

比如,下面的`PATCH`请求被解译为只更新`title`和`text`两个属性:

```javascript
PATCH /articles/1 HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "To TDD or Not",
      "text": "TLDR; It's complicated... but check your test coverage regardless."
    }
  }
}
```

#### 更新资源关联 <a href="#crud-updating-resource-relationships" id="crud-updating-resource-relationships" class="headerlink"></a>

资源的所有或者任何一个关联都可能包括在`PATCH`请求的资源对象中。

如果一个请求不包括资源所有的关联,那么服务器解译请求时必须添加这些属性,并赋予当前的值。
服务器不能给缺失的属性赋值为null或者为空。

如果关联是被`PATCH`请求的资源对象中`relationships`给出的,那么它的值必须是一个有`data`成员的关联对象。
关联的值将被这个成员定义的值代替。

比如,下面的`PATCH`请求将会更新`author`关联:

```javascript
PATCH /articles/1 HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": {
    "type": "articles",
    "id": "1",
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "1" }
      }
    }
  }
}
```

相似的,下面的`PATCH`请求会更新整个`tags`关联:

```javascript
PATCH /articles/1 HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": {
    "type": "articles",
    "id": "1",
    "relationships": {
      "tags": {
        "data": [
          { "type": "tags", "id": "2" },
          { "type": "tags", "id": "3" }
        ]
      }
    }
  }
}
```

服务器可能拒绝更新多对象关联的请求。在这种情况,服务器必须阻止整个更新,并返回`403 Forbidden`状态码。

### 响应 <a href="#crud-updating-responses" id="crud-updating-responses" class="headerlink"></a>

#### 202 Accepted <a href="#crud-updating-responses-202" id="crud-updating-responses-202" class="headerlink"></a>

如果服务器接受更新,但在服务器响应时更新并未完成,那么服务器必须返回`202 Accepted`状态码。

#### 204 No Content <a href="#crud-updating-responses-204" id="crud-updating-responses-204" class="headerlink"></a>

如果更新成功，且服务器没有更新任何未提供的属性，那么服务器必须或者返回`204 No Content`状态码和响应文档,或者只返回`204 No Content`状态码,没有响应文档。

#### 200 OK <a href="#crud-updating-responses-200" id="crud-updating-responses-200" class="headerlink"></a>

如果服务器接受更新，但是在请求指定内容之外做了资源修改，必须响应`200 OK`以及更新的资源实例，像是向此URL发出`GET`请求。

如果更新成功,客户端当前属性保持更新,并且服务器只响应最上层元数据,那么服务器必须返回`200 OK`状态码。
这种情况下,服务器不能包括更新后的资源。

#### 其它响应 <a href="#crud-updating-responses-other" id="crud-updating-responses-other" class="headerlink"></a>

服务器使用其它HTTP错误状态码反映错误。
客户端必须依据HTTP规范处理这些错误信息。
如下所述，错误细节可能会一并返回。

### 更新关联 <a href="#crud-updating-relationships" id="crud-updating-relationships" class="headerlink"></a>

虽然关联可以通过资源被更新(如上所述),JSON API也支持单独通过`Relationship Links`更新关联。

#### 更新To-One 关联 <a href="#crud-updating-toone-relationship" id="crud-updating-toone-relationship" class="headerlink"></a>

`PATCH`请求必须包括上层名为`data`的成员,包括:

相关新资源的资源表示对象

或者

`null`,用来删除关联。

比如,下面的请求更新author关联:

```javascript
PATCH /articles/1/relationships/author HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": { "type": "people", "id": "12" }
}
```

而下面的请求会清除author关联:

```javascript
PATCH /articles/1/relationships/author HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": null
}
```

如果关联被成功更新,服务器必须返回一个成功的响应。

#### 更新To-Many 关联 <a href="#crud-updating-tomany-relationship" id="crud-updating-tomany-relationship" class="headerlink"></a>

对所有请求类型,主体必须包括一个`data`成员,其值是一个空数组,或是一个`资源标识对象`的数组。

如果客户端向一个to-many关联连接的URL发出`PATCH`请求,服务器必须或者完全更改关联的每一个成员,
如果资源不存在或者无法使用,返回合适的错误状态码,如果服务器不允许完全更改,则返回`403 Forbidden`。

比如,下面的请求会更改所有的`tag`:

```javascript
PATCH /articles/1/relationships/tags HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": [
    { "type": "tags", "id": "2" },
    { "type": "tags", "id": "3" }
  ]
}
```

而下面的请求会清除所有的`tag`:

```javascript
PATCH /articles/1/relationships/tags HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": []
}
```

如果客户端向一个关联连接的URL发出`POST`请求,那么服务器必须向关联添加新增的成员,
除非他们已经存在。如果所给的`type`和`id`已经存在,服务器不能再添加他们。

如果所有指定的资源都可以被添加到关联,或者已经存在在关联里,那么服务器必须返回一个成功响应。

在下面的例子中,ID为`123`的评论被添加到评论列表内ID为`1`的条目中。

```javascript
POST /articles/1/relationships/comments HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": [
    { "type": "comments", "id": "123" }
  ]
}
```

如果客户端向一个关联连接的URL发出`DELETE`请求,那么服务器必须在关联中删除指定的成员,或者返回`403 Forbidden`响应。
如果所有指定的资源都可以从关联中删除,或者未存在在关联中,那么服务器必须返回一个成功响应。

关联成员是被与`POST`请求同样的方式指定的。

在下面的例子中,ID为`12`和`13`的评论被从评论列表内ID为`1`的条目中删除:

```javascript
DELETE /articles/1/relationships/comments HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "data": [
    { "type": "comments", "id": "12" },
    { "type": "comments", "id": "13" }
  ]
}
```
### 响应 <a href="#crud-updating-relationship-responses" id="crud-updating-relationship-responses" class="headerlink"></a>

#### 202 Accepted <a href="#crud-updating-relationship-responses-202" id="crud-updating-relationship-responses-202" class="headerlink"></a>

如果服务器接受关联更新请求,但在服务器响应时更新并未完成,那么服务器必须返回`202 Accepted`状态码。

#### 204 No Content <a href="#crud-updating-relationship-responses-204" id="crud-updating-relationship-responses-204" class="headerlink"></a>

如果更新成功，并且请求中的资源与结果相符，那么服务器必须或者返回`204 No Content`状态码。

#### 200 OK <a href="#crud-updating-relationship-responses-200" id="crud-updating-relationship-responses-200" class="headerlink"></a>

如果服务器接受更新,但用在请求指定以外的方式更改了目标关联,那么b必须返回`200 OK`响应。
响应文档必须包括被更新关联的代表。

如果更新成功,客户端当前数据保持更新,并且服务器只响应最上层元数据,那么服务器必须返回`200 OK`状态码。
这种情况下,服务器不能包括更新后的资源。

#### 403 OK <a href="#crud-updating-relationship-responses-403" id="crud-updating-relationship-responses-403" class="headerlink"></a>

服务器必须向不支持的更新关联请求发出`403 Forbidden`响应。

#### 其它响应 <a href="#crud-updating-relationship-responses-other" id="crud-updating-relationship-responses-other" class="headerlink"></a>

服务器可能使用其它HTTP错误状态码反映错误。

服务器可能响应包括错误详情的错误响应。

服务器与客户端必须依照HTTP的语义准备和解译响应。


### 资源删除 <a href="#crud-deleting" id="crud-deleting" class="headerlink"></a>

向资源URL发出`DELETE`请求即可删除单个资源。

```text
DELETE /photos/1 HTTP/1.1
Accept: application/vnd.api+json
```

#### 响应 <a href="#crud-deleting-responses" id="crud-deleting-responses" class="headerlink"></a>

##### 202 Accepted <a href="#crud-deleting-responses-202" id="crud-deleting-responses-202" class="headerlink"></a>

如果删除资源的请求被接受处理,但在服务器响应时处理并未完成,那么服务器必须返回`202 Accepted`状态码。

##### 204 No Content <a href="#crud-deleting-responses-204" id="crud-deleting-responses-204" class="headerlink"></a>

如果删除请求成功，服务器必须返回`204 No Content` 状态码,并且没有返回内容

##### 200 OK <a href="#crud-deleting-responses-200" id="crud-deleting-responses-200" class="headerlink"></a>

如果删除请求成功,服务器必须返回`200 OK`状态码,并且返回上层的元数据。

##### 404 NOT FOUND <a href="#crud-deleting-responses-404" id="crud-deleting-responses-404" class="headerlink"></a>

如果因为资源不存在,删除请求失败,那么服务器应该返回`404 Not Found`状态码。

##### 其它响应 <a href="#crud-deleting-responses-other" id="crud-deleting-responses-other" class="headerlink"></a>

服务器可能使用其它HTTP错误状态码反映错误。

服务器可能响应包括错误详情的错误响应。

服务器与客户端必须依照HTTP的语义准备和解译响应。


## 查询参数 <a href="#query-parameters" id="query-parameters" class="headerlink"></a>

查询参数必须遵守成员命名的规则,同时必须包含至少一个非a-z字符(U+0061 to U+007A)。推荐使用 U+002D HYPHEN-MINUS, “-“, U+005F LOW LINE, “_”, 或者大写字母(如: camelCasing)。

如果服务器获取到不遵从以上命名规则的查询参数,并且不知道如何处理,那么服务器必须返回`400 Bad Request`。


## 错误 <a href="#errors" id="errors" class="headerlink"></a>

#### 处理错误 <a href="#errors-processing-error" id="errors-processing-error" class="headerlink"></a>

当遇到一个问题时,服务器可以选择停止处理,也可以继续处理,遇到多个问题。
比如,服务器可能处理多个属性,并在一个响应中返回多个验证问题。

当服务器在一个请求中遇到多个问题,在响应中应该使用HTTP最通用的错误状态码。
比如,`400 Bad Request`可被用于多个4xx错误,`500 Internal Server Error`可被用于多个5xx错误。

#### 错误对象 <a href="#errors-error-objects" id="errors-error-objects" class="headerlink"></a>

错误对象提供执行操作时遇到问题的额外信息。
在JSON API文档顶层，错误对象必须被作为`errors`键对应的数组返回。

错误对象可能有以下成员：

* `id` - 特定问题的唯一标识符。
* `links` - 一个包括以下成员的连接对象:
  * `about` - 指向特定问题更多具体内容的连接。
* `status` - 适用于这个问题的HTTP状态码，使用字符串表示。
* `code` - 应用特定的错误码，以字符串表示。
* `title` - 简短的，可读性高的问题总结。除了国际化本地化处理之外，不同场景下，相同的问题，值是不应该变动的。
* `detail` - 针对该问题的高可读性解释。与`title`相同,这个值可以被本地化。
* `source` - 包括错误资源的引用的对象。它也可以包括以下任意成员:
  * `pointer` - 一个指向请求文档中相关实体的`JSON Pointer`。
  * `parameter` - 指出引起错误的查询对象的字符串。
* `meta` - 一个包括关于错误的非标准元信息的元对象。
