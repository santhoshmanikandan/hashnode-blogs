## Understanding the need for Thread-Safe variables using a basic counter in Java

Parallelization is being done to use the number of cores available in servers and to reduce the time taken for execution. But if race condition isn't handled properly for required variables, it could create unexpected issues in our application.

First, let's start with a normal counter program and understand why it should be made thread-safe.

```
public class Main
{
	public static void main(String[] args) {
		BasicCounter counter = new BasicCounter(10);
		for (int i = 1; i <=10; i++) {
		    System.out.println("Count is " + counter.incrementAndGet());
		}
	}
```
**BasicCounter.java**
```
public class BasicCounter {
	int count = 0;
	int maxCount;

	public BasicCounter(int maxCount) {
		this.maxCount = maxCount;
	}

	public int incrementAndGet() {
		count++;
		if (count >= maxCount) {
			count = 0;
		}
		return count;
	}
}``` 

The output of the above program is 

>Count is 1<br>
>Count is 2<br>
>Count is 3<br>
>Count is 4<br>
>Count is 5<br>
>Count is 6<br>
>Count is 7<br>
>Count is 8<br>
>Count is 9<br>
>Count is 0<br>


Now let's try to parallelize the counting operation with ExecutorService and try to see the result


```
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadRun {

	private static final int THREADS = 3;
	static ExecutorService service = Executors.newFixedThreadPool(THREADS);
	private static CountDownLatch latch = new CountDownLatch(3);
	public static void main(String[] args) throws InterruptedException {
		BasicCounter counter = new BasicCounter(10);
		ThreadRun run = new ThreadRun();
		for (int i = 0; i< THREADS; i++) {
			run.submit("Thread-"+i, counter);
		}
		latch.await();
		System.out.println("Final count: " + counter.get());
	}

	void submit(String threadName, BasicCounter counter) {
		service.submit(new Runnable() {
			int noOfZeros = 0;
			@Override
			public void run() {
				for (int i = 1; i <=10000; i++) {
					if (counter.incrementAndGet() == 0l) {
						noOfZeros++;
					}
				}
				System.out.println("No of reset for " + threadName + " is " + noOfZeros);
				latch.countDown();
			}
		});
		
	}
}``` 

As all the count is reset whenever it reaches 10, the total number of reset for three threads should be (10000/10 * 3) = 3000.

**Sample outputs:**

*Try 1:*

>No of reset for Thread-0 is 1000<br>
>No of reset for Thread-1 is 1005<br>
>No of reset for Thread-2 is 1002<br>
>Final count: 5

*Try 2:*

>No of reset for Thread-0 is 1000<br>
>No of reset for Thread-1 is 999<br>
>No of reset for Thread-2 is 1001<br>
>Final count: 5

The obtained result above is not the expected as the count value in BasicCounter class is cached locally at the CPU level for optimization and few updates are missed or getting duplicated.  If one thread modifies its value the change might not reflect in the original one in the main memory instantly and this is affected by the  [Write Policy](https://en.wikipedia.org/wiki/CPU_cache#Write_policies) configuration.

The first expected approach is to stop caching the count variable at CPU and in JAVA, it could be done by using  [volatile keyword](https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html). But in our case, it won't be sufficient for our counter as it does two operations(increment and reset).

To make the whole increment and reset operation atomic, one can use either one of the following approaches

### Volatile + Synchronized
Using synchronized, we create a lock on the incrementAndGet method such that only one thread can execute the method at a time. One must make sure the operations that are done in synchronized is simpler so that we don't create a bottleneck in our application.

```
public class BasicCounter {
	volatile long count = 0;
	.....
	synchronized public long incrementAndGet() {
		count++;
		if (count >= maxCount) {
			count = 0;
		}
		return count;
	}
}
``` 

### AtomicLong
 [java.lang.concurrent](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html)  package provides few classes which does operations atomically.

Atomic long does update and check process to make sure the final value is expected one. So the operations that are specified to run atomically can be run n number of times until the expected output is reached and operations that are done inside atomic methods should be side-effect free.


The below code uses AtomicLong class to perform increment and reset operation atomically.

```
import java.util.concurrent.atomic.AtomicLong;

public class ConcurrentCounter {
	AtomicLong count = new AtomicLong(0);
	...
	public long incrementAndGet() {
		return count.updateAndGet(count -> (++count == maxCount) ? 0 : count);
	}
}
``` 

The caveat in making operations atomic is that it increases the time taken for execution and we should make sure to minimize the number of atomic operations in our application as those become a bottleneck in our application

To conclude, check whether an object will be accessed concurrently and make sure to make the variables declared in the object are thread-safe if the variable is representing a value whose consistency is critical.










