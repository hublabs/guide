**Coding guidelines for Hublabs**

# API

- 路由应该遵守RestfulAPI，**定位资源而不是定位动作**  
所以一般不应该出现 `get` `save` `modify` 等单词
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```
/product-api/v1/modify-skus
```

</td><td>

```
/product-api/v1/skus
// 【PUT method】
```

</td></tr>
</tbody></table>

### 特例：如果查询参数非常复杂，放在QueryString很麻烦，此时可以考虑使用POST请求，比如

```
/product-api/v1/skus/searches

// request body
{
    "filters": {
        "year": {
            "comparer": "in",
            "values": [
                "2019",
                "2018"
            ]
      }
}
```
这种写法并不违背RestfulAPI的原则，你可以将searches看做是一个资源（查询器），执行API的过程是创建查询器的过程

- 路由URL应该全部小写（不包括参数部分），单词之间用`-`连接
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```
/offer-api/v1/cart/Exclude_products
```

</td><td>

```
/offer-api/v1/cart/exclude-products
```

</td></tr>
</tbody></table>

- 参数部分（不管是QueryString还是Body中的）都应该使用CamelCase命名法，首字母小写，其他单词首字母大写，无连字符
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```
/product-api/v1/skus?Product_Id=1
```

</td><td>

```
/product-api/v1/skus?productId=1
```

</td></tr>
</tbody></table>

- 返回值应该统一使用 `github.com/labhubs/common/api` package中的`Result` struct  
参考：  
https://github.com/hublabs/common/tree/master/api#api-package-description  
https://github.com/hublabs/product-api/blob/master/controllers/utils.go

- Time格式使用`RFC3339`标准，比如`2019-07-11T07:29:05Z`  
由于这种格式带时区信息，Golang后台可以用`time.Time`类型接收和发送，不需要额外转换；前台使用`moment(t)`接收，使用`moment().format()`发送

- Date格式使用`yyyy-MM-dd`，除非有特殊要求，Time格式化到Date需要指定时区为`+8`

# Golang backend
- 除非查询结果的数据量在一定的范围内，比如查询品牌信息，最多一百条数据左右，否则应该分页
`maxResultCount`应该有默认值，比如30；当查询全部数据时，`maxResultCount`可以传-1
- 由于`xorm.Session`不支持`Goroutines`，所以请避免在多个`Goroutines`中使用同一个`xorm.Session`，可以用`xorm.Engine`代替
- 不要自己写排序算法，使用`sort.SliceStable`
- 避免使用硬代码表示类型，声明type定义类型，声明const定义类型枚举

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
GetOffer("brand")
```

</td><td>

```go
type OfferType string

const (
	OfferTypeBrand   OfferType = "brand"
	OfferTypeMember  OfferType = "member"
)

GetOffer(OfferTypeBrand)
```

</td></tr>
</tbody></table>

- 采用独立的错误流进行处理

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
if err != nil {
    // error handling
} else {
    // normal code
}
```

</td><td>

```go
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

</td></tr>
</tbody></table>

- 禁止取`for i, v := range s`中`v`的地址，注意修改`v`的值对`s`是无效的

- 使用Go modules管理项目包依赖
- - 包名必须以 `github.com/hublabs` 开始

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```
// go.mod
module product-api
```

</td><td>

```
// go.mod
module github.com/hublabs/product-api
```

</td></tr>
</tbody></table>

- - 提交代码前执行`go mod tidy`去除无用依赖

# DB
- 库名、表名、字段名应该全部小写，单词之间用`_`连接
- 尽可能将字段设置为`NOT NULL`
- 模糊查询禁止左模糊或者全模糊，比如`LIKE '%EE'`
- 分页查询时如果不需要页数，或`COUNT`语句太耗时，可以用`HasMore`模式，参考示例  
https://github.com/hublabs/product-api/blob/425e14691aa9cef1231cba8af3353925d3c50d4e/models/sku.go#L179
- 避免隐式转换，比如
```
type T struct {
    Keyword string `xorm:"index"`
}

// SQL
SELECT * FROM t WHERE keyword = 1
// 应该写成
SELECT * FROM t WHERE keyword = '1'
```

# Frontend
- 缩进使用2个空格
- 在`index.html`的`<head>`中加上以下代码，以避免发布后由于缓存无法更新页面的问题
```
<meta http-equiv="Cache-Control" content="no-cache" />
<meta http-equiv="Pragma" content="no-cache" />
<meta http-equiv="Expires" content="0" />
```
- 尽量使用`const`、`let`，不使用`var`
- 使用拓展运算符`...`复制数组

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```javascript
for (i = 0; i < items.length; i++) {
  itemsCopy[i] = items[i]
}
```

</td><td>

```javascript
itemsCopy = [...items]
```

</td></tr>
</tbody></table>

- 当需要使用对象的多个属性时，请使用解构赋值

<table>
<thead><tr><th>Bad</th><th>Good</th><th>Better</th></tr></thead>
<tbody>
<tr><td>

```javascript
function getFullName (user) {
  const firstName = user.firstName
  const lastName = user.lastName

  return `${firstName} ${lastName}`
}
```

</td><td>

```javascript
function getFullName (user) {
  const { firstName, lastName } = user

  return `${firstName} ${lastName}`
}
```

</td><td>

```javascript
function getFullName ({ firstName, lastName }) {
  return `${firstName} ${lastName}`
}
```

</td></tr>
</tbody></table>
