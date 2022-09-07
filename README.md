# 功能介绍

以一个编辑系统的数据保存操作为例，在调用接口保存前往往需要很多的校验操作，涉及用户确认与校验数据请求等很多步骤，这些功能需要定义很多的函数去实现，而基本上它们只在保存的这个流程中有意义。

临时函数定义到全局不算很大的问题，做好命名空间就可以了，主要是命名比较麻烦，对一些有取名障碍的开发者而言更是如此。

另一方面对流程来说增加步骤是很常见的需求，比如要多加几步校验，难免到最后命名会越来越随意，而这些步骤是通过函数名进行调用的，步骤间有较强的关联，如果需要调整顺序，比如先校验 B 再校验 A，就要调整一批调用，而除了这些函数名称后这些步骤可能真没什么关系。

本流程管理对象就是为了解决这些问题而存在的，把原来抽象的一个个步骤实体化纳入模块化管理，弱化步骤间的关系，当有顺序调整需求时，如果两个步骤内没有数据关联，直接移动代码块就可以了。

# 基本使用

```javascript
import Process from '@xaios/process-manage'

// 实例化流程并为之命名，参数非必须，可以忽略：this.handle_process = new Process
this.handle_process = new Process('测试流程')

// 添加一个步骤并为之命名，参数非必须，可以忽略：steps.Add()
// 单纯调用 Add 不调用 Todo，将不会有任何效果
this.handle_process.Add('步骤 1').Todo(next => {
  // Todo 接收一个函数，唯一参数 next 表示前往下一个步骤
  // 在调用 next 前流程都不会继续，所以步骤中可以进行很多复杂操作，比如接口请求或是等待用户操作
  next()
  // 调用时若所在步骤非实际步骤，调用将不会有任何效果，即步骤内重复 next 只有首次生效
  next()
})

// 第二参数默认为 true，步骤内发生错误时将跳过当前步骤继续执行，设为 false 后错误将导致中断流程
// 如果需要，Todo 接收的函数也可以使用 async
this.handle_process.Add('步骤 2', false).Todo(async next => {
  // next 的参数如果为 true，表示在进行上一步操作时，可以在此处继续前往上一步操作，比较少用到
  next(true)
}).Exit(() => {
  // Todo 后可以调用 Exit，传入一个无参函数，进行离开此步骤时的操作，比如清理一些临时数据
})

// 步骤定义完后，调用 Run 启动流程开始执行步骤
// 会重置当前步骤为第一步，如果某步骤内的逻辑正在执行中，不应该调用此函数，可能造成意外
// 步骤配置跟流程启动可以分开进行，但通常建议放在一起即配即用
this.handle_process.Run()
```

# 外部调用

在系统的任何一个位置都可以控制流程，但因为步骤是异步的，直接操作时需要注意步骤可能正在处理中。

```javascript
this.handle_process.Prev() // 上一个步骤
this.handle_process.Next() // 下一个步骤

this.handle_process.step   // 当前步骤的索引值
```

# 事件监听

```javascript
this.handle_process.$on('error', (e, step, step_name, process_name) => {
  // e             // 错误信息对象
  // step          // 发生错误的步骤索引
  // step_name     // 发生错误的步骤名称
  // process_name  // 发生错误的流程名称
})

this.handle_process.$on('update', step => {
  // step          // 更新后的步骤值，开始运行流程时不会触发事件
})
```
