### Mybatis 常见符号

```txt
原符号       <       <=      >       >=      <>
对应函数    lt()     le()    gt()    ge()    ne()
Mybatis-plus写法：  queryWrapper.ge("create_time", localDateTime);
Mybatis写法：       where create_time >= #{localDateTime}
```

