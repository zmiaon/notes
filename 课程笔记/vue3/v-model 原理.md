1 vue2 原生组件
`<input v-model="val">`
相当于：
`<input :value="val" @input='val = $event.target.value >`

2 vue2 自定义组件 
`<my-component v-model='val' />`
相当于：
`<my-component :value='val' @input='val = arguments[0]' />`  ---- 需要emit传值给父组件

3 表单元素
`<input type="checkbox" :checked='val' @change='' >` 

4 vue3 compile 以后的结果 (支持多个v-model)
`<my-component v-model='foo' />`
相当于：
```
h(Comp,{
modelValue:foo,
'onUpdate:modelValue':value => (foo = value)
})