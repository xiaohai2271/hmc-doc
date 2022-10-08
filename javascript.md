# SDK使用文档

## 安装使用

1. 整个SDK，就dist目录下hmc.*.js 这个文件即可使用全部功能
2. 目前支持微信小程序、H5

!> 下载路径 [https://github.com/xiaohai2271/hmc-sdk-js](https://github.com/xiaohai2271/hmc-sdk-js)

### 引入

#### 压缩文件引入

`hmc-x.x.x.min.js` 见 dist 目录下 

```javascript
const Hmc = require('{path}/hmc-x.x.x.min.js');
```

#### 源码引入

- Typescript

```typescript
const Hmc = require('{path}/src/index');
```

- JavaScript

```javascript
const Hmc = require('{path}/dist/index');
```

上述`{path}`请替换成源码所在的路径。

!> Nodejs无法使用压缩文件引入方式，请使用源码引入或者下文的Npm引入。

#### 使用npm

**安装**

```shell
npm install hmc-js-sdk
```

**引入**

```javascript
import Hmc from "hmc-js-sdk";
```

**问题**

1. 如果报如下错误信息

```
import Hmc from "hmc-js-sdk";
^^^^^^

SyntaxError: Cannot use import statement outside a module
```

请在`package.json`中设置`"type": "module"`

### 示例

```typescript
import Hmc from "hmc-js-sdk";

Hmc.initialize('你的secretKey', '你的applicationKey')

Hmc.table.query('category').exec().then(res => {
  console.log(res)
})
// 或者
Hmc.table.query('category').exec((res, err) => {
  console.log(res)
  err && console.error(err)
})
```

## 数据表操作

> 如果你有一丢丢的sql基础，那么你使用本sdk会非常容易上手，SDK是在Sql规范上来进行的结构化封装，使用对象的模式来表达Sql语句。

使用`Hmc.table`即可对数据表进行数据操作，包含数据的新增、修改、删除和查询。

其中`Hmc.table` 的类型为 `Table`, 具有以下属性方法

```typescript
export interface Table {
  query: (tableName: string) => TableQuery;
  save: (tableName: string) => TableSave;
  del: (tableName: string) => TableDelete;
}
```

### 公用方法

#### 设定操作的表名

**查询数据**：`Hmc.table.query('表名')`

**新增或者更新数据**: `Hmc.table.save('表名')`

**删除数据**：`Hmc.table.del('表名')`，

下面以查询举例：

1. 在调用query的时候指定查询表名

```javascript
let query = Hmc.table.query('category') // 设置表名
```

2. 调用`table` 方法

方法签名： 

```typescript
table(name: string): TableQuery;
table(table: Table): TableQuery;
```

使用方法如下：

```javascript
let query2 = Hmc.table.query();
query2.table('category');// 设置表名
query2.table({name: 'category'});// 设置表名
```

#### 添加筛选条件

添加查询条件有两种方式，一种是使用对象实例作为查询条件，使用对象实例时，默认对象的键值之间为等值运算。另一种是手动添加where条件。

- **将 queryEntity 添加为 where 查询条件，且默认为 等值运算，即 field = value** 

方法签名: `entity(queryEntity: { [field: string]: any }): TableQuery;`

```javascript
let query = Hmc.table.query('category')
// 添加查询添加，将传入对象转换为where条件
query.entity({
    id: 1,
    amount: 1000,
    name: '工资'
})
```

- **设置where查询添加，查询条件会追加** 

方法签名: `where(conditions: WhereCondition[]): TableQuery;`

```javascript
let query = Hmc.table.query('category')
// 设置where查询条件，该函数入参为数组
query.where([
    {column: 'id', op: '=', value: 1},
    {column: 'amount', op: '=', value: 1000},
    {column: 'name', op: '=', value: '工资'}
])
```

> 上述两种方式可以同时使用，也可多次调用，后调用的同名键值会替换前次调用的同名键值。

!> 因新增数据和更新数据使用同一个方法，在新增数据时，请避免传入筛选条件，否则会被识别为更新数据


#### 执行数据操作

`exec<M>(callback?: (result: any, error: any) => void): Promise<M> | void;`
该方法接受一个回调函数入参，或者返回一个Promise对象，当传入回调函数时返回void，否则返回Promise对象

```javascript
let query = Hmc.table.query('category')
// 返回Promise对象
query.exec().then(res => {
    console.log(JSON.stringify(res))
}).catch(e => {
    console.error(e)
})
// 回调函数
query.exec((res, err) => {
    console.log(JSON.stringify(res))
    err && console.error(err)
})
```

### 查询数据

> 暂不支持多表查询，待后续更新。

调用`Hmc.table.query('表名')`会返回`TableQuery`对象，可以通过`TableQuery`对象，对本次查询进行设置。

下面的内容均基于`Hmc.table.query('表名')`操作。


#### 设置待查询的字段

>  默认为 * 查表所有字段

方法签名： `select(fields: string[]): TableQuery;`

```javascript
let query = Hmc.table.query('category')
query.select(['name', 'type']) // 查询name和type字段

query.select(['*']) // 查询所有字段，默认
```

#### 设置分页查询

传入页码和页面数据大小

方法签名：

`pageable(pageNumber: number, pageSize: number): TableQuery;`

```javascript
let query = Hmc.table.query('category')
query.pageable(1, 2) // 设置第一页 查两条数据
query.exec().then(res => {
    console.log(JSON.stringify(res))
}).catch(e => {
    console.error(e)
})
```

#### 设置查询去重

默认不去重

方法签名： `distinct(enable: boolean): TableQuery;`

```javascript
let query = Hmc.table.query('category')
query.select(['name']);
query.distinct(true) // 设置去重
```

#### 设置排序方式

方法签名： 

```typescript
orderBy(column: string, by: 'ASC' | 'DESC' | 'asc' | 'desc'): TableQuery;
orderBy(condition: OrderByCondition): TableQuery;
```

示例：

```typescript
let query = Hmc.table.query('category')
query.orderBy({id: "desc"}) // id 降序
query.orderBy('name', "asc") // name 升序
```



### 删除数据

#### 设置表名

见 [设定操作的表名](###设定操作的表名)

#### 添加查询条件

见 [添加筛选条件](###添加筛选条件)

### 新增数据

#### 设定为数据新增

方法签名: `saveMode(saveMode: boolean): TableSave;`

设定为数据新增模式, 该值可缺省，如果未调用该方法，执行时会根据是否有where筛选条件来进行判断，见 [添加筛选条件](###添加筛选条件)

```typescript
const save = Hmc.table.save('category')
save.saveMode(true); // 新增数据
```

#### 设置新增的数据

示例：

```typescript
const save = Hmc.table.save('category')

save.set("name", "张三")
save.set("age", 10)

save.entity({
    name: '张三',
    age: 10
})

```



### 更新数据

#### 设定为数据更新

方法签名: `saveMode(saveMode: boolean): TableSave;`

设定为数据新增模式, 该值可缺省，如果未调用该方法，执行时会根据是否有where筛选条件来进行判断，见 [添加筛选条件](###添加筛选条件)

```typescript
const update = Hmc.table.save('category')
update.saveMode(false); // 更新数据
```

#### 设置更新的数据

示例：

```typescript
const update = Hmc.table.save('category')

update.set("name", "张三")
update.set("age", 10)

update.entity({
    name: '张三',
    age: 10
})

update.where([{column: 'id', op: '=', value: 1}])
```

