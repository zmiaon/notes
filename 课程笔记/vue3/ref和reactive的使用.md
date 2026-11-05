1. 类型：ref 支持所有类型（基本类型+引用类型）
reactive仅支持引用类型（对象、数组、Map、Set)
2. 访问类型：reactive 直接访问；ref脚本内使用.value，模板中自动解包
3. 赋值响应式：
	reactive直接整个赋值，会丢失响应式