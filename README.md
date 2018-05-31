# React Mobx 教程

* 目录  
简介  
React入门  
Mobx入门  

[本教程的地址: https://github.com/yaoang/react-mobx](https://github.com/yaoang/react-mobx)  

## 简介

Mobx是React使用的一套状态管理库，类似Redux，却相对简单易用。  
当你在对React路由间进行状态管理时，需要使用状态库以简化操作。  

## React入门

### React能干的事

React其实是一个UI层面的库，采用虚拟DOM方式，所以只是使用React而不加状态管理库的话是很麻烦的。

应该说，React也定义了一种开发规范。

### 如何快速生成React项目

1. 首先，为了更省事地生成React，最好安装一个工具——yo  
  ```
  npm install -g yo
  ```
2. 其次，安装yo的React-mobx生成器  
  ```
  npm install -g generator-mobx-react
  ```
3. 使用yo生成React-mobx项目  
  ```
  yo mobx-react  
  ```

  当然，其实并不是很支持使用这种方式创建项目，毕竟还是比较不灵活，需要修改的地方还是很多。
## Mobx入门

### Mobx的功能

主要功能是利用Mobx库进行React组件间状态(**state**)的管理。  并且达到动态更新组件的目的。

干货来了。看个简单的Simple。很快就能开发一个项目。

[ReactJS Mobx Simple Project](ReactJS_Mobx.md)

[ReactJS Unit Test](ReactJS_Test.md)

[Code Re-Factoring](Re-Factoring.md)