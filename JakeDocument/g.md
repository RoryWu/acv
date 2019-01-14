# 问题汇总

## 1.Backend

1. 关于跨域

```go
beego.InsertFilter("*", beego.BeforeRouter, cors.Allow(&cors.Options{
   AllowAllOrigins:  true,
   AllowMethods:     []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
   AllowHeaders:     []string{"Origin", "Authorization", "Access-Control-Allow-Origin", "Access-Control-Allow-Headers", "Content-Type", "accesstoken"},
   ExposeHeaders:    []string{"Content-Length", "Access-Control-Allow-Origin", "Access-Control-Allow-Headers", "Content-Type"},
   AllowCredentials: true,
}))
```

2. 关于jwt

3. 生成文档：

   第一次创建文档:

   > bee run -gendoc=true -downdoc=true

   更新文档：

   > bee generate docs 

4. Json / string / struct

   > https://blog.csdn.net/zhaoliang831214/article/details/80254214



5. ETC





## 2. Frontend

1. transformRequest 的作用

   ```javascript
   transformRequest: [function (data) {
       // let ret = '';
       // for (let it in data) {
       //     ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&';
       // }
       // return ret;
     let data2 = JSON.stringify(data)
     return data2
   }],
   ```

2. axios.get

3. 





