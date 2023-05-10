## Vite解析

### 编译过程

1. 首先会判断模块路径，如果是`import vue from 'vue'`这种，需要将后面的路径替换成`@/modules/vue`
2. 