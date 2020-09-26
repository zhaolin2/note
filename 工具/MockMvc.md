# MockMvc

 https://www.cnblogs.com/lyy-2016/p/6122144.html 

## SpringBoot基础之MockMvc单元测试

https://mp.weixin.qq.com/s/yqn4TKsFch5GT5ntX1uBTg

整个过程：
1、mockMvc.perform执行一个请求；
2、MockMvcRequestBuilders.get("/user/1")构造一个请求
3、ResultActions.andExpect添加执行完成后的断言
4、ResultActions.andDo添加一个结果处理器，表示要对结果做点什么事情，比如此处使用MockMvcResultHandlers.print()输出整个响应结果信息。
5、ResultActions.andReturn表示执行完成后返回相应的结果。

整个测试过程非常有规律：
1、准备测试环境
2、通过MockMvc执行请求
3.1、添加验证断言
3.2、添加结果处理器
3.3、得到MvcResult进行自定义断言/进行下一步的异步请求
4、卸载测试环境