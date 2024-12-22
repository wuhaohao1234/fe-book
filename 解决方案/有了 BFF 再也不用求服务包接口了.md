虽然很多后端同学会调侃自己的工作是 CURD，但最让后端同学方案的工作可能是没完没了为前端包接口，某个接口已经为 App 写了，但 Web 上增删了一些字段，改了一下结构就要重写一个新接口，确实没什么技术含量，还要跟着走一遍需求流程

BFF（Backend for Frontend）服务于前端的后端，它为不同的前端（如移动应用、网页应用等）提供定制化的后端服务。这些服务会根据前端的需求进行数据的聚合、裁剪和转换，以适应不同设备和场景的特点，旨在解决前端与后端协作中的低效问题

## BFF 技术选型
业界有非常多 BFF 的解决方案，因为 BFF 接口代码大部分时候由前端来写，所以很多团队选择了使用前端更熟悉的 Node.js 来实现 BFF，但实践下来根据团队现有的后端技术栈和语言选择 BFF 技术栈，往往能带来更多的优势和长期收益

虽然 BFF 主要服务于前端，但本质上都是调用后端服务能力，很多服务有可能面临与 Node.js 互调用的集成成本，选择与团队现有后端技术栈一致的语言，可以更方便地与现有系统、数据库、中间件等进行集成，简化数据传输和服务调用

同时如果只是简单使用 Node.js 来实现 BFF，仅仅是把封装 DO 转 VO 的工作转嫁给了前端，考虑到前端同学可能对后端服务能力知识的欠缺，并不会带来显著的效率提升

GraphQL 是 Facebook 开发的一种用于 API 的查询语言，以及用于响应这些查询的运行时环境。GraphQL 的核心理念是客户端驱动的数据获取, 这意味着客户端指定它需要什么样的数据结构，服务器则返回符合要求的数据，非常适合数据查询为主的 BFF 诉求

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1734245983928-38056206-4cb9-4807-908e-8b14ac97a5b5.png)

GraphQL 有 Java、Go、Python、PHP、Node.js、C#、Rust 等主流语言的实现，可以轻松集成到团队的后端技术栈里

## GraphQL 核心概念
### Schema 和类型系统
**<font style="color:rgb(51, 51, 51);">Schema</font>**<font style="color:rgb(51, 51, 51);"> 是 GraphQL 的核心，定义了 API 的结构。它包括类型定义、查询类型、变更类型和订阅类型。类型系统确保了数据的一致性，并为开发者提供了自描述的 API</font>

+ **<font style="color:rgb(51, 51, 51);">标量类型（Scalar Types）</font>**<font style="color:rgb(51, 51, 51);">：基本的数据类型，如 </font>`<font style="color:rgb(51, 51, 51);">Int</font>`<font style="color:rgb(51, 51, 51);">, </font>`<font style="color:rgb(51, 51, 51);">Float</font>`<font style="color:rgb(51, 51, 51);">, </font>`<font style="color:rgb(51, 51, 51);">String</font>`<font style="color:rgb(51, 51, 51);">, </font>`<font style="color:rgb(51, 51, 51);">Boolean</font>`<font style="color:rgb(51, 51, 51);">, </font>`<font style="color:rgb(51, 51, 51);">ID</font>`<font style="color:rgb(51, 51, 51);"></font>
+ **<font style="color:rgb(51, 51, 51);">对象类型（Object Types）</font>**<font style="color:rgb(51, 51, 51);">：具有一组字段的对象，如 </font>`<font style="color:rgb(51, 51, 51);">User</font>`<font style="color:rgb(51, 51, 51);">, </font>`<font style="color:rgb(51, 51, 51);">Post</font>`<font style="color:rgb(51, 51, 51);"></font>
+ **<font style="color:rgb(51, 51, 51);">枚举类型（Enum Types）</font>**<font style="color:rgb(51, 51, 51);">：有限集合的值，如 </font>`<font style="color:rgb(51, 51, 51);">Status</font>`<font style="color:rgb(51, 51, 51);"></font>
+ **<font style="color:rgb(51, 51, 51);">输入类型（Input Types）</font>**<font style="color:rgb(51, 51, 51);">：用于定义查询和变更输入的结构</font>
+ **<font style="color:rgb(51, 51, 51);">接口（Interfaces）</font>**<font style="color:rgb(51, 51, 51);"> 和 </font>**<font style="color:rgb(51, 51, 51);">联合类型（Unions）</font>**<font style="color:rgb(51, 51, 51);">：用于描述不同类型之间的关系</font>

```graphql
type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post]
}

type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
}

type Query {
    users: [User]
    user(id: ID!): User
    posts: [Post]
    post(id: ID!): Post
}

type Mutation {
    createUser(name: String!, email: String!): User
    createPost(title: String!, content: String!, authorId: ID!): Post
}
```

### 查询
Query 用于读取或获取数据，客户端通过构造查询，指定需要的数据字段

```graphql
query {
    user(id: "1") {
        name
        email
        posts {
            title
        }
    }
}
```

响应示例

```json
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com",
      "posts": [
        {
          "title": "GraphQL Introduction"
        },
        {
          "title": "Advanced GraphQL"
        }
      ]
    }
  }
}
```

### 变更
Mutation 用于创建、更新或删除数据，类似于 REST 的 POST、PUT、DELETE 操作

```graphql
mutation {
    createUser(name: "Bob", email: "bob@example.com") {
        id
        name
    }
}
```

响应示例

```json
{
  "data": {
    "createUser": {
      "id": "2",
      "name": "Bob"
    }
  }
}
```

### 订阅
Subscription 用于实时获取数据更新，类似于 WebSocket，客户端订阅某个事件，当该事件触发时服务器会推送更新的数据

```graphql
subscription {
    postAdded {
        id
        title
        author {
            name
        }
    }
}
```

响应示例

```json
{
  "data": {
    "postAdded": {
      "id": "3",
      "title": "Real-time GraphQL",
      "author": {
        "name": "Charlie"
      }
    }
  }
}
```

### 解析器
Resolver 是处理 GraphQL 查询和变更的函数，每个字段有一个对应的解析器函数，负责获取该字段的数据

Node.js 实现的 Apollo Server 解析器示例

```javascript
const { ApolloServer, gql } = require('apollo-server');

// 定义 Schema
const typeDefs = gql`
    type User {
        id: ID!
        name: String!
        email: String!
        posts: [Post]
    }

    type Post {
        id: ID!
        title: String!
        content: String!
        author: User!
    }

    type Query {
        users: [User]
        user(id: ID!): User
        posts: [Post]
        post(id: ID!): Post
    }

    type Mutation {
        createUser(name: String!, email: String!): User
        createPost(title: String!, content: String!, authorId: ID!): Post
    }
`;

// 模拟数据
const users = [];
const posts = [];

// 定义解析器
const resolvers = {
  Query: {
    users: () => users,
    user: (_, { id }) => users.find(user => user.id === id),
    posts: () => posts,
    post: (_, { id }) => posts.find(post => post.id === id),
  },
  Mutation: {
    createUser: (_, { name, email }) => {
      const user = { id: `${users.length + 1}`, name, email };
      users.push(user);
      return user;
    },
    createPost: (_, { title, content, authorId }) => {
      const post = { id: `${posts.length + 1}`, title, content, authorId };
      posts.push(post);
      return post;
    }
  },
  User: {
    posts: (user) => posts.filter(post => post.authorId === user.id)
  },
  Post: {
    author: (post) => users.find(user => user.id === post.authorId)
  }
};

// 创建 Apollo Server 实例
const server = new ApolloServer({ typeDefs, resolvers });

// 启动服务器
server.listen().then(({ url }) => {
  console.log(`服务器已启动，访问地址：${url}`);
});
```

## 使用 Node.js 实现 GraphQL
为了方便理解，我们使用 apollo server 写一个简单的 BFF

### 文件结构
```plain
graphql-example/
├── schema.js
├── resolvers.js
├── index.js
├── package.json
└── README.md
```

### package.json
```json
{
  "name": "graphql-example",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "apollo-server": "^3.13.0",
    "graphql": "^16.9.0"
  }
}
```

### schema.js
```javascript
const { gql } = require('apollo-server');

const typeDefs = gql`
  type Product {
    id: ID!
    name: String!
    price: Float!
  }

  type Query {
    products: [Product]
    product(id: ID!): Product
  }

  type Mutation {
    addProduct(name: String!, price: Float!): Product
    updateProduct(id: ID!, name: String, price: Float): Product
  }
`;

module.exports = typeDefs;
```

### resolvers.js
```javascript
// 使用 mock 数据
let products = [
  { id: '1', name: 'Laptop', price: 999.99 },
  { id: '2', name: 'Phone', price: 499.99 },
];

const resolvers = {
  Query: {
    products: () => products,
    product: (_, { id }) => products.find(product => product.id === id),
  },
  Mutation: {
    addProduct: (_, { name, price }) => {
      const newProduct = { id: `${products.length + 1}`, name, price };
      products.push(newProduct);
      return newProduct;
    },
    updateProduct: (_, { id, name, price }) => {
      const product = products.find(product => product.id === id);
      if (!product) {
        throw new Error('Product not found');
      }
      if (name !== undefined) product.name = name;
      if (price !== undefined) product.price = price;
      return product;
    },
  },
};

module.exports = resolvers;
```

### index.js
```javascript
const { ApolloServer } = require('apollo-server');
const typeDefs = require('./schema');
const resolvers = require('./resolvers');

// 创建 Apollo Server 实例
const server = new ApolloServer({
  typeDefs,
  resolvers,
});

// 启动服务器
server.listen(4000).then(({ url }) => {
  console.log(`服务器已启动，访问地址：${url}`);
});
```

### 启动与测试
```bash
$ npm install
$ npm start
```

这时候会看到提示

```plain
服务器已启动，访问地址：http://localhost:4000/
```

在浏览器中打开 `http://localhost:4000/`，将看到 Apollo Server 提供的 **<font style="color:rgb(51, 51, 51);">Apollo Studio Explorer</font>**

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1734250483410-c61fe140-db5c-455b-bc1e-f3b123272da1.png)

### 使用 fetch 发送请求
**<font style="color:rgb(51, 51, 51);">GraphQL</font>**<font style="color:rgb(51, 51, 51);"> 请求通常通过 </font>**<font style="color:rgb(51, 51, 51);">HTTP POST</font>**<font style="color:rgb(51, 51, 51);"> 方法发送，其 Body 是一个 </font>**<font style="color:rgb(51, 51, 51);">JSON</font>**<font style="color:rgb(51, 51, 51);"> 对象，包含以下字段：</font>

+ `**<font style="color:rgb(51, 51, 51);">query</font>**`<font style="color:rgb(51, 51, 51);">：必需的，包含要执行的 GraphQL 查询或变更</font>
+ `**<font style="color:rgb(51, 51, 51);">variables</font>**`<font style="color:rgb(51, 51, 51);">：可选的，包含查询或变更中使用的变量</font>
+ `**<font style="color:rgb(51, 51, 51);">operationName</font>**`<font style="color:rgb(51, 51, 51);">：可选的，指定要执行的具体操作名称，特别是在同一请求中包含多个操作时</font>

```json
{
  "query": "GraphQL查询或变更字符串",
  "variables": { "变量名": "变量值", ... },
  "operationName": "操作名称"
}
```

使用 fetch 发送请求

```javascript
const query = `
  query GetProduct($id: ID!) {
    product(id: $id) {
      id
      name
      price
    }
  }
`;

const variables = { id: "2" };

fetch('http://localhost:4000/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ query, variables }),
}).then(res => res.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));

```

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1734250816869-55c8a88b-011c-49dd-affe-427d74ac5cb7.png)

