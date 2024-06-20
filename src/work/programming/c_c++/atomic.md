# std::atomic memory_order 含义

主要参考并翻译自 [Memory model synchronization modes](https://gcc.gnu.org/wiki/Atomic/GCCMM/AtomicSync)

### Sequentially Consistent

严格的顺序执行，不知道如何描述，看例子

```
 -Thread 1-       -Thread 2-
 y = 1            if (x.load() == 2)
 x.store (2);        assert (y == 1)
```

上例中如果 T2 `if` 触发，即它**看到了** T1 的 `x.store(2)` 动作，那么可以保证 T1 已经完成了 y 的赋值

```
             a = 0
             y = 0
             b = 1
 -Thread 1-              -Thread 2-
 x = a.load()            while (y.load() != b)
 y.store (b)                ;
 while (a.load() == x)   a.store(1)
    ;
```

上例中，如果 T2 的 loop 结束，即它看到了 T1 对 y 的 store，则 T1 一定完成了 `x = a.load()`，而当 T2 完成 a 的 store 后，T1 的 loop 才会结束

```
 -Thread 1-       -Thread 2-                   -Thread 3-
 y.store (20);    if (x.load() == 10) {        if (y.load() == 10)
 x.store (10);      assert (y.load() == 20)      assert (x.load() == 10)
                    y.store (10)
                  }
```

上例的几个 assertion 均成立

### Relaxed

memory_order_seq_cst 模型提供的是 *happens-before* 约束，relaxed 则去除了这一约束

例子

```
-Thread 1-
y.store (20, memory_order_relaxed)
x.store (10, memory_order_relaxed)

-Thread 2-
if (x.load (memory_order_relaxed) == 10)
  {
    assert (y.load(memory_order_relaxed) == 20) /* assert A */
    y.store (10, memory_order_relaxed)
  }

-Thread 3-
if (y.load (memory_order_relaxed) == 10)
  assert (x.load(memory_order_relaxed) == 10) /* assert B */
```

这是刚才的例子，但在 relaxed 模型下两个 assertion 均可能 FAIL。例如当 T2 观测到 x == 10 时，T1 可能并未将 20 写到 y

但即使在 relaxed 的模型下，对**同一变量**的多次修改仍然是顺序的，例如

```
x = 0

-Thread 1-
x.store (1, memory_order_relaxed)
x.store (2, memory_order_relaxed)

-Thread 2-
y = x.load (memory_order_relaxed)
z = x.load (memory_order_relaxed)
assert (y <= z)
```

这个例子中的 assertion 永远不会 FAIL，如果在 T2 中 x load 到了 2，那么一定不会再 load 到 1

### Acquire/Release

是前面两个模型的混合，Acquire/Release 只对互相依赖的变量保证 *happens-before*，没太看懂原文的例子，参考 [Understanding `memory_order_acquire` and `memory_order_release` in C++11](https://stackoverflow.com/questions/59626494/understanding-memory-order-acquire-and-memory-order-release-in-c11)

memory_order_release 可以提交本线程在此之前的所有变更，而 acquire 到的线程一定能见到被 release 的变更。这里的变更包括非 atomic 变量，即当作 memory barrier 用

### Consume

比 Acquire/Release 更松的模型，Acquire/Release 下所有 load 之后的读写都不许被移动到 load 之前，即在 load 时强制同步，但 Consume 模式下只有依赖原子变量的读写才会保证在 load 之后

```
n = 0
m = 0

-Thread 1-
 n = 1
 m = 1
 p.store (&n, memory_order_release)

 -Thread 2-
 t = p.load (memory_order_acquire);
 assert( *t == 1 && m == 1 );

 -Thread 3-
 t = p.load (memory_order_consume);
 assert( *t == 1 && m == 1 );
```

T2 中的 assertion 一定 PASS，因为 Acquire load 一定会同步 `n = 1; m = 1`

T3 中的 assertion 可能 FAIL，因为 Consume load 只会同步 `n = 1`，不保证同步 m

