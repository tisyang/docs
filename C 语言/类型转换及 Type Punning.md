类型转换及 Type Punning
=======================

将一个 `float` 类型的值转换为网络字节序，该如何做？

除了手动编码转换外，我们知道 `htonl` 是将一个32bit无符号整数转换为网络字节序，而 `float` 也是32bit，那么只要能将一个 `float` 类型的值转换为bit位相同的 `uint32` 值，之后再同样转换回 `float` 就应该可以了。

一般我们会想到利用指针强制类型转换来迫使编译器重新解释指针指向的内存：

    float hton_f(float value)
    {
        int tmp = htonl(*(unsigned int *)&value);
        return *(float *)&tmp;
    }

指针存储的是内存中的地址，地址上的内容由指针类型来解释。这种强制类型转换的技巧叫做 [Type punning][1] . 事实上，这种方法存在隐患。

>__Type punning__  
>A form of [pointer aliasing][2] where two pointers and refer to the same location in memory but represent that location as different types. The compiler will treat both "puns" as unrelated pointers. Type punning has the potential to cause dependency problems for any data accessed through both pointers.

多数时候，Type punning 并不会引起任何问题。虽然在 C 标准中它属于依赖实现的实现，但通常可以正常工作。[[参考*]][3]

当打开 GCC 的编译选项 `-fstrict_aliasing` （优化选项 `-O2` 等会默认打开这个选项）时，则有可能出现问题。编译器认定两种不同类型（`signed`/`unsigned`不影响，还有 `char *`、`void *` 是例外）的指针不会指向同一内存地址，它们之间也没有依赖关系，这种认定下编译器可以对程序进行优化，比如会将不同类型的指针进行操作的代码的顺序（编译的机器码）打乱。而如果这两种类型指针事实上指向同一位置，那么程序代码的执行结果可能会出乎意料（执行顺序跟代码中的顺序不一致）。这种规则也称之为 *strict aliasing rule* .

一个可以使用的解决方法是使用 `union` 来进行转换：

    float hton_f(float value)
    {
        union tagFTOI { 
            float f;
            unsigned int i;
        };
        ((union tagFTOI *)&value)->i = htonl(((union tagFTOI *)&value)->i);               
        return ((union tagFTOI *)&value)->f;
    }

除了这种指针写法外，还可以用值方法，就是临时定义一个 `union` 对象，然后进行赋值转换。

根据 C 标准，任何使用 `type punning` 的行为依赖编译器实现。那么根据“标准”来看，使用 `union` 也不是一定能解决问题。标准中规定，设置了 `union` 中的某个域的值，那么就应该在相同的域读回。

但是大多数编译器都支持使用 `union` 来进行 `type punning` 而不破坏 *strict aliasing rule* . 比如 GCC，GCC 文档中写道：

>The practice of reading from a different union member than the one most recently written to (called “type-punning”) is common. Even with -fstrict-aliasing, type-punning is allowed, provided the memory is accessed through the union type.

*strict aliasing rule* 有两个例外，`char *` 和 `void *` 指针，举个栗子，用代码实现 `float` 的绝对值函数：[[来源*][4]]

    float funky_float_abs (float *a)
    {
        float temp_float = *a;
        // valid, because it's a char pointer. These are special.
        unsigned char * temp = (unsigned char *) a;
        temp[3] &= 0x7f;
        return temp_float;
    }
    
    float funky_float_abs (float *a)
    {
        int temp_int i;
        float temp_float result;
        // Memcpy takes void pointers, so it will force aliasing as well.
        memcpy (&i, a, sizeof (int));
        i &= 0x7fffffff;
        memcpy (&result, &i, sizeof (int));
        return result;
    }

上面两个函数都可以正常工作而不违反 *strict aliasing rule* 规则。至于函数怎么实现 `float` 的绝对值，可以参考 `float` 通常是如何在计算机上存储的。

- - - - - -
**参考资料**：

1.  [Type punning.][1]
2.  [Type punning isn't funny: Using pointers to recast in C is bad.][3]
3.  ["What are the common undefined/unspecified behavior for C that you run into?][4]
4.  [Type-punning and strict-aliasing.][5]

[1]: https://en.wikipedia.org/wiki/Type_punning "Type punning -- Wikipedia"
[2]: https://http://en.wikipedia.org/wiki/Pointer_alias "Pointer alias -- Wikipedia"
[3]: http://www.cocoawithlove.com/2008/04/using-pointers-to-recast-in-c-is-bad.html "Type punning isn't funny: Using pointers to recast in C is bad."
[4]: http://stackoverflow.com/questions/98340/what-are-the-common-undefined-unspecified-behavior-for-c-that-you-run-into "What are the common undefined/unspecified behavior for C that you run into? -- Stackoverflow"
[5]: http://blog.qt.digia.com/blog/2011/06/10/type-punning-and-strict-aliasing/ "Type-punning and strict-aliasing"
