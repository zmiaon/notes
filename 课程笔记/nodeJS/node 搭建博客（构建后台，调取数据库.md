安装：
**cross-env**：Node.js生态里专门解决跨平台环境变量设置问题的工具包，核心作用是抹平Windows、macOS/Linux在命令行设置环境变量的语法差异，让你写一套npm脚本就能在所有系统上正常运行。
1. **为什么需要它？**  
    不同系统设置环境变量的命令完全不一样：
    
    - macOS/Linux/Unix系统：直接写 `NODE_ENV=production 执行命令`
    - Windows系统：必须写成 `set NODE_ENV=production && 执行命令`  
        直接用Unix风格的命令在Windows上会直接报错，导致跨平台项目的脚本需要写多套适配代码，维护成本很高。
2. 安装：`npm install --save-dev cross-env`
3. 使用：设置环境变量的命令前面加前缀cross-env，就能统一使用`NODE_ENV=production