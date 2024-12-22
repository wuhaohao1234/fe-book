```javascript
function Express() {
  const middlewares = [];

  const app = () => {
    let index = 0;

    function next() {
      if (index < middlewares.length) {
        const currentMiddleware = middleware[index++];
        currentMiddleware(next);
      }
    }

    next();
  };

  app.use = middleware => {
    middlewares.push(middleware);
  };

  return app;
}
```

测试用例

```javascript
const app = express();

app.use(next => {
  console.log(1);
  next();
  console.log(6);
});
app.use(next => {
  console.log(2);
  next();
  console.log(5);
});
app.use(next => {
  console.log(3);
  next();
  console.log(4);
});

app();  // 1 2 3 4 5 6
```

