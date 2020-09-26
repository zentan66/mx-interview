## 介绍

- `MVC`指的是`Model-View-Controller`，即模型-视图-控制器。
  - 使用`MVC`的目的就是将模型与视图分离
  - `MVC`属于单向通信，必须通过`Controller`来承上启下，必须由控制器来获取数据，将结果返回前端，页面重新渲染
- `MVVM`指的是`Model-View-ViewModel`，即模型-视图-视图模型，「模型」指的是后端传递的数据，「视图」指的是所看到的页面，「视图模型」是`MVVM`的核心，连接`View`和`Model`的桥梁，实现`View`的变化会自动更新到`viewModel`中，`viewModel`中的变化也会自动显示在`view`上，是一种数据驱动视图的模型

## 区别

- MVC中的Controller在MVVM中演变成viewModel
- MVVm通过数据来显示视图，而不是通过节点操作
- MVVM主要解决了MVC中大量的DOM操作，使页面渲染性能降低，加载速度慢，影响用户体验的问题

