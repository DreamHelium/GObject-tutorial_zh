# GObject

## 类和实例

GObject实例使用`g_object_new`函数构造。
GObject不仅有实例还有类。

- 一个GObject类在第一次使用函数`g_object_new`时创建。
然后只存在一个GObject类。
- GObject实例只要在`g_object_new`使用时就会被创建。
所以，两个或更多的GObject实例可以同时存在。

广义上说，GObject意味着对象，包括其类和实例。
狭义上说，GObject是一个C结构的定义。

~~~C
typedef struct _GObject  GObject;
struct  _GObject
{
  GTypeInstance  g_type_instance;
  
  /*< private >*/
  guint          ref_count;  /* (atomic) */
  GData         *qdata;
};
~~~

`g_object_new`函数分配GObject结构体大小的内存，初始化内存并且返回这个内存的指针。
这块内存是一个GObject实例。

同样地，GObject类的内存也由`g_object_new`分配并且它的结构由GObjectClass定义。
接下来的内容是从`gobject.h`拖取的。
但是你现在不需要知道这个结构的细节。

~~~C
struct  _GObjectClass
{
  GTypeClass   g_type_class;

  /*< private >*/
  GSList      *construct_properties;

  /*< public >*/
  /* seldom overridden */
  GObject*   (*constructor)     (GType                  type,
                                 guint                  n_construct_properties,
                                 GObjectConstructParam *construct_properties);
  /* overridable methods */
  void       (*set_property)		(GObject        *object,
                                         guint           property_id,
                                         const GValue   *value,
                                         GParamSpec     *pspec);
  void       (*get_property)		(GObject        *object,
                                         guint           property_id,
                                         GValue         *value,
                                         GParamSpec     *pspec);
  void       (*dispose)			(GObject        *object);
  void       (*finalize)		(GObject        *object);
  /* seldom overridden */
  void       (*dispatch_properties_changed) (GObject      *object,
					     guint	   n_pspecs,
					     GParamSpec  **pspecs);
  /* signals */
  void	     (*notify)			(GObject	*object,
					 GParamSpec	*pspec);

  /* called when done constructing */
  void	     (*constructed)		(GObject	*object);

  /*< private >*/
  gsize		flags;

  gsize         n_construct_properties;

  gpointer pspecs;
  gsize n_pspecs;

  /* padding */
  gpointer	pdummy[3];
};
~~~

GObject的程序包含在GLib源文件中。
你可以从[GNOME下载页面](https://download.gnome.org/sources/glib/)中下载GLib源文件。

在GObject教程源中的[src/misc](misc)文件夹中有例式程序。
你可以通过以下方式编译它们：

~~~
$ cd src/misc
$ meson setup _build
$ ninja -C _build
~~~

其中的一个程序是`example1.c`。
其代码如下。

@@@include
misc/example1.c
@@@

- 5-6: `instance1`和`instance2`都是指向GObject实例的指针。
`class1`和`class2`指向这些实例的类。
- 8-11: 函数`g_object_new`创建一个GObject实例。
GObject实例是拥有GObject结构(`struct _GObject`)的一块内存。
参数`G_TYPE_OBJECT`是GObject的类型。
这个类型和一般的C语言类型如`char`或`int`不同。
GObject自有一套基础的*类型系统*。
每个类型比如GObject必须注册到这个类型系统中。
类型系统有一系列的函数用于注册类型。
如果其中一个函数被调用，类型系统决定对象`GType`的类型值然后返回给调用者。
`GType`在原作者的计算机上是一个无符号整数但是它与硬件相关。
`g_object_new`分配GObject般大小的内存然后将项层地址的指针返回。
在创建后，本程序展示实例的地址。
- 13-16: 宏`G_OBJECT_GET_CLASS`返回参数的类的指针。
因此，`class1`指向`instance1`的类，`class2`指向`instance2`的类。
本程序展示两个类的地址。
- 18-19: `g_object_unref`将在下个子部分中介绍。
它析构实例，释放内存。

现在，运行这个程序。

~~~
$ cd src/misc; _build/example1
The address of instance1 is 0x55895eaf7ad0
The address of instance2 is 0x55895eaf7af0
The address of the class of instance1 is 0x55895eaf7880
The address of the class of instance2 is 0x55895eaf7880
~~~

两个实例`instance1`和`instance2`的地址不一样。
每个实例有着自己的内存。
两个类`class1`和`class2`的地址一样。
两个GObject实例共享同一个类。

![类和实例](../image/class_instance.png){width=10cm height=7.5cm}

## 引用计数

GObject实例有它自己的内存。
它们在创建时由系统分配。
当它们变得无用时，内存必须被释放。
可是，我们怎么决定它是否是无用的呢？
GObject提供了引用计数机制来解决这问题。

一个实例被创建并且被另一个实例或者主程序应用。
那就是说，这个实例被引用了。
如果这个实例被A和B引用，那么这个引用的数字是2。
这个数字被称为*引用计数*.
让我们幻想一种类似于下面的情景： 

- A使用了`g_object_new`并且拥有了一个实例G。
A引用了G，所以G的引用计数是1。
- B也想用G。
B使用了`g_object_ref`并且增加了1点引用计数。
现在这个引用计数是2。
- A不再使用G。
A使用`g_object_unref`并且减少了1点引用计数。
现在这个引用计数是1。
- B不再使用G。
B使用`g_object_unref`并且减少了1点引用计数。
现在这个引用计数是0。
- 因为这个引用计数是0，G知道没人引用它。
G自己开始了终结化进程。
G消失然后这个内存被释放。

程序`example2.c`基于以上的情景。

@@@include
misc/example2.c
@@@

现在执行一下。

~~~
$ cd src/misc; _build/example2
bash: cd: src/misc: No such file or directory
Call g_object_new.
Reference count is 1.
Call g_object_ref.
Reference count is 2.
Call g_object_unref.
Reference count is 1.
Call g_object_unref.
Now the reference count is zero and the instance is destroyed.
The instance memories are possibly returned to the system.
Therefore, the access to the same address may cause a segmentation error.
~~~

`example2`展示了：

- `g_object_new`创建了一个GObject实例并且将其引用计数设为1.
- `g_object_ref`增加了1个引用计数。
- `g_object_unref`减少了1个引用计数。
如果引用计数减少到0, 实例自己析构。

## 初始化和析构进程

真正的GObject初始化和析构过程非常复杂。
接下来是没有讲述细节的简化描述。

初始化

1. 使用类型系统注册GObject类型。
这在函数`main`被调用前在GLib初始化过程中完成。
(如果编译器是gcc，那么`__attribute__ ((constructor))`被用来修饰初始化函数。
参考[GCC手册](https://gcc.gnu.org/onlinedocs/gcc-10.2.0/gcc/Common-Function-Attributes.html#Common-Function-Attributes).)
2. 分配空间给GObjectClass和GObject结构。
3. 初始化GObjectClass结构内存。
这块内存将是GObject的类。
4. 初始化GObject结构内存。
这块内存将是GObject实例。

这个初始化步骤在`g_object_new`函数被第一次调用时执行。
在第二次和后续的`g_object_new`调用中，它只执行两个步骤：(1) GObject结构的内存分配 (2) 初始化内存。
`g_object_new`返回指向这个实例的指针（分配给GObject结构的内存）。

析构

1. 析构GObject实例。实例的内存被释放。

GObject类型是一个静态类型。
静态类型永不破坏它的类。
所以即使破坏的实例是最后一个实例，类仍然存在。

当你写代码去定义一个GObject的子对象时，理解以上步骤很重要。
详细的步骤将在后续章节解释。
