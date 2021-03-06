2. 作用域
----------------

2.2. 静态函数和全局函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    使用静态函数, 尽量不要用裸的全局函数.

结论:

    定义在同一编译单元的函数, 被其他编译单元直接调用可能会引入不必要的耦合和链接时依赖.

    如果你必须定义全局函数, 又只是在 ``.cc`` 文件中使用它, 可使用 ``static`` 链接关键字 (如 ``static int Foo() {...}``) 限定其作用域.

2.3. 局部变量
~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    将函数变量尽可能置于最小作用域内, 并在变量声明时进行初始化.

C 最新标准(C99开始)允许在函数的任何位置声明变量. 我们提倡在尽可能小的作用域中声明变量, 离第一次使用越近越好. 这使得代码浏览者更容易定位变量声明的位置, 了解变量的类型和初始值. 特别是，应使用初始化的方式替代声明再赋值, 比如:

    .. code-block:: c

        int i;
        i = f(); // 坏——初始化和声明分离
        int j = g(); // 好——初始化时声明


注意, GCC 可正确实现了 ``for (int i = 0; i < 10; ++i)`` (``i`` 的作用域仅限 ``for`` 循环内), 所以其他 ``for`` 循环中可以重新使用 ``i``. 在 ``if`` 和 ``while`` 等语句中的作用域声明也是正确的, 如:

    .. code-block:: c

        while (const char* p = strchr(str, ‘/’)) str = p + 1;


2.4. 静态和全局变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    静态生存周期的对象，即包括了全局变量，静态变量和函数静态变量，都必须是原生数据类型 (POD : Plain Old Data): 即 int, char 和 float, 以及 POD 类型的指针、数组和结构体。

不允许用函数返回值来初始化 POD 变量，除非该函数不涉及（比如 getenv() 或 getpid()）不涉及任何全局变量。（函数作用域里的静态变量除外，毕竟它的初始化顺序是有明确定义的，而且只会在指令执行到它的声明那里才会发生。）

同理，全局和静态变量在程序中断时会被析构，无论所谓中断是从 ``main()`` 返回还是对 ``exit()`` 的调用。析构顺序正好与构造函数调用的顺序相反。但既然构造顺序未定义，那么析构顺序当然也就不定了。比如，在程序结束时某静态变量已经被析构了，但代码还在跑——比如其它线程——并试图访问它且失败；再比如，一个静态 string 变量也许会在一个引用了前者的其它变量析构之前被析构掉。

改善以上析构问题的办法之一是用 ``quick_exit()`` 来代替 ``exit()`` 并中断程序。它们的不同之处是前者不会执行任何析构，也不会执行 ``atexit()`` 所绑定的任何 handlers. 如果您想在执行 ``quick_exit()`` 来中断时执行某 handler（比如刷新 log），您可以把它绑定到 ``_at_quick_exit()``. 如果您想在 ``exit()`` 和 ``quick_exit()`` 都用上该 handler, 都绑定上去。

综上所述，我们只允许 POD 类型的静态变量.

.. note:: Yang.Y 译注:

    上文提及的静态变量泛指静态生存周期的对象, 包括: 全局变量, 静态变量.

读者笔记
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. 尽量不用全局函数和全局变量, 考虑作用域和命名空间限制, 尽量单独形成编译单元;
#. 作用域的使用, 除了考虑名称污染, 可读性之外, 主要是为降低耦合, 提高编译/执行效率.
#. 局部变量在声明的同时进行显式值初始化，比起隐式初始化再赋值的两步过程要高效，同时也贯彻了计算机体系结构重要的概念「局部性（locality）」。
#. 注意别在循环犯大量构造和析构的低级错误。
