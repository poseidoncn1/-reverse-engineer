### Ring3切换到Ring0 指令
```
new： cpu提供 sysenter/sysexit
 old： 通过中断门 int 0x2e(windows)    int 0x80(linux)
 ```
 
 ### Ring3切换到Ring0 切换过程
 
 - 保存调用号到 eax寄存器   保存栈顶(esp)到 edx寄存器(Ring3)
 
 - cpu获取 内核寄存器环境
 ```
 rdmsr指令，读取MSR寄存器。wrmsr 写入msr寄存器。
 cpu读取MSR寄存器0x174 加载到CS寄存器；
读取MSR 0x174值+8 加载到SS寄存器；
读取 MSR 0x175值 加载到ESP寄存器
读取 MSR 0x176值 加载到EIP寄存器
```
- 参数传递

 
 
 
