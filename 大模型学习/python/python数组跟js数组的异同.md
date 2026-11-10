python数组
1. 截断：使用\[下标：下标]
2. 放入：使用append
3. 拼接：使用+

|操作|Python|JavaScript|
|---|---|---|
|取第一个|`lst[0]`|`arr[0]`|
|取最后一个|`lst[-1]`|`arr.at(-1)`|
|第 n 到 m（不含 m）|`lst[n:m]`|`arr.slice(n, m)`|
|从 n 到末尾|`lst[n:]`|`arr.slice(n)`|
|前 n 个|`lst[:n]`|`arr.slice(0, n)`|
|最后 n 个|`lst[-n:]`|`arr.slice(-n)`|
|步长|`lst[::2]`|无直接对应，用 `filter` / 循环|
|倒序|`lst[::-1]`|`arr.slice().reverse()`|
|浅拷贝|`lst[:]` 或 `lst.copy()`|`arr.slice()` 或 `[...arr]`|
|原地增删|`lst[i:i+1] = [...]` / `del`|`arr.splice()`|
