# Python dict 与 JS object 对照

| JS | Python | 备注 |
|---|---|---|
| `obj[key] = v` | `d[key] = v` | 新增或覆盖，一条语法两种行为 |
| `Object.assign(o, x)` | `d.update(x)` | 批量合并 |
| 没有 `add` 方法 | 也没有 `dict.add()` | 误写会 AttributeError |
| `obj.key` 点访问 | 不支持，只能 `d["key"]` | 容易顺手写错 |

 **坑** ：`setdefault(k, v)` 在 key 已存在时不会覆盖，取值才用它，别拿来赋值。
