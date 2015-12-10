# C语言时间处理

标签（空格分隔）： 数据

---
## 不同时钟源的执行耗时
```
void clock_gettime_by_process_cpu_time(void)
{
	clockid_t clock = CLOCK_PROCESS_CPUTIME_ID;
	struct timespec res[10000] = {0};
	int count = 0;
	for (count = 0 ; count < 10000; count ++)
	{
		clock_gettime(clock, &res[count]);
	}
	long long sum = 0;
	for (count = 0; count < 9999; count ++)
	{
		sum += res[count+ 1].tv_nsec - res[count].tv_nsec;
		printf("差:%ldns\t", res[count+ 1].tv_nsec - res[count].tv_nsec);
	}
	printf("\n平均用时:%lfns\n", (sum / (double)9999));
}

void clock_gettime_by_thread_cpu_time(void)
{
	clockid_t clock = CLOCK_THREAD_CPUTIME_ID;
	struct timespec res[10000] = {0};
	int count = 0;
	for (count = 0 ; count < 10000; count ++)
	{
		clock_gettime(clock, &res[count]);
	}
	long long sum = 0;
	for (count = 0; count < 9999; count ++)
	{
		sum += res[count+ 1].tv_nsec - res[count].tv_nsec;
		printf("差:%ldns\t", res[count+ 1].tv_nsec - res[count].tv_nsec);
	}
	printf("\n平均用时:%lfns\n", (sum / (double)9999));
}

void clock_gettime_by_clock_real_time(void)
{
	clockid_t clock = CLOCK_REALTIME;
	struct timespec res[10000] = {0};
	int count = 0;
	for (count = 0 ; count < 10000; count ++)
	{
		clock_gettime(clock, &res[count]);
	}
	long long sum = 0;
	for (count = 0; count < 9999; count ++)
	{
		sum += res[count+ 1].tv_nsec - res[count].tv_nsec;
		printf("差:%ldns\t", res[count+ 1].tv_nsec - res[count].tv_nsec);
	}
	printf("\n平均用时:%lfns\n", (sum / (double)9999));
}

void clock_gettime_by_clock_monotonic(void)
{
	clockid_t clock = CLOCK_MONOTONIC;
	struct timespec res[10000] = {0};
	int count = 0;
	for (count = 0 ; count < 10000; count ++)
	{
		clock_gettime(clock, &res[count]);
	}
	long long sum = 0;
	for (count = 0; count < 9999; count ++)
	{
		sum += res[count+ 1].tv_nsec - res[count].tv_nsec;
		printf("差:%ldns\t", res[count+ 1].tv_nsec - res[count].tv_nsec);
	}
	printf("\n平均用时:%lfns\n", (sum / (double)9999));
}

void clock_gettime_by_clock_monotonic_raw(void)
{
	clockid_t clock = CLOCK_MONOTONIC_RAW;
	struct timespec res[10000] = {0};
	int count = 0;
	for (count = 0 ; count < 10000; count ++)
	{
		clock_gettime(clock, &res[count]);
	}
	long long sum = 0;
	for (count = 0; count < 9999; count ++)
	{
		sum += res[count+ 1].tv_nsec - res[count].tv_nsec;
		printf("差:%ldns\t", res[count+ 1].tv_nsec - res[count].tv_nsec);
	}
	printf("\n平均用时:%lfns\n", (sum / (double)9999));
}

void clock_gettime_by_gettimeofday(void)
{
	struct timeval res[100] = {0};
	int count = 0;
	for (count = 0; count < 100; count ++)
	{

		struct timespec start, end;
		clock_gettime(CLOCK_MONOTONIC, &start);
		gettimeofday(&res[count], NULL);
		clock_gettime(CLOCK_MONOTONIC, &end);
		printf("gettimeofday:sec:%ld\t\tnsec:%ld\n",  end.tv_sec - start.tv_sec, end.tv_nsec - start.tv_nsec);
	}
	long long sum = 0;
	for (count = 0; count < 99; count ++)
	{
		sum += res[count+ 1].tv_usec - res[count].tv_usec;
		printf("差:%ldms\t", res[count+ 1].tv_usec - res[count].tv_usec);
	}
	printf("\n平均用时:%lfms\n", (sum / (double)99));
}

```
| 时钟源        | 平均耗时（获取当前时间）   |  附注  |	初次耗时	|
| --------   | :-----  | :----  |
| CLOCK_PROCESS_CPUTIME_ID     | 189.481848ns |   处理器给每个进程的高精度时钟，只计算cpu执行时间，sleep等造成的时间不计算在内     | 884ns |
| CLOCK_THREAD_CPUTIME_ID	|   132.799380ns   |   处理器给每个线程的高精度时钟，只计算cpu执行时间，sleep等造成的时间不计算在内   | 596ns |
| CLOCK_REALTIME|    29.538554ns    |  系统级的真实时间，每个linux系统都会实现，应当优先使用，如果管理改变系统时钟，该值会受到影响	| 110ns |
| CLOCK_MONOTONIC        |    21.216922ns    |  任何进程都无法更改的单调递增时钟，表示自某个非特定起始点开始到现在的相见，即单调递增的相对时间  | 69ns |
| CLOCK_MONOTONIC_RAW        |    65.360036ns    |  类似CLOCK_MONOTONIC，不过该时钟源无法进行调整（对误差进行微调）  | 147ns |

附注：
>*  测试中，所有时钟源在初次使用的时候，执行时间都大于平均耗时，小者几倍耗时，大者数十倍耗时，具备一定的随机性，在大量使用中，也偶尔会出现部分耗时偏高的现象。
>* 进程和线程时钟源耗时偏高的原因应该为系统要为程序分配新的特殊时钟，存在cpu损耗，而其他时钟源则为使用已创建好的时钟，故而耗时相对较少。
>* gettimeofday获取时间的时间消耗与realtime时钟源差不多，能够保证微妙级以下，在无精度要求和特殊功能要求的情况下，可优先使用。

## 阻塞操作耗时

#### clock_nanosleep

```
	struct timespec res;
	res.tv_nsec = 10000000;
	res.tv_sec = 0;
	clockid_t clock = CLOCK_REALTIME;
	struct timespec start, end;
	int count = 0;
	long long sum = 0;
	for (count = 0; count < 100; count ++)
	{
		clock_gettime(clock, &start);
		clock_nanosleep(clock, 0, &res, NULL);
		clock_gettime(clock, &end);
		sum += end.tv_nsec - start.tv_nsec;
		printf("nanosleep:%ld\n", end.tv_nsec - start.tv_nsec);
	}
```

![enter image description here](http://7xjfjn.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20151210153419.png)

#### usleep
```
	clockid_t clock = CLOCK_REALTIME;
	struct timespec start, end;
	int count = 0;
	for (count = 0; count < 100; count ++)
	{
		clock_gettime(clock, &start);
		usleep(20000);
		clock_gettime(clock, &end);
		sum += end.tv_nsec - start.tv_nsec;
		printf("uslee sec:%ld\t", end.tv_sec - start.tv_sec);
		printf("ns:%ld\n", end.tv_nsec - start.tv_nsec);
	}
```
![enter image description here](http://7xjfjn.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20151210154800.png)

附注：
>* 无论是usleep还是nanosleep在进行sleep操作时都存在一定的指令耗时，用于切换cpu时间片和存储上下文等操作，该耗时一般为1微秒（百万分之一秒）左右。建议在进行sleep操作时，不要sleep低于1微秒的时间。
>* linux建议使用nanosleep | clock_nanosleep替换sleep和usleep，因为这两个函数是POSIX标准函数，不使用信号（信号可靠性相对较差），同时还能够在sleep中断后进行一定程度的恢复。
