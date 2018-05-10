---
title: perf example
date: 2018-05-10 21:08
categories: perf
tags: 
- perf
---

```
agentzh
3月27日 02:00 来自 微博 weibo.com
测量整条命令的运行时间，还是用 perf stat -r N -d 命令比较靠谱和精确。
用 bash 的 time 命令一来系统误差较大（可能高达 2 毫秒），二来精度不足，只到毫秒精度。
当然，perf stat 还能输出很多其他有用的信息，比如 dcache 命中率还有 branch misprediction 比例等很多方面的 CPU 统计信息。
```

###### http://www.brendangregg.com/perf.html
###### https://www.ibm.com/developerworks/cn/linux/l-cn-perf1/index.html


```
  606  2018-05-10 20:31:22 apt-get install linux-tools-common
  609  2018-05-10 20:54:44 apt-get install linux-tools-3.13.0-32-generic linux-cloud-tools-3.13.0-32-generic
```

test.cc
gcc -o t1 -g test.cc 
perf stat -r 1 -d ./t1
```
 Performance counter stats for './t1':

        276.377556 task-clock (msec)         #    0.999 CPUs utilized          
                11 context-switches          #    0.040 K/sec                  
                 9 cpu-migrations            #    0.033 K/sec                  
               112 page-faults               #    0.405 K/sec                  
   <not supported> cycles                  
   <not supported> stalled-cycles-frontend 
   <not supported> stalled-cycles-backend  
   <not supported> instructions            
   <not supported> branches                
   <not supported> branch-misses           
   <not supported> L1-dcache-loads         
   <not supported> L1-dcache-load-misses   
   <not supported> LLC-loads               
   <not supported> LLC-load-misses         

       0.276712785 seconds time elapsed

```


time ./t1
```
real    0m0.278s
user    0m0.277s
sys     0m0.001s
```
``` c

void longa()
{
  int i,j;
  for(i = 0; i < 1000000; i++)
  j=i; //am I silly or crazy? I feel boring and desperate. 
}

void foo2()
{
  int i;
  for(i=0 ; i < 10; i++)
       longa();
}

void foo1()
{
  int i;
  for(i = 0; i< 100; i++)
     longa();
}

int main(void)
{
  foo1();
  foo2();
}

```
