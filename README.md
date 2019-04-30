# APITestGuide
手把手教会你做API自动化测试，你需要有一点点扣腚经验，至少保证能准确的ctrl+c ctrl+v

## 环境配置
- 下载安装
  * [Postman](https://www.getpostman.com/downloads/)
  * [Node.js](https://nodejs.org/en/)

- 命令行安装
  * newman
  ```
  npm install -g newman
  ```
  * jsonserver(非必须，本教程用做API服务器的mock，用法很多，自行[参考](https://github.com/typicode/json-server)
)
  ```
  npm install -g json-server
  ```

## 创建模拟被测环境
找到一个空目录(参考sample目录)，创建一个db.json的文件
```json
{
  "animals": [
  ]
}
```

启动模拟环境
```
json-server --watch db.json
```

## 教程说明
本教程会通过下面4个API请求，覆盖大部分用法
- 通过post请求创建一个动物
- 通过get获取所有的动物
- 通过get请求获取创建完的动物
- 通过delete请求删除既有的东西

自动化测试的执行需要3个主要输入文件
- 测试用例
- 测试数据
- 环境配置

本教程会教会你怎么创建并使用这3个文件

当然，报告也是必不可少的，如何生成报告也会在本教程中被覆盖

## 使用postman进行手工测试
自动化测试只是把手工测试的步骤自动化执行，so第一步，我们先来手工做一遍

打开postman，创建一个collection
![图](./pics/0010.png)
![图](./pics/0020.png)

创建一个请求:通过post请求创建一个动物
![图](./pics/0030.png)
![图](./pics/0040.png)

试一下
![图](./pics/0050.png)

结果如下图，返回201，表示正常，创建完的数据也加上了id并放回
![图](./pics/0060.png)

这个请求没啥问题，保存起来
![图](./pics/0070.png)
![图](./pics/0080.png)
![图](./pics/0090.png)

光发送请求是不够的，我们在增加一些验证，先对我们要做的验证做一点描述
![图](./pics/0100.png)
![图](./pics/0110.png)

接下来增加验证的代码，验证我们用(chai的assert模块)[https://www.chaijs.com/api/assert/]
![图](./pics/0120.png)
Tests框内的代码部分可以从下面黏贴，代码部分说明参考注释
```javascript
// 引入chai的assert模块作为验证用模块
const chai = require('chai');
const assert = chai.assert;
// 验证返回值，通常返回值可能有200，201，202，本例中3返回值为这3个中的一个即为pass，实际使用中可以按需修改
pm.test("验证返回值201", function () {
    pm.expect(pm.response.code).to.be.oneOf([200,201,202]);
});
// 验证返回值
pm.test("验证返回数据跟预想一致", function () {
  // 获取返回值
  const jsonData = pm.response.json();
  // 设置预想值
  const expectData = {
    name: 'first animal',
    type: 'cat',
    age: 2,
    likes: [
      'fish', 'mouse'
    ]
  };
  // 为方便后期排查，我们把预想值和实际值转换成string类型
  const actualstr = JSON.stringify(jsonData);
  const expectstr = JSON.stringify(expectData);
  // 比较预想值和实际值，当验证不通过事后输出参数3的字符串方便排查
  // 本例用的json结果的比较，jsonData里只需要包含所有的expectData即为通过，其他验证类型参考文档
  assert.deepInclude(jsonData, expectData, `acutal:${actualstr} | expect:${expectstr}`);
  // id为自增字段，此处保留id留作后面使用
  pm.globals.set("newid", jsonData.id);

});

```
