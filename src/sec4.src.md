# 信号

## 信号

信号提供一种在对象间通讯的方法。
信号在某事发生或者完成时发出。

编程出一个信号的步骤在下述描述。

1. 注册一个信号。
一个信号属于一个对象，所以这个注册是在对象的类初始化的函数中完成的。
2. 写一个信号处理者。
一个信号处理者是一个在信号发出时被调用的函数。
3. 将信号和处理者连接。
信号使用`g_connect_signal`或其相关函数被连接到处理者。
4. 发出信号。

步骤1和4在属于信号的对象中完成。
步骤3通常在对象外完成。

这些信号相关的过程很复杂并且它需要很久去解释所有的功能。
受限于这节内容，只能完成最少量的事去写一个简单的信号并且不必要很准确。
如果你需要一个准确的信息，参考GObject API手册。
有四个部分介绍了信号。

- [Type System Concepts -- signals](https://docs.gtk.org/gobject/concepts.html#signals)
- [Funcions (g\_signal\_XXX series)](https://docs.gtk.org/gobject/#functions)
- [Funcions Macros (g\_signal\_XXX series)](https://docs.gtk.org/gobject/#function_macros)
- [GObject Tutorial -- How to create and use signals](https://docs.gtk.org/gobject/tutorial.html#how-to-create-and-use-signals)

## 信号注册

在这节的一个例子是一个当除0发生时发出的信号。
首先，我们需要确定信号的名称。
信号名称包含字母，数字，划线（`-`）和下划线（`_`）。
名字的第一个字符必须是一个字母。
所以，字符串"div-by-zero"对于信号的名称很合适。

有四个函数用于注册信号。
我们将使用[`g_signal_new`](https://docs.gtk.org/gobject/func.signal_new.html)注册"div-by-zero"信号。

~~~C
guint
g_signal_new (const gchar *signal_name,
              GType itype,
              GSignalFlags signal_flags,
              guint class_offset,
              GSignalAccumulator accumulator,
              gpointer accu_data,
              GSignalCMarshaller c_marshaller,
              GType return_type,
              guint n_params,
              ...);
~~~

要解释每个参数需要费很大劲。
现在我只展示给你`g_signal_new`函数调用，从`tdouble.c`中调出的。
（译者注：译者对里面的注释也进行了翻译）

~~~C
t_double_signal =
g_signal_new ("div-by-zero",
              G_TYPE_FROM_CLASS (class),
              G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
              0 /* 类偏移。子类不能覆盖类处理者(默认处理者)。 */,
              NULL /* 累加器 */,
              NULL /* 累加器数据 */,
              NULL /* C装配器。g_cclosure_marshal_generic()将被使用 */,
              G_TYPE_NONE /* 返回值类型 */,
              0     /* n_params */
              );
~~~

- `t_double_signal`是一个静态guint变量。
guint类和unsigned int是一样的。
它被设定为由`g_signal_new`函数返回的信号id。
- 第二个参数是属于这个信号的对象的类型 (GType) 。
`G_TYPE_FROM_CLASS (class)`返回对应类的类型(`class`是指向对象的类的指针)。
- 第三个参数是一个信号标识。
要解释这个标识需要花费大量篇幅。
所以，我想在这列出来。
上面的参数可以用在很多场景中。
定义在[GObject API Reference -- SignalFlags](https://docs.gtk.org/gobject/flags.SignalFlags.html)中有描述。
- 返回类型是G_TYPE_NONE意味着没有值由信号处理者返回。
- `n_params`是参数数。
这个信号不含有参数，所以它是0。

这个函数位于类初始化函数中 (`t_double_class_init`)。

你可以使用其他函数例如`g_signal_newv`。
更多细节参阅[GObject API Reference](https://docs.gtk.org/gobject/func.signal_newv.html)。

## 信号处理者

信号处理者是当信号发出时调用的函数。
处理者有2个参数。

- 信号所属者的实例
- 在信号连接中提供的用户数据的指针。

"div-by-zero"信号不需要用户数据。

~~~C
void div_by_zero_cb (TDouble *self, gpointer user_data) { ... ... ...}
~~~

第一个参数`self`是发出信号的实例。
你可以省略第二个参数。

~~~C
void div_by_zero_cb (TDouble *self) { ... ... ...}
~~~

如果一个信号有参数，参数在实例和用户数据之间。
例如，在GtkApplication的"window-added"信号的处理者是：

~~~C
void window_added (GtkApplication* self, GtkWindow* window, gpointer user_data);
~~~

第二个参数`window`是信号的参数。
"window-added"信号在有新窗口加入应用时发出。
参数`window`指向一个新添加的窗口。
更多信息参见[GTK API reference](https://docs.gtk.org/gtk4/signal.Application.window-added.html)。

"div-by-zero"信号的处理者只是展示一个错误信息。

~~~C
static void
div_by_zero_cb (TDouble *self, gpointer user_data) {
  g_print ("\n错误：被0除\n\n");
}
~~~

## 信号连接

一个信号和一个处理者使用[`g_signal_connect`](https://docs.gtk.org/gobject/func.signal_connect.html)函数连接。

~~~C
g_signal_connect (self, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
~~~

- `self`是一个信号属有者的实例。
- 第二个参数是信号名。
- 第三个参数是信号处理者。
它必须由`G_CALLBACK`包裹。
- 最后一个参数是一个用户数据。
这个信号不需要一个用户数据，所以分配NULL。

## 信号释放

信号在对象上释放。
接下来是`tdouble.c`的一部分。

~~~C
TDouble *
t_double_div (TDouble *self, TDouble *other) {
... ... ...
  if ((! t_double_get_value (other, &value)))
    return NULL;
  else if (value == 0) {
    g_signal_emit (self, t_double_signal, 0);
    return NULL;
  }
  return t_double_new (self->value / value);
}
~~~

如果除数是0，将会释放信号。
[`g_signal_emit`](https://docs.gtk.org/gobject/func.signal_emit.html)含有三个参数。

- 第一个参数是释放信号的实例。
- 第二个参数是信号id。
信号id是函数`g_signal_new`的返回值。
- 第三个参数是一个细节。
"div-by-zero"信号不包含细节，所以这个参数是0。
细节不在这个章节阐述但是通常你可以使用0作为第三个参数。
如果你想知道更多细节，参考[GObject API Reference -- Signal Detail](https://docs.gtk.org/gobject/concepts.html#the-detail-argument)。

如果一个信号含有参数，它们是第四个和随后的参数。

## 例子

一个示例程序在[src/tdouble3](tdouble3)中。

tdouble.h

@@@include
tdouble3/tdouble.h
@@@

tdouble.c

@@@include
tdouble3/tdouble.c
@@@

main.c

@@@include
tdouble3/main.c
@@@

Change your current directory to src/tdouble3 and type as follows.

~~~
$ meson setup _build
$ ninja -C _build
~~~

Then, Executable file `tdouble` is created in the `_build` directory.
Execute it.

~~~
$ _build/tdouble
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error: division by zero.

-10.000000 = -10.000000
~~~

## Default signal handler

You may have thought that it was strange that the error message was set in `main.c`.
Indeed, the error happens in `tdouble.c` so the message should been managed by `tdouble.c` itself.
GObject system has a default signal handler that is set in the object itself.
A default signal handler is also called "default handler" or "object method handler".

You can set a default handler with `g_signal_new_class_handler`.

~~~c
guint
g_signal_new_class_handler (const gchar *signal_name,
                            GType itype,
                            GSignalFlags signal_flags,
                            GCallback class_handler, /*default signal handler */
                            GSignalAccumulator accumulator,
                            gpointer accu_data,
                            GSignalCMarshaller c_marshaller,
                            GType return_type,
                            guint n_params,
                            ...);
~~~

The difference from `g_signal_new` is the fourth parameter.
`g_signal_new` sets a default handler with the offset of the function pointer in the class structure.
If an object is derivable, it has its own class area, so you can set a default handler with `g_signal_new`.
But a final type object doesn't have its own class area, so it's impossible to set a default handler with `g_signal_new`.
That's the reason why we use `g_signal_new_class_handler`.

The C file `tdouble.c` is changed like this.
The function `div_by_zero_default_cb` is added and `g_signal_new_class_handler` replaces `g_signal_new`.
Default signal handler doesn't have `user_data` parameter.
A `user_data` parameter is set in the `g_signal_connect` family functions when a user connects their own signal handler to the signal.
Default signal handler is managed by the instance, not a user.
So no user data is given as an argument.

@@@include
tdouble4/tdouble.c div_by_zero_default_cb t_double_class_init
@@@

`g_signal_connect` and `div_by_zero_cb` are removed from `main.c`.

Compile and execute it.

~~~
$ cd src/tdouble4; _build/tdouble
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error: division by zero.

-10.000000 = -10.000000
~~~

The source file is in the directory [src/tdouble4](tdouble4).

If you want to connect your handler (user-provided handler) to the signal, you can still use `g_signal_connect`.
Add the following in `main.c`.

~~~C
static void
div_by_zero_cb (TDouble *self, gpointer user_data) {
  g_print ("\nError happens in main.c.\n");
}

int
main (int argc, char **argv) {
... ... ...
  g_signal_connect (d1, "div-by-zero", G_CALLBACK (div_by_zero_cb), NULL);
... ... ...
}
~~~

Then, both the user-provided handler and default handler are called when the signal is emitted.
Compile and execute it, then the following is shown on your display.

~~~
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error happens in main.c.

Error: division by zero.

-10.000000 = -10.000000
~~~

This tells us that the user-provided handler is called first, then the default handler is called.
If you want your handler called after the default handler, then you can use `g_signal_connect_after`.
Add the lines below to `main.c` again.

~~~C
static void
div_by_zero_after_cb (TDouble *self, gpointer user_data) {
  g_print ("\nError has happened in main.c and an error message has been displayed.\n");
}

int
main (int argc, char **argv) {
... ... ...
  g_signal_connect_after (d1, "div-by-zero", G_CALLBACK (div_by_zero_after_cb), NULL);
... ... ...
}
~~~

Compile and execute it, then:

~~~
10.000000 + 20.000000 = 30.000000
10.000000 - 20.000000 = -10.000000
10.000000 * 20.000000 = 200.000000
10.000000 / 20.000000 = 0.500000

Error happens in main.c.

Error: division by zero.

Error has happened in main.c and an error message has been displayed.

-10.000000 = -10.000000
~~~

The source files are in [src/tdouble5](tdouble5).

## Signal flag

The order that handlers are called is described in [GObject API Reference -- Sigmal emission](https://docs.gtk.org/gobject/concepts.html#signal-emission).

The order depends on the signal flag which is set in `g_signal_new` or `g_signal_new_class_handler`.
There are three flags which relate to the order of handlers' invocation.

- `G_SIGNAL_RUN_FIRST`: the default handler is called before any user provided handler.
- `G_SIGNAL_RUN_LAST`: the default handler is called after the normal user provided handler (not connected with `g_signal_connect_after`).
- `G_SIGNAL_RUN_CLEANUP`: the default handler is called after any user provided handler.

`G_SIGNAL_RUN_LAST` is the most appropriate in many cases.

Other signal flags are described in [GObject API Reference](https://docs.gtk.org/gobject/flags.SignalFlags.html).
