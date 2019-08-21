﻿ #### 限制input输入框只能输入数字
```
import Vue from 'vue';
/**
 *
 * @param {element} el 标签
 * @param {object} binding 传入参数
 * @param {boolean} binding.limitLess100 是否控制小于100
 * @param {*} vnode
 * @param {*} oldVnode
 */
const addEvent = (el, binding, vnode, oldVnode) => {
    let inputEl = null;
    if (el.tagName.toLowerCase() === 'input') {
        inputEl = el;
    } else {
        inputEl = el.getElementsByTagName('input')[0];
    }

    if (!inputEl) {
        return;
    }

    inputEl.numberHandler = function(e) {
        e.target.value = e.target.value.replace(/\D/g, '') || null;
        if (binding && binding.value.limitLess100 && +e.target.value > 100) {
            e.target.value = null;
        }
    };

    inputEl.addEventListener('keyup', inputEl.numberHandler, false);
};

Vue.directive('input-number', {
    bind: addEvent,
    update: addEvent
});
```
