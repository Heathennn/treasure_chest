```
父组件.vue
<子组件 @change1="getArticle"></子组件>


子组件.vue
v-on='$listeners'直接把所有事件的回调也就是 function getArticle() {} 传递了下去
<孙组件 v-on='$listeners'></孙组件>


孙组件.vue
打印$listeners => change1: f () {}
$listeners.change1() 相当于直接调用了getArticle()
<template>
    <div @click="$listeners.change1()"></div>
</template>    

```