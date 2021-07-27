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
* 提供一个JSON块，然后返回一个可以查询的cJSON对象。完成后调用cJSON_Delete。 */
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

6.定义一些关于*Array*与*Object*的函数

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

/* 这些调用创建适当Type的cJSON项。 */
extern cJSON *cJSON_CreateNull(void);
extern cJSON *cJSON_CreateTrue(void);
extern cJSON *cJSON_CreateFalse(void);
extern cJSON *cJSON_CreateBool(int b);
extern cJSON *cJSON_CreateNumber(double num);
extern cJSON *cJSON_CreateString(const char *string);
extern cJSON *cJSON_CreateArray(void);
extern cJSON *cJSON_CreateObject(void);
 
/* 根据count创建数组。 */
extern cJSON *cJSON_CreateIntArray(const int *numbers,int count);
extern cJSON *cJSON_CreateFloatArray(const float *numbers,int count);
extern cJSON *cJSON_CreateDoubleArray(const double *numbers,int count);
extern cJSON *cJSON_CreateStringArray(const char **strings,int count);
 
/* 向指定的数组/对象追加项。 */
extern void cJSON_AddItemToArray(cJSON *array, cJSON *item);
extern void	cJSON_AddItemToObject(cJSON *object,const char *string,cJSON *item);
extern void	cJSON_AddItemToObjectCS(cJSON *object,const char *string,cJSON *item);	/* 当字符串确实是常量(例如，是一个文本，或者与常量一样好)，并且肯定会在cJSON对象中存活时，就使用这个方法 */
/* 将对项的引用追加到指定的数组/对象。当您想要将现有的cJSON添加到新的cJSON中，但又不想破坏现有的cJSON时，可以使用此方法。 */
extern void cJSON_AddItemReferenceToArray(cJSON *array, cJSON *item);
extern void	cJSON_AddItemReferenceToObject(cJSON *object,const char *string,cJSON *item);
 
/* 从数组/对象中删除/分离项。 */
extern cJSON *cJSON_DetachItemFromArray(cJSON *array,int which);
extern void   cJSON_DeleteItemFromArray(cJSON *array,int which);
extern cJSON *cJSON_DetachItemFromObject(cJSON *object,const char *string);
extern void   cJSON_DeleteItemFromObject(cJSON *object,const char *string);
	
/* 更新数组项 */
extern void cJSON_InsertItemInArray(cJSON *array,int which,cJSON *newitem);	/* 将已存在的项目向右移动。 */
extern void cJSON_ReplaceItemInArray(cJSON *array,int which,cJSON *newitem);
extern void cJSON_ReplaceItemInObject(cJSON *object,const char *string,cJSON *newitem);
 
/* 复制一个cJSON项目 */
extern cJSON *cJSON_Duplicate(cJSON *item,int recurse);
/* Duplicate将在需要释放的新内存中创建一个与传递的cJSON相同的新项目。
递归!=0，它将复制连接到该项的所有子元素。条目->next和->prev指针在从Duplicate返回时总是0。*/
 
/* ParseWithOpts允许您要求(并检查)JSON是否以null结尾，并检索指向解析后的最终字节的指针。 */
extern cJSON *cJSON_ParseWithOpts(const char *value,const char **return_parse_end,int require_null_terminated);
 
extern void cJSON_Minify(char *json);
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

