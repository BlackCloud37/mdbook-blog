# String Hash

- 4-16 个字符的短字符串
```
|               ns/op |                op/s |    err% |          ins/op |         bra/op |   miss% |     total | benchmark
|--------------------:|--------------------:|--------:|----------------:|---------------:|--------:|----------:|:----------
|       11,094,427.00 |               90.14 |    0.4% |  111,833,763.00 |  12,383,248.00 |    4.0% |      0.12 | `CRC32`
|        8,181,207.00 |              122.23 |    3.2% |   81,658,267.00 |  12,940,438.00 |    3.8% |      0.09 | `FNV`
|        5,397,684.00 |              185.26 |    0.4% |   56,646,296.00 |   9,432,868.00 |    5.3% |      0.06 | `Murmur2`
|        6,749,671.00 |              148.16 |    1.2% |   67,046,257.00 |   9,432,870.00 |    5.3% |      0.08 | `Murmur3_x86_32`
|       11,833,767.00 |               84.50 |    0.5% |  128,028,092.00 |   5,949,857.00 |    8.5% |      0.13 | `Murmur3_x86_128`
|        7,636,166.00 |              130.96 |    0.8% |   91,747,270.00 |   6,441,582.00 |    7.7% |      0.09 | `Murmur3_x64_128`
```

- ~200 个字符的中等长度字符串
```
|               ns/op |                op/s |    err% |          ins/op |         bra/op |   miss% |     total | benchmark
|--------------------:|--------------------:|--------:|----------------:|---------------:|--------:|----------:|:----------
|      134,931,464.00 |                7.41 |    0.0% |1,235,797,487.00 |  80,973,317.00 |    1.2% |      1.48 | `CRC32`
|      154,305,226.00 |                6.48 |    0.0% |1,250,489,432.00 | 179,927,192.00 |    0.6% |      1.70 | `FNV`
|       41,743,783.00 |               23.96 |    0.0% |  474,381,552.00 |  50,949,403.00 |    1.8% |      0.46 | `Murmur2`
|       44,541,974.00 |               22.45 |    0.1% |  446,744,071.00 |  51,629,173.00 |    1.8% |      0.49 | `Murmur3_x86_32`
|       44,402,780.00 |               22.52 |    0.1% |  461,885,844.00 |  16,677,849.00 |    5.3% |      0.49 | `Murmur3_x86_128`
|       26,934,988.00 |               37.13 |    0.2% |  307,733,470.00 |  16,584,743.00 |    5.3% |      0.30 | `Murmur3_x64_128`
```

