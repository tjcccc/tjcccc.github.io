---
layout: post
title: Two Ways to Script and Modularize GraphQL Schema
key: 20190307
tags: GraphQL Schema
---
# GraphQL Schema 的两种写法及模块化

上个礼拜学习了一下 GraphQL。我觉得入门的一个难点就是写 Schema，跟着 YouTube 教学视频和文档，学习了两种写法，一种是用 GraphQL 的 Schema 语法写，另一种是用 GraphQLSchema 构造器写。

## Using GraphQL Schema Language

内置的 Schema 语法是写在模板字符串里的，由 `buildSchema()` 创建。

<!--more-->

举个例子，有两个数据类型：`User` 和 `Post`，要求可以查询 User 和 Posts、可以创建 User 和 Post：

```js
const express = require('express');
const graphqlHTTP = require('express-graphql');
const { buildSchema } = require('graphql');

var rootSchema = buildSchema(`
  type User {
    id: String
    name: String
  }
  
  type Post {
    id: String
    title: String
    content: String
  }
  
  input PostInput {
    title: String,
    content: String
  }

  type RootQuery {
    user(id: String): User,
    posts: [Posts]
  }
  
  type RootMutation {
    createUser(name: String): User,
    createPost(newPost: PostInput): Post
  }
`);

const rootResolvers = {
  user: (id) => {
    return user;
  },
  posts: () => {
    return posts;
  },
  createUser: (name) => {
    return user;
  }
  createPost: (args) => {
    return post;
  }
};

const app = express();
app.use('/graphql', graphqlHTTP({
  schema: rootSchema,
  rootValue: rootResolvers,
  graphiql: true,
}));
app.listen(4000);
```

`rootResolvers` 中的 `user` 和 `posts` 是查询的方法，方法名要和 `schema` 中 `RootQuery` 一样，前者是具体实现，后者类似于登记声明。`createUser` 和 `createPost` 对数据进行了变更，所以要放在 `RootMutation` 中定义。对 `createPost` 的参数输入定义了一个新的类型 `PostInput`，也写在了 `schema` 中，用 `input` 定义，如果输入参数多，可用此法。

`rootResolvers` 中方法的具体内容省略了。

再看：

```js
app.use('/graphql', graphqlHTTP({
  schema: rootSchema,
  rootValue: rootResolvers,
  graphiql: true,
}));
```

把定义好的 `rootSchema` 和 `rootResolvers` 填进去，GraphQL 服务就可以使用了。

## Modularize for Schema Language

上文例子简单明了，但并不适于一般正经项目。因为数据不可能这么少，难道各种定义和方法都写在一个文件里？必须模块化。

首先 `rootSchema` 和 `rootResolvers` 要分成两个文件吧。然后继续细分。`rootSchema` 可分为 `userSchema` 和 `postSchema` 两种。以 `postSchema` 为例：

```js
module.exports.postSchema = `
type Post {
  id: String
  title: String
  content: String
}

input PostInput {
  title: String
  content: String
}`;

module.exports.postQuery = `posts: [Post]`;
module.exports.postMutation = `createPost(newPost: PostInput): Post`
```

因为 `schema` 实际由 Type、Query、Mutation 三种定义构成，所以每个 `schema` 模块最好定义三块内容。

全部定义好，在 `rootSchema` 中整合：

```js
const { userSchema, userQuery, userMutation } = require('./user.schema');
const { postSchema, postQuery, postMutation } = require('./post.schema');

const schemas = buildSchema(
  ``.concat(
    userSchema,
    postSchema,
    `
    type RootQuery {
      ${userQuery},
      ${postQuery}
    }

    type RootMutation {
      ${userMutation},
      ${postMutation}
    }

    schema {
      query: RootQuery
      mutation: RootMutation
    }
    `
  )
);

module.exports = schemas;
```

因为是字符串，所以可用 `concat` 来连接。一拼接就像样多了。

接着是 `rootResolvers` 的模块化。这个相对简单，就是定义好每种类型的 Query 和 Mutation 方法，然后整合起来：

```js
const userResolver = require('./user.resolver');
const postResolver = require('./post.resolver');

const resolvers = {
  ...userResolver,
  ...postResolver
}

module.exports = resolvers;
```

最后，原来的写法就变成了：

```js
const express = require('express');
const graphqlHTTP = require('express-graphql');
const rootSchema = require('./schemas/index');
const rootResolvers = require('./resolvers/index');

const app = express();
app.use('/graphql', graphqlHTTP({
   schema: rootSchema,
   rootValue: rootResolvers,
   graphiql: true,
}));

app.listen(4000);
```

## Using GraphQLSchema Constructor

用 Schema 语法有时觉得还是太 low 了，能不能更面向对象一些，更 programmatically 一些？可以的，用 `GraphQLSchema()` 来构造 schema。

写在一个文件里是这样的：

```js
const express = require('express');
const graphqlHTTP = require('express-graphql');
const graphql = require('graphql');

const UserType = new graphql.GraphQLObjectType({
  name: 'User',
  fields: {
    id: { type: graphql.GraphQLString },
    name: { type: graphql.GraphQLString },
  }
});

const PostType = new graphql.GraphQLObjectType({
  name: 'Post',
  fields: {
    id: { type: graphql.GraphQLString },
    title: { type: graphql.GraphQLString },
    content: { type: graphql.GraphQLString }
  }
});

const PostInput = new GraphQLInputObjectType({
  name: 'PostInput',
  fields: {
    title: { type: GraphQLString },
    content: { type: GraphQLString }
  }
});

const queryType = new graphql.GraphQLObjectType({
  name: 'Query',
  fields: {
    users: {
      type: new GraphQLList(UserType),
      args: null,
      resolve: (_, {id}) => {
        return user;
      }
    },
    createUser: {
      type: {
        name: { type: GraphQLString }
      },
      resolve: (_, args) => {
        return newUser;
      }
    },
    posts: {
      type: new GraphQLList(PostType),
      args: null,
      resolve: async () => {
        return posts;
      }
    },
    createPost: {
      type: PostType,
      args: {
        newPost: { type: PostInput }
      },
      resolve: (_, args) => {
        return newPost;
      }
    }
  }
});

var schema = new graphql.GraphQLSchema({query: queryType});

var app = express();
app.use('/graphql', graphqlHTTP({
  schema: schema,
  graphiql: true,
}));
app.listen(4000);
console.log('Running a GraphQL API server at localhost:4000/graphql');
```

注意它定义属性类型是用 `{ type: graphql.GraphQLString }` 这种方式的，还有数组类型是 `new GraphQLList(YourType)`。

## Modularize for GraphQLSchema

接下来用模块化的方式重构。方式很简单，Type 放在一个文件夹里，每个 Model 一个文件，Query 也是按 Model 分，再用一个 Index 整合。

Type 的例子：

```js
// post.type.js

const PostType = new GraphQLObjectType({
  name: 'Post',
  fields: () => ({
    id: { type: GraphQLString },
    title: { type: GraphQLString },
    content: { type: GraphQLString }
  })
});

module.exports.PostType = PostType;
```

输入参数类型可用 `GraphQLInputObjectType()` 定义：

```js
const PostInput = new GraphQLInputObjectType({
  name: 'PostInput',
  fields: {
    title: { type: GraphQLString },
    content: { type: GraphQLString }
  }
});

module.exports.PostInput = PostInput;
```

Query 的例子（具体定义省略）：

```js
const { PostType, PostInput } = require('../types/post.query');

const postQueries = {
  posts: {
    type: new GraphQLList(PostType),
    args: null,
    resolve: async () => {
      return posts;
    }
  },
  createPost: {
    type: PostType,
    args: {
      newPost: { type: PostInput }
    },
    resolve: (_, args) => {
      return newPost;
    }
  }
}

module.exports.postQueries = postQueries;
```

整合 Query 如下所示：

```js
// queries/index.js

const { GraphQLObjectType } = require('graphql');
const { userQueries } = require('../queries/user.query');
const { postQueries } = require('../queries/post.query');

const QueryType = new GraphQLObjectType({
  name: 'QueryType',
  fields: {
    ...userQueries,
    ...postQueries
  }
});

module.exports = QueryType;
```

然后再用来构造 Schema：

```js
// schema.constructor.js

const { GraphQLSchema } = require('graphql');
const QueryType = require('./queries/index');

module.exports = new GraphQLSchema({
  query: QueryType
});
```

最后：

```js
const express = require('express');
const graphqlHTTP = require('express-graphql');
const schemaConstructor = require('./schema.constructor');

const app = express();
app.use('/graphql', graphqlHTTP({
   schema: schemaConstructor
   graphiql: true,
}));

app.listen(4000);
```

注意此时已经不需要 `rootValue` 了。

## To Be Continued

写了这么多，只是刚好用来查询而已，只能算入了个门。觉得 GraphQL 真的挺方便，我将继续深入学习和使用。