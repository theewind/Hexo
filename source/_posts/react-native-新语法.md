layout: react
title: react native 新语法
date: 2016-06-20 13:12:08
tags: react native
---

最新的RN，已经全部转移到ES6，所以很多之前老的写法，会报错，比如
在react-native中引用React的做法发生了变更（在当前版本老的做法会报错）：

之前

	import React, { Component, View } from 'react-native';

现在

	import React, { Component } from 'react';
	import { View } from 'react-native';

具体有很多，可以参考[这篇文章](https://www.facebook.com/groups/reactnativeoss/permalink/1540818949548067/) 科学上网。

具体就是：

```
n React 0.14 for Web we started splitting up the React package into two packages `react` and `react-dom`. Now I'd like to make this consistent in React Native. The new package structure would be...
"react":
Children
Component
PropTypes
createElement
cloneElement
isValidElement
createClass
createFactory
createMixin
"react-native":
hasReactNativeInitialized
findNodeHandle
render
unmountComponentAtNode
unmountComponentAtNodeAndRemoveContainer
unstable_batchedUpdates
View
Text
ListView
...
and all the other native components.
So for a lot of components you actually have to import both packages.
var React = require('react');
var { View } = require('react-native');
var Foo = React.createClass({
render() { return <View />; }
});
However, for components that doesn't know anything about their rendering environment just need the `react` package as a dependency.
Currently a lot of these are accessible from both packages but we'd start issuing warnings if you use the wrong one.
This would be a little spammy so ideally we would have a simple codemod script that you can run on your imports to clean them up.
E.g. something that translates existing patterns like:
var React = require('react-native');
var { View } = React;
into:
var React = require('react');
var { View } = require('react-native');
If anyone wants to write and share that script with the community, that would be highly appreciated. We can start promoting it right now before we deprecate it.
```
