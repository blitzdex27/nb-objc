
# Table of contents
<!-- vscode-markdown-toc -->
* 1. [dispatch_after](#dispatch_after)
* 2. [dispatch_barrrier_async](#dispatch_barrrier_async)
	* 2.1. [test](#test)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

##  1. <a name='dispatch_after'></a>dispatch_after

Using dispatch_time()

Using dispatch_walltime()

```c
dispatch_time_t getDispatchTimeByDate(NSDate *date)
{
 NSTimeInterval interval;
 double second, subsecond;
 struct timespec time;
 dispatch_time_t milestone;
 interval = [date timeIntervalSince1970];
 subsecond = modf(interval, &second);
 time.tv_sec = second;
 time.tv_nsec = subsecond * NSEC_PER_SEC;
 milestone = dispatch_walltime(&time, 0);
 return milestone;
} 
```

##  2. <a name='dispatch_barrrier_async'></a>dispatch_barrrier_async

```c
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk1_for_reading);
dispatch_async(queue, blk2_for_reading);
dispatch_async(queue, blk3_for_reading);
dispatch_barrier_async(queue, blk_for_writing);
dispatch_async(queue, blk4_for_reading);
dispatch_async(queue, blk5_for_reading);
dispatch_async(queue, blk6_for_reading);
dispatch_async(queue, blk7_for_reading); 
```

## dispatch_apply
Add a Block to a dispatch queue for a number of times, and then it waits until all the tasks are finished. 
```objc
dispatch_queue_t queue =
 dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(10, queue, ^(size_t index) {
 NSLog(@"%zu", index);
});
NSLog(@"done");
```

## dispatch_suspend/resume

```objc
dispatch_suspend(queue);
dispatch_resume(queue); 
```

## dispatch_semaphore

```objc
dispatch_queue_t queue =
 dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
NSMutableArray *array = [[NSMutableArray alloc] init]; 

for (int i = 0; i < 100000; ++i) {
 dispatch_async(queue, ^{

 dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER); 
  [array addObject:[NSNumber numberWithInt:i]]; 

dispatch_semaphore_signal(semaphore);
 });
} 

```

