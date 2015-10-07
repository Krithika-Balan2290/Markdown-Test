> Using the material that we have covered in Lectures 8, 9, and 10, explain why the broken program doesn't work. What concurrency-related problems are occuring in this program? If you see the program end in livelock, then describe what is happening with the threads. Why can't they make progress? If you see the program end in another way, such as getting to the point where it prints out the product ids but doesn't include all of them, explain why you think that happened. Note: If you start to add println statements to the Producers and Consumers, you may actually altar the behavior of the program! If you observe this, then also include a discussion on why that happens as well. In your answer, if you want to include snippets of code and/or output to explain what you are seeing, then do so. Use all of Markdown's capabilities to display what you need to explain the concurrency-related problems that you are observing.

When we ran the first version of the broken program, we got the output as below:

	Queue empty!
	Producer 5 Produced: Product<4> on iteration 0
	Producer 9 Produced: Product<0> on iteration 0
	Producer 4 Produced: Product<5> on iteration 0
	Producer 7 Produced: Product<0> on iteration 0
	Producer 1 Produced: Product<7> on iteration 0
	Producer 0 Produced: Product<8> on iteration 0
	Producer 0 Produced: Product<14> on iteration 1
	Producer 3 Produced: Product<6> on iteration 0
	Producer 2 Produced: Product<0> on iteration 0
	Producer 3 Produced: Product<16> on iteration 1
	Producer 8 Produced: Product<3> on iteration 0
	Producer 6 Produced: Product<2> on iteration 0
	Producer 2 Produced: Product<17> on iteration 1
	Producer 0 Produced: Product<15> on iteration 2
	Producer 1 Produced: Product<13> on iteration 1
	Producer 7 Produced: Product<12> on iteration 1
	Producer 5 Produced: Product<9> on iteration 1
	Producer 4 Produced: Product<11> on iteration 1
	Producer 9 Produced: Product<10> on iteration 1
	Too many items in the queue: 19!
	Consumer 0 Consumed: Product<0>
	Consumer 5 Consumed: Product<14>
	Consumer 9 Consumed: Product<8>
	Consumer 4 Consumed: Product<4>
	Consumer 1 Consumed: Product<4>
	Consumer 3 Consumed: Product<4>
	Consumer 7 Consumed: Product<0>
	Consumer 8 Consumed: Product<7>
	Consumer 6 Consumed: Product<5>
	Too many items in the queue: 17!
	Consumer 2 Consumed: Product<4>
	Consumer 6 Consumed: Product<12>
	Consumer 8 Consumed: Product<13>
	Consumer 7 Consumed: Product<15>
	Consumer 3 Consumed: Product<17>
	Consumer 1 Consumed: Product<2>
	Consumer 4 Consumed: Product<3>
	Consumer 9 Consumed: Product<16>
	Consumer 5 Consumed: Product<0>
	Consumer 0 Consumed: Product<6>
	Consumer 8 Consumed: Product<10>
	Consumer 6 Consumed: Product<11>
	Consumer 2 Consumed: Product<9>
	Queue empty!
	Queue empty!
	Queue empty!
	Queue empty!
	Queue empty!
	Queue empty!
	Queue empty!
	
At this point we had to use Ctrl+C to terminate the run. The existence of race conditions and livelocks can be seen in the output. For example, the Product <0> is produced three times by different producers.

	...
	Producer 9 Produced: Product<0> on iteration 0
	...
	Producer 7 Produced: Product<0> on iteration 0
	...
	Producer 2 Produced: Product<0> on iteration 0
	...

Also, product 4 is produced only once, but is consumed 4 times by four different consumers.

	...
	Consumer 4 Consumed: Product<4>
	Consumer 1 Consumed: Product<4>
	Consumer 3 Consumed: Product<4>
	...
	Consumer 2 Consumed: Product<4>
	...
	
These two instances clearly demonstrate race conditions. This is because multiple threads are reading or writing the shared object *queue* at the same time. In the first example, the threads *Producer <2>, Producer <7> and Producer <9>*, all start at the same time and create the first object in the queue. All three threads create *Product <0>*. The required functionality was that one thread must have created *Product <0>*. The second thread should have read that *Product <0>* and then subsequently created *Product <1>* and so on. The race condition that happened here was because all three threads read the queue at the same time and added *Product <0>* to it.

In the second example, *Consumer <1>, Consumer <2>, Consumer <3> and Consumer <4>* all read the object *queue* at the same time. Due to this, they all retrieved the object *Product <4>*. The remove(pos) method in Java linked list removes the node at the specified position, *pos*, and returns the item removed. In this case, when the four consumer threads call the remove() function at the same time, the current item at the top of the queue is *Product <4>*. All four consumers, therefore, consume the same product in the list. Because of this, it looks like the consumer threads consumed more products than what the producers produced. The producers have produced 19 products whereas the consumers have consumed 22.

Due to the race condition among the producers, all product numbers are not written into the queue. Because three consumers created *Product <0>*, the total number of products created by the producers decreases. We expect each of the 10 threads to create 20 different products giving a total of 200 products, but because of repetitions of products, the actual number of products created is less than 200. The race condition  among the consumers causes them to add multiple entries to the HashMap for one product on the queue. A livelock occurs in the end resulting in the produers not being able to write any more products to the queue and in the consumers not being able to retrieve any more objects from the queue. This halts the execution of the program since neither the producer threads nor the consumer threads are able to proceed and complete execution.

The race conditions are caused because there are no locks placed on the shared resource *queue* in the program. All producer and consumer thread are able to freely read and update the list *queue* at any time. Because of this, *Consumer <1>, Consumer <2>, Consumer <3> and Consumer <4>* were able to read the same product and *Producer <2>, Producer <7> and Producer <9>* were able to update the same object to the queue. The livelock is caused because the producer threads are not able to run because the consumer threads are running continuously. The main thread creates and starts 10 producer threads and then sleeps for a minute before creating and starting 10 consumer threads. The producer threads run until they realize that the size of the queue is greater than 10. In this program, since there is no lock on the queue, by the time the producers realize that the size of the queue has exceeded 10, the queue already conains 19 elements!

  ...
	Too many items in the queue: 19!
	
At this point, the producers stop creating objects and the consumers start up a second after execution starts and start consuming the objects. The 10 consumers atart up and consume all the objects in the queue and once the queue is empty, they wait for the producers to create more objects while constantly checking if the queue has any products. Even though the size of the queue changes, this change is never propogated to the producers function. Because of this, the procers never create more products to insert into  the queue and so, the consumers have nothing to read from the queue. Therefore, even though all the threads are running, none of them is making any progress which is the livelock occurring in this program.
