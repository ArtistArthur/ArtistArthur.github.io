单`#`号意思是把其后面的宏参数字符串化，即两边加上引号，变为字符串：  
`#define PRINT_ERR(EXP) fprintf(stderr,"WARNING:"#EXP"\n");`当使用`PRINT_ERR(divder==0)`就会打印`"WARNING:divider==0\n"`    
`##`把两个token连成一个，或者说把两个字符连成一个：  
```cpp
#define PRINT_TRAP(n) printf("trap"#n" = %d\n",trap##n);
int trap10=10;
PRINT_TRAP(10);
```
结果会打印`"trap10=10\n"`  
<!--more-->

