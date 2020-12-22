宏写成 do {} while(0)

由于宏对外是表现为一个语句，但是内部可能是多个语句，为了避免这种差异产生预期外的情况
当宏后面加分号时可以不出错 liujianghu 2020 0723 birthday nongli 6.3
c语言文件

FILE是个文件类型，不能直接当字符串用
FILE* fp=fopen("filename","r+); read add and modify
为了避免读取空文件，应该while((ch=getc(fp))!=EOF)
fclose(fp)成功返回0，否则返回EOF，应该判断是否成功：if(fclose(fp)!=0) printf("Error in closing file %s\n", argv[1]); 磁盘已满，磁盘被移走，I/O错误都会fclose失败