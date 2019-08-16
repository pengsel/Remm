### RestfulAPI

1. 命名：小写

2. 版本：URL放入API版本号

3. 路径：API具体网址

   * 名词
   * 复数

4. HTTP动词与返回结果

   | Request Type | Database Operation | Description    | Return         |
   | ------------ | ------------------ | -------------- | -------------- |
   | GET          | SELECT             | 取出           | 单个资源或列表 |
   | POST         | CREATE             | 新建           | 新生成资源     |
   | PUT          | UPDATE             | 更新(完整资源) | 完整资源       |
   | PATCH        | UPDATE             | 更新(属性)     | 完整资源       |
   | DELETE       | DELETE             | 删除           | 空文档         |

5. 过滤信息：过滤返回结果

   * limit
   * offset
   * page、per_page
   * sortby、order

6. 状态码

   | Status Code | Message               | Request Type   | Detailed Description   |
   | ----------- | --------------------- | -------------- | ---------------------- |
   | 200         | OK                    | GET            | 成功返回               |
   | 201         | CREATED               | POST/PUT/PATCH | 新建或更新成功         |
   | 202         | ACCEPTED              | *              | 异步任务进入后台排队   |
   | 204         | NO CONTENT            | DELETE         | 删除成功               |
   | 400         | NVALID REQUEST        | POST/PUT/PATCH | 无效请求               |
   | 401         | Unauthorized          | *              | 没有权限               |
   | 403         | Forbidden             | *              | 得到授权，但访问被禁止 |
   | 404         | NOT FOUND             | *              | 请求资源不存在         |
   | 406         | Not Acceptable        | GET            | 格式不可得             |
   | 410         | Gone                  | GET            | 资源被永久删除         |
   | 422         | Unprocessable         | POST/PUT/PATCH | 创建对象时发生错误     |
   | 500         | INTERNAL SERVER ERROR | *              | 服务器错误             |

   ps：幂等性，对同一个系统，使用同样的条件，一次请求和重复的多次请求对系统资源的影响是一致的。

7. 错误处理：{error:"出错信息"}

8. Hypermedia API：返回结果提供链接