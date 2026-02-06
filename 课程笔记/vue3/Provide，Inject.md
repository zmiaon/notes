vue2,vue3都可以用的使用方法：

>provide('test':'3333')
>inject('test')

<mark class="hltr-yellow">但是这种方法传递的值没有响应式</mark>

响应式传递的方法：

>1.引入provide,inject方法
>2.在setup中使用
>3.provide('test','3333')

const test = inject('test')