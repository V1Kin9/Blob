# 背景

最近由于工作原因，需要移植一个基于ARM的MPU(memeory protection unit)机制的rt-thread的组件[MAL](https://github.com/V1Kin9/rt-thread-mal)
看到这个仓库的risc-v目录下是有一份代码，但尚未实现，一时心血来潮，想试着实现一个基于risc-v的rt-thread-mal

## 接口评估

看了一下ARM相关的MAL实现接口，它主要是需要实现以下三个接口:

```c
static struct rt_mpu_ops _mpu_ops =
{
    .init         = _mpu_init,
    .switch_table = _mpu_switch_table,
    .get_info     = _mpu_get_info
};
```

以上三个接口分别：

1. init: 对硬件能使用的最大的memory region作MPU的初始化
2. switch_table: 对每个thread的MAL table作切换
3. 获取当前thread的MAL属性

基于上述三点，需要硬件实现的主要是1和2；其中在risc-v架构中，若rt-thread整体运行在M-mode下，且不把PMP的pmpcfgx.L设置为1，则能够实现根据thread动态修改PMP的配置属性（权限与地址），因此从原理上来看是可行的。

## 注意事项

1. 由于risc-v与ARM不同，没有为PMP设置单独的使能控制位来关闭或使能全局的PMP，因此在动态切换PMP的配置的时候，需要首先把pmpcfgx.A设置为0，表示该PMP entry不启用，然后再作修改
2. 当PMP的pmpcfgx.A配置为NAPOT的时候，需注意pmpaddrx的地址需保证与2的若干次幂对齐；此时可能会放大或缩小thread的MAL配置表，因此在可以的情况下，尽量把pmpcfgx.A配置为TOR，尽管这样会利用的内存配置区域
