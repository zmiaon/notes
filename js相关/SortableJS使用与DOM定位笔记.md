# SortableJS 使用与 DOM 定位笔记

> 关联项目实践：`hospital-pc-frontend/src/components/DraggableTableSortable.vue`、`主动健康管理/hospital-pc-frontend/src/views/printServerManagement/SortFom.vue`

---

## ⭐ 核心原则：不能直接修改 Vue 项目的 DOM 节点

### 原理

Vue 是**数据驱动**的框架：界面 = `f(data)`。Vue 内部保存着**虚拟 DOM**（它认为自己上次渲染出的真实 DOM 结构），数据变化时通过 diff 新旧虚拟 DOM，对真实 DOM 打最小补丁。

任何绕过数据、直接对真实 DOM 的操作——`appendChild` / `insertBefore` / `removeChild` / `innerHTML` / 直接改 class——都会造成：

```
虚拟 DOM（Vue 的账面） ≠ 真实 DOM（实物）
```

### 失配的后果

| 症状                                                    | 说明                                              |
| ----------------------------------------------------- | ----------------------------------------------- |
| `NotFoundError: Failed to execute 'insertBefore' ...` | Vue 按旧账本移动节点，但节点已被外部挪走，参照关系断裂（最经典报错）            |
| 补丁错位                                                  | 下一次数据更新时，Vue 按错误的真实结构打补丁，行内容错乱、重复或丢失            |
| 状态贴错行                                                 | 选中态、hover 样式、`row-class-name` 等与行对象绑定的东西落到错误的行上 |
| 幽灵 bug                                                | 第一次可能"碰巧"显示正确，偏差一直潜伏，后续任意小改动随时引爆                |

### 正确姿势

- **永远改数据，让 Vue 重渲染**：`this.list.splice(...)` 而不是手动挪 `<tr>`
- **第三方库必须动 DOM 时**（如 SortableJS 拖拽收尾）：先**还原 DOM** 到 Vue 渲染时的样子，再通过数据更新把改动"正规"地走一遍——全程只有 Vue 一个"动 DOM 的人"，单一数据源：

```js
onEnd(evt) {
  const { oldIndex, newIndex } = evt
  if (oldIndex === newIndex) return
  // ① 还原 SortableJS 造成的 DOM 移动，把现场还给 Vue
  const item = evt.item, to = evt.to
  to.removeChild(item)
  to.insertBefore(item, to.children[oldIndex] || null)
  // ② 再用同样的下标改数据，由 Vue 重渲染出新的 DOM 顺序
  const rows = [...this.tableData]
  const [moved] = rows.splice(oldIndex, 1)
  rows.splice(newIndex, 0, moved)
  this.tableData = rows
}
```

> 参考：`DraggableTableSortable.vue` 的 `onEnd` 就是这个模式的完整范例——先 `removeChild + insertBefore` 还原，再 `splice` 移动数据。

### 例外边界

Vue **不管理**的纯浮层/临时元素（如旧手写拖拽时代挂在 `document.body` 上的拖拽影子节点）可以直接操作，但必须在自己的生命周期结束（`beforeDestroy`）时清理掉，防止泄漏。

---

## 一、SortableJS 基本用法

### 1. 安装与引入

```bash
npm install sortablejs --save
```

```js
import Sortable from 'sortablejs'
```

### 2. 核心 API

```js
// 创建实例：el 是"容器"，可拖项 = 容器的直接子元素（默认 draggable: '>*'）
const sortable = Sortable.create(el, options)

sortable.option('disabled', true)  // 运行时启停（常配 watch 做权限/状态响应式）
sortable.destroy()                 // 销毁实例、解绑监听（配 beforeDestroy）
```

**绑定对象的选择**：绑"装着可拖项的容器"。el-table 场景绑 tbody——它的直接子元素恰好一层排着所有 `<tr class="el-table__row">`：

```
div.el-table
└─ div.el-table__body-wrapper
   └─ <table class="el-table__body">
      └─ <tbody>   ← ★ Sortable.create 绑定它
         ├─ <tr class="el-table__row">   ← 可拖项（evt.item）
         └─ <tr> ...
```

> 空数据的"暂无数据"提示块在 tbody 外面，不占子元素位置，不干扰下标。

### 3. 常用 options 速查

| option | 说明 | 项目中的用法 |
| --- | --- | --- |
| `animation: 150` | 排序动画 ms | 开启，拖放有过渡效果 |
| `handle: '.xxx'` | 只允许从匹配选择器的元素**发起**拖拽 | `appointDraggableCol` 时传 `'.draggable-cell'`，限定从某列发起；不传 = 整行可拖 |
| `draggable: '>*'` | 哪些直接子元素算可拖项 | 默认即可（tbody 的直接子元素就是 tr） |
| `disabled` | 禁用拖拽 | 创建时传 `!this.draggable`，配合 `watch` + `option()` 响应式启停 |
| `filter: '.xxx'` | 容器内不可拖的元素 | 表格场景一般不用 |
| `ghostClass` | 拖拽中占位元素的 class | 需要自定义占位样式时用 |
| `chosenClass` | 被选中拖动元素的 class | 同上 |

### 4. onEnd 事件对象字段表

```js
Sortable.create(tbody, {
  onEnd(evt) { /* ... */ }
})
```

| 字段 | 指向 | 说明 |
| --- | --- | --- |
| `evt.item` | `<tr>` | 被拖的那一行元素 |
| `evt.to` | `<tbody>` | 拖放**目标容器**（= create 时绑定的容器） |
| `evt.from` | `<tbody>` | 拖拽**来源容器**；单列表场景 `from === to`，跨容器拖拽才不同 |
| `evt.oldIndex` | number | 被拖行在容器子元素中的**拖动前**排位 |
| `evt.newIndex` | number | **拖动后**排位（SortableJS 已把行物理移到新位置后的位置） |
| `evt.oldDraggableIndex / newDraggableIndex` | number | 只统计匹配 `draggable` 选择器元素的下标（有不可拖子元素时与上面不同） |

### 5. Vue2 + el-table 实战模板（项目沉淀）

```js
import Sortable from 'sortablejs'

export default {
  data: () => ({ tableData: [], sortable: null }),
  watch: {
    draggable(val) {                       // 拖拽开关响应式
      if (this.sortable) this.sortable.option('disabled', !val)
    }
  },
  beforeDestroy() {
    if (this.sortable) { this.sortable.destroy(); this.sortable = null }
  },
  methods: {
    setTableData(data) {                   // 外部灌数据入口
      this.tableData = data || []
      this.$nextTick(() => this.initSort())
    },
    initSort(attempt = 0) {
      this.$nextTick(() => {
        const tbody = this.$el.querySelector('.el-table__body-wrapper tbody')
        if (!tbody) {                      // 表格未渲染完 → 重试（上限 20 次 × 200ms）
          if (attempt < 20) setTimeout(() => this.initSort(attempt + 1), 200)
          return
        }
        if (this.sortable) {               // tbody 失联（被重建）→ 销毁重挂
          if (this.sortable.el && this.sortable.el.isConnected) {
            this.sortable.option('disabled', !this.draggable)
            return
          }
          this.sortable.destroy()
          this.sortable = null
        }
        this.sortable = Sortable.create(tbody, {
          animation: 150,
          disabled: !this.draggable,
          handle: this.appointDraggableCol ? '.draggable-cell' : null,
          onEnd(evt) { /* 见核心原则的还原 → 改数据 → emit 三步 */ }
        })
      })
    }
  }
}
```

要点回顾：

1. `$nextTick` 后再找 tbody，找不到定时重试（首次灌数据时 DOM 可能还没渲染完）
2. 每次数据重灌不需要重新 create——tbody 元素是持久复用的，只需确认实例还连着
3. 拖拽语义用 **splice 移动**（不是交换），与后端 `sort({id, previousId})` 的"移动"契约一致
4. emit 给页面时携带**移动后的数组 + 前后下标**，页面据此计算 `{id, previousId}`：`id = data[newIndex].id`，`previousId = data[newIndex-1].id`（newIndex 为 0 不传 = 置顶）

---

## 二、"数哥哥"求下标：previousElementSibling

### 1. 一对容易混淆的 API

| API | 遍历对象 | 陷阱 |
| --- | --- | --- |
| `previousSibling` | 上一个**节点**（含文本/注释节点） | HTML 源码里的换行缩进会产生空白文本节点，数出来的"哥哥"数量不稳定 ❌ |
| `previousElementSibling` | 上一个**元素节点** | 只数标签元素，稳定可靠 ✅ |

```html
<ul>
  <li>a</li><li>b</li>
</ul>
```

`<li>b</li>` 的 `previousSibling` 可能是文本节点 `"\n  "`（取决于源码格式），而 `previousElementSibling` 永远是 `<li>a</li>`。**求下标永远用 Element 系列。**

### 2. SortableJS 内部式的下标计算

SortableJS"知道"行下标的原理不是记忆，而是**每次从实时 DOM 现数**——数被拖元素前面有几个可拖的兄弟：

```js
function index(el, selector) {
  let i = 0
  while ((el = el.previousElementSibling)) {   // 逐个往前数"哥哥"
    if (!selector || el.matches(selector)) i++ // 只数可拖项（如 tr）
  }
  return i
}

// SortableJS 两个时机各数一次：
// onStart:  oldIndex = index(dragEl)   —— DOM 还没动，数出拖动前排位
// 拖动过程：光标每越过某个兄弟行的中点，就把 dragEl 物理移动一次（DOM 即状态）
// onEnd:    newIndex = index(dragEl)   —— DOM 已在新位置，重新数即新排位
```

因为 el-table 按 `tableData` 顺序 1:1 渲染 `<tr>`，**DOM 排位号 = 数组下标**，所以拿到的下标可以直接用于 `splice`。

### 3. 等价写法对比

```js
// 写法一：previousElementSibling 往前数（SortableJS 内部思路，边拖边数性能好）
function index(el) {
  let i = 0
  while ((el = el.previousElementSibling)) i++
  return i
}

// 写法二：转数组 indexOf（语义直白，一次性求值）
const idx = [...parent.children].indexOf(el)

// 写法三：Array.prototype.indexOf.call（类数组兼容老环境）
const idx2 = Array.prototype.indexOf.call(parent.children, el)
```

---

## 三、closest() 小知识

### 1. 作用与规则

`Element.closest(selectorString)`：从**当前元素自身**开始沿祖先向上查找，返回第一个匹配 CSS 选择器的元素，找不到返回 `null`。

```js
// 点击落在行的任何内部元素（td/span/img）时，找到它所属的行
const tr = event.target.closest('tr.el-table__row')
```

规则要点：

- **包含自身**：元素自己匹配也直接返回（不需要先 `parentNode` 再查）
- 只能从 `Element` 上调用（`document.closest` 不存在；文本节点也没有，需先取 `parentElement`）
- 参数必须是合法 CSS 选择器字符串，写错抛 `SyntaxError`
- 匹配失败返回 `null`（不是 `undefined`），判空用 `!tr`
- IE 不支持（本项目 Element UI / Vue2 场景面向现代浏览器，无碍）

### 2. 与 matches() 配合：事件委托的原理

`el.matches(selector)`：判断单个元素是否匹配选择器，不向上爬。

`closest` ≈ `matches` + 向上循环。SortableJS 内部的事件委托正是这么做的——只在容器（tbody）上挂**一个** mousedown 监听，按下时对 `event.target` 做 `closest` 判断：

```js
// 伪码：SortableJS 内部
container.addEventListener('mousedown', e => {
  // 传了 handle 时：必须落在 handle 元素内才启动拖拽
  if (options.handle && !e.target.closest(options.handle)) return
  const dragEl = e.target.closest(options.draggable)  // '>*' → 直接找最近的子元素
  if (!dragEl) return
  oldIndex = index(dragEl)   // ← 二章的"数哥哥"
  startDrag(dragEl)
})
```

这就是为什么我们在 `onEnd` 里能直接拿到"哪一行"——库内部用 closest 把点击目标（td/span/img）归位到了具体 `<tr>`。对比旧手写组件里 4 层 `parentNode` 爬升 + class 字符串解析 `row-#id-id` 的 ~60 行代码，`closest` 一行搞定且更健壮。

### 3. 常见坑

| 坑 | 说明 |
| --- | --- |
| 对文本节点调用 | 文本节点没有 `closest`，事件委托时 `e.target` 可能是文本节点？——不会，`e.target` 在 DOM 事件里永远是元素；但自己手动遍历文本节点时需先 `node.parentElement` |
| 选择器非法 | `el.closest('div>')` 抛 `SyntaxError`，动态拼接选择器时注意 |
| 忘记判 null | 未匹配返回 `null`，链式继续调用会 `TypeError` |
| 只查祖先不含 document | `closest` 不会匹配到 `document`/`documentElement` 之外的根，最远到 `<html>` |

---

## 附：本项目沉淀的两个可复用模式

1. **拖拽排序组件内核**（`DraggableTableSortable.vue` / `糖尿病管理 draggableTable.vue`）：`initSort` 重试挂载 + `onEnd` 三步（还原 DOM → splice 移动 → emit `{data, draggingRowIndex, dropRowIndex}`）
2. **后端排序契约换算**：`sort({id, previousId})` = 把 id 移到 previousId 之后；`previousId = 移动后新位置的上一行`；拖到首位不传 previousId（置顶）
