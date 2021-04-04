
 /*
 
 */
 // 不包括结尾的'\0'
// 注意参数是const char* 而不是void*
 int strlen(const char *s)
 {
     int n;
     for (n = 0; *s != '\0'; n++, s++)
         ;
     return n;
}
<!--more-->

    // 功能：获取字符串s中实际字符个数，不包括结尾的'\0'；
    // 如果实际个数 <= maxlen，则返回n，否则返回第二个参数。
    // 注意参数是const char* 而不是void*
    int strnlen(const char *s, size_t size)
{
    int n;
    for (n = 0; size > 0 && *s != '\0';size--,s++)
        n++;
    return n;
}
// 注意第二个参数是const char* 而不是char*
char* strcpy(char*dst,const char* src)
{
    char *ret = dst;
    for (; *src != '\0';)
    {
        *dst++ = *src++;
    }
    return ret;
}
char* strcat(char*dst, const char*src)
{
    int n = strlen(src);
    strcpy(dst+n, src);
    return dst;
}
char*strncpy(char*dst, const char*src,size_t size)
{
    size_t i;
    char *ret=dst;
    for (i = 0; i < size ;i++)
    {
        *dst++ = *src;
        if(*src!='\0')
            src++;

    }

    return dst;
}
void *memmove(void*dst,const void*src,size_t size)
{

    char *d = (char *)dst;
    const char *s = (const char *)src;
    if(s<d&&d-s<size)
    {
        s += size;
        d += size;
        while(size--)
        {
            *d-- = *s--;
        }
    }
        else {
            *d++ = *s++;
        }
        return dst;
}
int partition(int *a,int begin,int end)
{
    if(begin==end)
        return end;
    int x = a[end];
    int index = begin - 1;
    for (int i = begin; i < end;i++)
    {
        if(a[i]<=x)
        {
            index++;
            swap(a[i], a[index]);
        }

    }
    index++;
    swap(a[index], a[end]);
    return index;
}