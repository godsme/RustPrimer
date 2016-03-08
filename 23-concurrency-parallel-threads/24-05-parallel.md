## 24.5 并行
理论上并行和语言并没有什么关系，所以在理论上的并行方式，都可以尝试用Rust来实现。本小节不会详细全面地介绍具体的并行理论知识，只介绍用Rust如何来实现相关的并行模式。通常有两种方式把任务并行化：

    1. 把所有数据拆分成一批一批更小的数据，然后并行处理这些数据。
    2. 把一个任务拆分成多个可以并行执行的小任务。
    
理想的情况而言，第一种拆分数据的方式更好，这样并行执行的线程之间不用进行任何控制，从而能够最大化地提升效率。相对而言第二种方式则需要进行一些通信和控制，并行化的效率要稍微低一点。那么我们还是优先从简单的方式入手，然后再一步一步探究，直到第二种方式的实现。

在图形编程中，我们经常要处理归一化的问题： 即把一个范围内的值，转换到范围1内的值。比如把一个颜色值255归一后就是1。假设我们有一个表示颜色值的数组要进行归一，用非并行化的方式来处理非常简单，可以自行尝试。下面我们将采用并行化的方式来处理，在此之前，你可以先用之前介绍的知识尝试一下，因为接下来我们要用到的知识，都是已经介绍过的，所采用的并行化方式是第一种方式，把数组中的值同时分开给多个线程一起并行归一化处理，下面是一种简单实现：

```rust
use std::thread;

// 并行化处理的线程个数
const THREAD_COUNT: usize = 4;
// 待处理颜色数组元素个数
const COLOR_COUNT: usize = 11;
// 待处理颜色数组
static mut colors : [f32; COLOR_COUNT] = [1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0]; 

fn main() {

	let mut threads = vec!();

	// 创建4个新线程执行归一化处理
	for index in 0..THREAD_COUNT {
	    let new_thread = thread::spawn(move|| {

	    	let mut i = index;
	    	while i < COLOR_COUNT {
	    	    unsafe {
	    	    	// 对号入入座归一化颜色值
	    	    	colors[i] /= 255.0;
				}
				i += THREAD_COUNT;
	    	}
		});
		threads.push(new_thread);
	}

	// 等待线程执行完成
	for thread in threads {
	    thread.join().unwrap();
	}

	// 输出结果
	println!("result:");
	unsafe {
		for index in 0..COLOR_COUNT {
		    println!("{}", colors[index]);
		}
	}
}
```
运行结果：
```
result:
0.003921569
0.007843138
0.011764706
0.015686275
0.019607844
0.023529412
0.02745098
0.03137255
0.03529412
0.039215688
0.043137256
```
关于代码的说明参见代码注释，是不是很简单。其实上面的代码还有很多问题，比如用了可以修改的`static`变量，比如使用了`unsafe`代码块，比如直接使用了数组下标，比如换成`Map`就不能处理了。虽然有这些不足，但是还可以运行。从中，你也可以简单明了地理解Rust并行化处理实现。

针对上面的不足，你是否可以利用之前学习到的知识，继续完善它？ 这就是本章的最后一个练习题。为了更深入的加深对Rust并发编程的理解和实践，还安排了一个挑战任务：实现一个Rust版本的MapReduce模式。值得你挑战。