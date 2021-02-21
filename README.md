# eigen3-multithread-arm-issue
This repository demonstrates a lock-like issue when running Eigen 3 dense matrix multiplication in parallel on ARM 32 or 64 bit.

## Compiling

### Pre-requisites

Clone eigen 3 from official repository
```
$ git clone https://gitlab.com/libeigen/eigen.git
```

My current version is the last commit right now:

```
$ git log -1
commit 73922b0174f84e78f486eb89de86dffc881acbcb (HEAD -> master, origin/master, origin/HEAD)
Date:   Sat Feb 20 18:56:42 2021 +0100
```

### Compiling on ARM 32 bits

```
$ g++ -march=native -mfpu=neon -mfloat-abi=hard -O3 -DNDEBUG -I eigen/ main.cpp -o test -lpthread
```

### Compiling on ARM 64 bits

```
$ g++ -O3 -DNDEBUG -I eigen/ main.cpp -o test -lpthread
```

### Compiling on Intel 64 bits

```
$ g++ -O3 -DNDEBUG -I eigen/ main.cpp -o test -lpthread
```

## Running

After compiling

```
$ ./test
```

It would run a single thread and output 1-30 rows with performance data. For example:

```
$ ./test 
1 eigen threads
t-0     0       59.2417 fps     1688 ms
...
```

After running using a single thread, you would like to run two or three threads for compare the performance. Thus, uncomment the lines in `main()` to include other threads:

```c++
std::thread t0(worker, "t-0");
//std::thread t1(worker, "t-1");
//std::thread t2(worker, "t-2");

t0.join();
//t1.join();
//t2.join();
```

and compile & run again in order to compare the performance of each individual thread in a multi-thread scenario.

## Results

In my experiments, runnning a single thread on Intel 64 bits results in ~1700ms per cycle:

```
$ ./test 
1 eigen threads
t-0     0       59.2417 fps     1688 ms
t-0     1       60.4961 fps     1653 ms
t-0     2       63.3312 fps     1579 ms
t-0     3       64.3087 fps     1555 ms
t-0     4       56.6893 fps     1764 ms
^C
```
On my intel processor, each thread has the same performance regardless I am running a single one, two or three. For example, if I run two threads I get again ~1600ms:

```
$ ./test 
1 eigen threads
t-1     0       62.8535 fps     1591 ms
t-0     0       62.3441 fps     1604 ms
t-1     1       62.2278 fps     1607 ms
t-0     1       62.5391 fps     1599 ms
t-0     2       58.3431 fps     1714 ms
t-1     2       57.0776 fps     1752 ms
t-0     3       58.9623 fps     1696 ms
t-1     3       59.6659 fps     1676 ms
^C
```

and even 3 threads give me nearly the same time per cycle:

```
$ ./test 
1 eigen threads
t-0     0       58.8582 fps     1699 ms
t-1     0       58.8235 fps     1700 ms
t-2     0       57.0451 fps     1753 ms
t-0     1       60.2047 fps     1661 ms
t-1     1       59.6659 fps     1676 ms
t-2     1       58.4795 fps     1710 ms
t-0     2       60.4961 fps     1653 ms
t-1     2       60.241 fps      1660 ms
t-2     2       59.9161 fps     1669 ms
^C
```

So, on Intel there is no problem at all. So far, I can reproduce the problem only on ARM processors.

To illustrate it, if I run 1 single thread in ARM 64 bits I get around 10.000ms per cycle:

```
$ ./test 
1 eigen threads
t-0     0       10.0371 fps     9963 ms
t-0     1       10.3445 fps     9667 ms
t-0     2       10.8707 fps     9199 ms
t-0     3       11.0681 fps     9035 ms
t-0     4       11.1632 fps     8958 ms
t-0     5       11.1495 fps     8969 ms
t-0     6       11.1707 fps     8952 ms
^C
```

Performance however significally drops when I try to run two threads:

```
$ ./test 
1 eigen threads
t-0     0       8.97827 fps     11138 ms
t-1     0       7.1803 fps      13927 ms
t-0     1       8.23723 fps     12140 ms
t-1     1       8.31601 fps     12025 ms
t-1     2       9.05141 fps     11048 ms
t-0     2       7.00918 fps     14267 ms
t-0     3       7.61209 fps     13137 ms
t-1     3       6.37633 fps     15683 ms
^C
```

And for 3 threads things get way worse:

```
$ ./test 
1 eigen threads
t-2     0       5.04821 fps     19809 ms
t-1     0       4.23621 fps     23606 ms
t-0     0       4.0868 fps      24469 ms
t-2     1       4.60936 fps     21695 ms
t-1     1       4.38808 fps     22789 ms
t-0     1       4.35047 fps     22986 ms
t-2     2       4.74631 fps     21069 ms
t-1     2       4.27204 fps     23408 ms
t-0     2       4.38231 fps     22819 ms
^C
```

On ARM 32 bit I have found similar performance issue. For one thread I got ~17000ms per cycle:

```
$ ./test 
1 eigen threads
t-0     0       5.88859 fps     16982 ms
t-0     1       5.90145 fps     16945 ms
t-0     2       5.81531 fps     17196 ms
t-0     3       5.2712 fps      18971 ms
^C
```

but for two threads I got ~21.000ms per thread:

```
$ ./test 
1 eigen threads
t-1     0       4.84848 fps     20625 ms
t-0     0       4.82812 fps     20712 ms
t-0     1       4.78744 fps     20888 ms
t-1     1       4.68209 fps     21358 ms
t-0     2       4.60575 fps     21712 ms
t-1     2       4.54938 fps     21981 ms
^C
```

and finally running 3 threads the result is really bad:

```
$ ./test 
1 eigen threads
t-2     0       2.95744 fps     33813 ms
t-1     0       2.59101 fps     38595 ms
t-0     0       2.38277 fps     41968 ms
t-0     1       2.89394 fps     34555 ms
t-2     1       2.3411 fps      42715 ms
t-1     1       2.46245 fps     40610 ms
^C
```

## Opened issue

I just opened an issue on Eigen official repository for this problem: https://gitlab.com/libeigen/eigen/-/issues/2159
