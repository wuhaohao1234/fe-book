```javascript
class Koa {
  constructor() {
    this.middlewares = [];
  }

  // 注册中间件
  use(middleware) {
    this.middlewares.push(middleware);
  }

  // 执行中间件链
  compose(ctx) {
    const dispatch = async (index) => {
      if (index === this.middlewares.length) return;

      const middleware = this.middlewares[index];
      await middleware(ctx, dispatch(index + 1));
    };
    
    return dispatch(0);
  }

  // 模拟请求处理
  run() {
    const context = {};
    return this.compose(context);
  }
}
```

使用示例

```javascript
const app = new Koa();

app.use(async (ctx, next) => {
  console.log('1 Start');
  await next();
  console.log('1 End');
});

app.use(async (ctx, next) => {
  console.log('2 Start');
  await next();
  console.log('2 End');
});

app.use(async (ctx, next) => {
  console.log('3 Start');
  await next();
  console.log('3 End');
});

app.run();
```

