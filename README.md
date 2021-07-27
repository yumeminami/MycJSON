# MycJSON
cJSON.h源码分析

1.头文件开头与结尾

```c++
#ifndef cJSON__h  
#define cJSON__h  
  
#ifdef __cplusplus  
extern "C"  
{  
#endif 
...

...

#ifdef __cplusplus  
}  
#endif  
  
#endif  
```

extern "C"的主要作用就是为了能够正确实现C++代码调用其他C语言代码。加上extern "C"后，会指示编译器这部分代码按C语言的进行编译，而不是C++的。由于C++支持函数重载，因此编译器编译函数的过程中会将函数的参数类型也加到编译后的代码中，而不仅仅是函数名；而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，一般之包括函数名。

2.定义*cJSON Types*

```C++
#define cJSON_False 0
#define cJSON_True 1
#define cJSON_NULL 2
#define cJSON_Number 3
#define cJSON_String 4
#define cJSON_Array 5
#define cJSON_Object 6
	
#define cJSON_IsReference 256
#define cJSON_StringIsConst 512
```

3.定义*cJSON Structure*

```c++
typedef struct cJSON {
	struct cJSON *next,*prev;	
	struct cJSON *child;		
 
	int type;			
 
	char *valuestring;	        
	int valueint;		        
	double valuedouble;	        
 
	char *string;		        
}cJSON;
```

4.定义*Memory management*函数

```C++
typedef struct cJSON_Hooks {
      void *(*malloc_fn)(size_t sz);
      void (*free_fn)(void *ptr);
} cJSON_Hooks;
 
/* 向cJSON提供malloc和free函数 */
extern void cJSON_InitHooks(cJSON_Hooks* hooks);
```

5.定义*cJSON_Parse()*解析和*cJSON_Print()*打印

```C++
/* 提供一个JSON块，然后返回一个可以查询的cJSON对象。完成后调用cJSON_Delete。 */
extern cJSON *cJSON_Parse(const char *value);
/* 将cJSON实体呈现为用于传输/存储的文本。完成后释放char*。 */
extern char  *cJSON_Print(cJSON *item);
/* 将cJSON实体呈现为用于传输/存储的文本，而不进行任何格式化。完成后释放char*。 */
extern char  *cJSON_PrintUnformatted(cJSON *item);
/* 使用缓冲策略将cJSON实体呈现为文本。预缓冲是对最终大小的猜测。猜测减少了重新分配。fmt=0表示无格式，=1表示有格式 */
extern char *cJSON_PrintBuffered(cJSON *item,int prebuffer,int fmt);
/* 删除一个cJSON实体和所有子实体。 */
extern void   cJSON_Delete(cJSON *c);
```

6.定义一些关于*Array*与*Object*的函数,以下只简单罗列，若有需求则到cJSON.h查找

```C++
/* 返回数组(或对象)中的项数。 */
extern int	  cJSON_GetArraySize(cJSON *array);
/* 从数组“数组”中检索项目编号“项目”。如果不成功，返回NULL。 */
extern cJSON *cJSON_GetArrayItem(cJSON *array,int item);
/* 从对象中获取项目"string"。不区分大小写。 */
extern cJSON *cJSON_GetObjectItem(cJSON *object,const char *string);
 
/* 用于分析失败的语法。这将返回一个指向解析错误的指针。
你可能需要回头看几个字才能理解它。当cJSON_Parse()返回0时定义。当cJSON_Parse()成功时为0。 */
extern const char *cJSON_GetErrorPtr(void);

```

7.最后定义宏

```C++
/* 用于快速创建内容的宏。 */
#define cJSON_AddNullToObject(object,name)		cJSON_AddItemToObject(object, name, cJSON_CreateNull())
#define cJSON_AddTrueToObject(object,name)		cJSON_AddItemToObject(object, name, cJSON_CreateTrue())
#define cJSON_AddFalseToObject(object,name)		cJSON_AddItemToObject(object, name, cJSON_CreateFalse())
#define cJSON_AddBoolToObject(object,name,b)	        cJSON_AddItemToObject(object, name, cJSON_CreateBool(b))
#define cJSON_AddNumberToObject(object,name,n)	        cJSON_AddItemToObject(object, name, cJSON_CreateNumber(n))
#define cJSON_AddStringToObject(object,name,s)	        cJSON_AddItemToObject(object, name, cJSON_CreateString(s))
 
/* 在分配整数值时，也需要将其传播到valuedouble。 */
#define cJSON_SetIntValue(object,val)			((object)?(object)->valueint=(object)->valuedouble=(val):(val))
#define cJSON_SetNumberValue(object,val)		((object)?(object)->valueint=(object)->valuedouble=(val):(val))

```

