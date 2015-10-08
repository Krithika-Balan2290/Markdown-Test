###1
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
	
At this point, the producers stop creating objects and the consumers start up a second after execution starts and start consuming the objects. The 10 consumers start up and consume all the objects in the queue and once the queue is empty, they wait for the producers to create more objects while constantly checking if the queue has any products. The memory barrier caused by optimizations made by the compiler results in a situation where, even though the size of the queue changes, this change is never propogated to the producers thread. Because of this, the procers never create more products to insert into  the queue and so, the consumers have nothing to read from the queue. Therefore, even though all the threads are running, none of them is making any progress which is the livelock occurring in this program.

###2
>Now switch your attention to the broken2 program. The only difference between the two programs are the synchronized keywords on the methods contained in ProductionLine.java. For this question, explain why this approach to fixing the program failed. Why is it that synchronizing these methods is not enough. What interactions between the threads are still occurring that cause the program to not be able to produce the correct output? Again, you may use snippets of code and/or output to illustrate your points. The only requirement for this question is that you focus exclusively on issues related to why this particular approach fails to solve the problem. In other words, your answer to this question should be different than your answer to the question above where you are discussing the program and its concurrency problems in general.

Output of broken2:
	$ bin/runbroken
	Queue empty!
	Producer 1 Produced: Product<0> on iteration 0
	Producer 9 Produced: Product<8> on iteration 0
	Producer 6 Produced: Product<5> on iteration 0
	Producer 1 Produced: Product<9> on iteration 1
	Producer 4 Produced: Product<3> on iteration 0
	Producer 5 Produced: Product<4> on iteration 0
	Producer 4 Produced: Product<12> on iteration 1
	Producer 0 Produced: Product<0> on iteration 0
	Producer 8 Produced: Product<7> on iteration 0
	Producer 7 Produced: Product<6> on iteration 0
	Producer 3 Produced: Product<2> on iteration 0
	Producer 8 Produced: Product<16> on iteration 1
	Producer 0 Produced: Product<15> on iteration 1
	Producer 4 Produced: Product<14> on iteration 2
	Producer 5 Produced: Product<13> on iteration 1
	Producer 6 Produced: Product<10> on iteration 1
	Producer 1 Produced: Product<11> on iteration 2
	Producer 9 Produced: Product<9> on iteration 1
	Producer 2 Produced: Product<1> on iteration 0
	Too many items in the queue: 19!
	Too many items in the queue: 19!
	Consumer 0 Consumed: Product<0>
	Consumer 0 Consumed: Product<8>
	Consumer 0 Consumed: Product<9>
	Consumer 1 Consumed: Product<5>
	Consumer 6 Consumed: Product<196>
	Consumer 6 received done notification. Goodbye.
	0
	1
	2
	3
	4
	5
	6
	7
	8
	9
	10
	11
	12
	13
	14
	15
	16
	17
	18
	20
	21
	22
	23
	24
	25
	26
	27
	28
	29
	30
	31
	32
	33
	34
	35
	36
	37
	38
	39
	40
	41
	42
	43
	44
	45
	46
	47
	48
	49
	50
	51
	52
	53
	54
	55
	56
	57
	58
	59
	61
	62
	63
	64
	65
	66
	67
	68
	70
	71
	72
	73
	74
	75
	76
	77
	78
	79
	80
	81
	82
	83
	84
	85
	86
	88
	89
	90
	91
	92
	93
	94
	95
	96
	97
	98
	99
	100
	101
	102
	103
	105
	106
	107
	108
	109
	110
	111
	112
	113
	114
	115
	116
	117
	118
	119
	120
	121
	122
	123
	124
	125
	126
	127
	128
	129
	130
	131
	132
	133
	134
	135
	136
	137
	138
	139
	140
	141
	142
	143
	144
	145
	146
	147
	148
	149
	150
	151
	152
	153
	154
	155
	156
	157
	158
	159
	160
	161
	162
	163
	164
	165
	166
	167
	168
	169
	170
	171
	172
	173
	174
	175
	176
	177
	178
	179
	180
	181
	182
	183
	184
	185
	186
	187
	188
	189
	190
	191
	192
	193
	194
	195
	196

The synchronized keyword is used to prevent threads from interfering with each other and also to prevent memory inconsistency errors.
In the program broken 2, the methods by themselves are synchronized. But the entire object, which the method is accessing is not synchronized. This causes the producers and consumers to access the object at the same time this causes it to miss a few objects. 
So when the producer produces the product the consumer reads the product at the same time the producer produces another product which the consumer reads and consumes. This leads to gap in the queue. This is because the access to the methods have been synchronized and the access to the queue still remains unprotected. This leads to arbitrary access to the queue object from threads of either class.
As seen from the output 

	Producer 4 Produced: Product<196> on iteration 19
	Consumer 6 Consumed: Product<195>
	Producer 4 is done. Shutting down.
	Consumer 6 Consumed: Product<196>
	Consumer 6 received done notification. Goodbye.

We see that the last product consumed is Product<196> and consumer 6 consumes the product. But according to the problem statement there should be 199 products, the last 3 product are missing, this shows that even though the methods are synchronised, the access to the methods is not synchronised.

###3
> Now turn your attention to creating an implementation of the program that functions correctly in the fixed directory. In your answer to this question, you should discuss the approach you took to fix the problem and get your version of the program to generate output that is similar to the example_output.txt file that is included with the repo.

The main issues in the broken program were the race conditions and the livelock which happens due to the memory barriers between the producer and consumer threads. To fix the race condition, we need to make sure that only one thread is reading or updating a queue at any particular time. To do this, we need to sychronize the reads and writes to the queue among the producers and consumers. To do this, we put the block of code which reads from and writes to the queue in a synchronized code block as follows:

Producer:

    ...
    while (count < 20) { 
    	synchronized(queue) { 
    	  if (queue.size() < 10) {
    			  Product p = new Product();
    			  String msg = "Producer %d Produced: %s on iteration %d";
    			  System.out.println(String.format(msg, id, p, count));
    			  queue.append(p);
    			  count++;
    		  }
    	  }
    	}
    	...
    	
Consumer:

    ...
    while (true) {
      synchronized(queue) {
      	if (queue.size() > 0) {
      		Product p = queue.retrieve();
        	if (p.isDone()) {
          		String msg = "Consumer %d received done notification. Goodbye.";
          		System.out.println(String.format(msg, id));
          		return;
        	}
         	else {
          		products.put(p.id(), p);
          		String msg = "Consumer %d Consumed: %s";
          		System.out.println(String.format(msg, id, p));
          
          	}
    	}
      }
    }
    ...
    
Running the program with the above changes gives the output as below:

	...
		Consumer 8 Consumed: Product<151>
	Consumer 8 Consumed: Product<152>
	Consumer 7 Consumed: Product<153>
	Consumer 2 Consumed: Product<154>
	Consumer 2 Consumed: Product<155>
	Consumer 2 Consumed: Product<156>
	Consumer 2 Consumed: Product<157>
	Consumer 2 Consumed: Product<158>
	Consumer 2 Consumed: Product<159>
	Consumer 2 received done notification. Goodbye.
	Producer 9 Produced: Product<160> on iteration 13
	Producer 9 Produced: Product<161> on iteration 14
	Producer 9 Produced: Product<162> on iteration 15
	Producer 9 Produced: Product<163> on iteration 16
	Producer 9 Produced: Product<164> on iteration 17
	Producer 9 Produced: Product<165> on iteration 18
	Producer 9 Produced: Product<166> on iteration 19
	Producer 9 is done. Shutting down.
	Producer 2 Produced: Product<167> on iteration 3
	Producer 2 Produced: Product<168> on iteration 4
	Consumer 0 Consumed: Product<160>
	Consumer 0 Consumed: Product<161>
	Consumer 0 Consumed: Product<162>
	Consumer 0 Consumed: Product<163>
	Consumer 0 Consumed: Product<164>
	Consumer 0 Consumed: Product<165>
	Consumer 0 Consumed: Product<166>
	Consumer 0 received done notification. Goodbye.
	Consumer 5 Consumed: Product<167>
	Consumer 5 Consumed: Product<168>
	Producer 2 Produced: Product<169> on iteration 5
	Producer 2 Produced: Product<170> on iteration 6
	Producer 2 Produced: Product<171> on iteration 7
	Producer 2 Produced: Product<172> on iteration 8
	Producer 2 Produced: Product<173> on iteration 9
	Producer 2 Produced: Product<174> on iteration 10
	Producer 2 Produced: Product<175> on iteration 11
	Producer 2 Produced: Product<176> on iteration 12
	Producer 2 Produced: Product<177> on iteration 13
	Producer 2 Produced: Product<178> on iteration 14
	Consumer 8 Consumed: Product<169>
	Consumer 8 Consumed: Product<170>
	Consumer 8 Consumed: Product<171>
	Consumer 8 Consumed: Product<172>
	Consumer 8 Consumed: Product<173>
	Consumer 8 Consumed: Product<174>
	Consumer 7 Consumed: Product<175>
	Consumer 7 Consumed: Product<176>
	Consumer 7 Consumed: Product<177>
	Consumer 7 Consumed: Product<178>
	Producer 2 Produced: Product<179> on iteration 15
	Producer 2 Produced: Product<180> on iteration 16
	Producer 2 Produced: Product<181> on iteration 17
	Producer 2 Produced: Product<182> on iteration 18
	Producer 2 Produced: Product<183> on iteration 19
	Producer 0 Produced: Product<184> on iteration 17
	Producer 0 Produced: Product<185> on iteration 18
	Producer 2 is done. Shutting down.
	Producer 5 Produced: Product<186> on iteration 7
	Producer 5 Produced: Product<187> on iteration 8
	Consumer 8 Consumed: Product<179>
	Consumer 8 Consumed: Product<180>
	Consumer 8 Consumed: Product<181>
	Consumer 8 Consumed: Product<182>
	Consumer 8 Consumed: Product<183>
	Consumer 8 received done notification. Goodbye.
	Consumer 7 Consumed: Product<184>
	Consumer 7 Consumed: Product<185>
	Consumer 7 Consumed: Product<186>
	Consumer 7 Consumed: Product<187>
	Producer 0 Produced: Product<188> on iteration 19
	Producer 0 is done. Shutting down.
	Producer 5 Produced: Product<189> on iteration 9
	Producer 5 Produced: Product<189> on iteration 10
	Producer 5 Produced: Product<190> on iteration 11
	Producer 5 Produced: Product<191> on iteration 12
	Producer 5 Produced: Product<192> on iteration 13
	Producer 5 Produced: Product<193> on iteration 14
	Producer 5 Produced: Product<194> on iteration 15
	Producer 5 Produced: Product<195> on iteration 16
	Consumer 5 Consumed: Product<188>
	Consumer 5 received done notification. Goodbye.
	Consumer 7 Consumed: Product<189>
	Consumer 7 Consumed: Product<189>
	Consumer 7 Consumed: Product<190>
	Consumer 7 Consumed: Product<191>
	Consumer 7 Consumed: Product<192>
	Consumer 7 Consumed: Product<193>
	Consumer 7 Consumed: Product<194>
	Consumer 7 Consumed: Product<195>
	Producer 5 Produced: Product<196> on iteration 17
	Producer 5 Produced: Product<197> on iteration 18
	Producer 5 Produced: Product<198> on iteration 19
	Producer 5 is done. Shutting down.
	Consumer 7 Consumed: Product<196>
	Consumer 7 Consumed: Product<197>
	Consumer 7 Consumed: Product<198>
	Consumer 7 received done notification. Goodbye.
	0
	1
	2
	3
	4
	5
	6
	7
	8
	9
	10
	11
	12
	13
	14
	15
	16
	17
	18
	19
	20
	21
	22
	23
	24
	25
	26
	27
	28
	29
	30
	31
	32
	33
	34
	35
	36
	37
	38
	39
	40
	41
	42
	43
	44
	45
	46
	47
	48
	49
	50
	51
	52
	53
	54
	55
	56
	57
	58
	59
	60
	61
	62
	63
	64
	65
	66
	67
	68
	69
	70
	71
	72
	73
	74
	75
	76
	77
	78
	79
	80
	81
	82
	83
	84
	85
	86
	87
	88
	89
	90
	91
	92
	93
	94
	95
	96
	97
	98
	99
	100
	101
	102
	103
	104
	105
	106
	107
	108
	109
	110
	111
	112
	113
	114
	115
	116
	117
	118
	119
	120
	121
	122
	123
	124
	125
	126
	127
	128
	129
	130
	131
	132
	133
	134
	135
	136
	137
	138
	139
	140
	141
	142
	143
	144
	145
	146
	147
	148
	149
	150
	151
	152
	153
	154
	155
	156
	157
	158
	159
	160
	161
	162
	163
	164
	165
	166
	167
	168
	169
	170
	171
	172
	173
	174
	175
	176
	177
	178
	179
	180
	181
	182
	183
	184
	185
	186
	187
	188
	189
	190
	191
	192
	193
	194
	195
	196
	197
	198

While this program almost works as expected (*Producer <5>* created *Product <189>* twice and missed *Product <199>*. Because of this, the output only has numbers from 0 to 198), we can see that it is no longer concurent. The producer threads start running when the consumer threads stop and the consumer threads run when the producer threads stop. This is not exactly what we want to do. As an alternative, in the consumer threads, we can synchronize the queue access inside the *if* condition. This could still cause a problem. If the queue size is 1 and two consumer threads read the size at the same time and them try to retrive an product synchronously, it could possibly result in a thread reading the list out of bounds. To avoid this, we could make the consumer threads sleep for a random length of time before trying to obtain the size. So we now change the code to:

	import java.util.concurrent.ConcurrentHashMap;
	import java.util.Random;
	...
	    while (true) {
	      try {
    		  Thread.sleep(rand.nextInt(1000));
	      } catch (Exception ex) {
    		}
	     if (queue.size() > 0) {
    		  synchronized(queue) {
		      Product p = queue.retrieve();
        		if (p.isDone()) {
	        	 String msg = "Consumer %d received done notification. Goodbye.";
	        	 System.out.println(String.format(msg, id));
	          	 return;
        		}
			else {
        	  		products.put(p.id(), p);
          	  		String msg = "Consumer %d Consumed: %s";
          	  		System.out.println(String.format(msg, id, p));
	        	}
    		}
    		...
    		
This works perfectly fine and we get the output as below:

	...
	Queue Size: 10
	Consumer 5 received done notification. Goodbye.
	Producer 9 Produced: Product<168> on iteration 16
	Consumer 8 Consumed: Product<159>
	Producer 3 Produced: Product<169> on iteration 14
	Consumer 0 Consumed: Product<160>
	Producer 5 Produced: Product<170> on iteration 16
	Consumer 1 Consumed: Product<161>
	Producer 5 Produced: Product<171> on iteration 17
	Consumer 9 Consumed: Product<162>
	Producer 3 Produced: Product<172> on iteration 15
	Consumer 3 Consumed: Product<163>
	Producer 1 Produced: Product<173> on iteration 14
	Consumer 3 Consumed: Product<164>
	Producer 2 Produced: Product<174> on iteration 15
	Queue Size: 10
	Consumer 9 Consumed: Product<165>
	Producer 1 Produced: Product<175> on iteration 15
	Consumer 1 Consumed: Product<166>
	Producer 5 Produced: Product<176> on iteration 18
	Consumer 0 Consumed: Product<167>
	Producer 4 Produced: Product<177> on iteration 13
	Consumer 8 Consumed: Product<168>
	Producer 3 Produced: Product<178> on iteration 16
	Consumer 7 Consumed: Product<169>
	Producer 4 Produced: Product<179> on iteration 14
	Consumer 8 Consumed: Product<170>
	Producer 1 Produced: Product<180> on iteration 16
	Consumer 1 Consumed: Product<171>
	Producer 1 Produced: Product<181> on iteration 17
	Queue Size: 10
	Consumer 3 Consumed: Product<172>
	Producer 5 Produced: Product<182> on iteration 19
	Producer 5 is done. Shutting down.
	Consumer 0 Consumed: Product<173>
	Consumer 9 Consumed: Product<174>
	Producer 3 Produced: Product<183> on iteration 17
	Consumer 7 Consumed: Product<175>
	Producer 1 Produced: Product<184> on iteration 18
	Consumer 9 Consumed: Product<176>
	Producer 1 Produced: Product<185> on iteration 19
	Producer 1 is done. Shutting down.
	Consumer 1 Consumed: Product<177>
	Consumer 7 Consumed: Product<178>
	Producer 2 Produced: Product<186> on iteration 16
	Consumer 9 Consumed: Product<179>
	Producer 3 Produced: Product<187> on iteration 18
	Consumer 0 Consumed: Product<180>
	Producer 2 Produced: Product<188> on iteration 17
	Queue Size: 10
	Consumer 9 Consumed: Product<181>
	Producer 3 Produced: Product<189> on iteration 19
	Producer 3 is done. Shutting down.
	Consumer 3 Consumed: Product<182>
	Consumer 9 received done notification. Goodbye.
	Producer 9 Produced: Product<190> on iteration 17
	Consumer 3 Consumed: Product<183>
	Producer 2 Produced: Product<191> on iteration 18
	Consumer 7 Consumed: Product<184>
	Producer 9 Produced: Product<192> on iteration 18
	Consumer 8 Consumed: Product<185>
	Producer 4 Produced: Product<193> on iteration 15
	Queue Size: 10
	Consumer 8 received done notification. Goodbye.
	Producer 2 Produced: Product<194> on iteration 19
	Producer 2 is done. Shutting down.
	Consumer 1 Consumed: Product<186>
	Consumer 0 Consumed: Product<187>
	Producer 9 Produced: Product<195> on iteration 19
	Producer 9 is done. Shutting down.
	Consumer 0 Consumed: Product<188>
	Consumer 0 Consumed: Product<189>
	Producer 4 Produced: Product<196> on iteration 16
	Consumer 3 received done notification. Goodbye.
	Producer 4 Produced: Product<197> on iteration 17
	Consumer 7 Consumed: Product<190>
	Producer 4 Produced: Product<198> on iteration 18
	Queue Size: 10
	Consumer 0 Consumed: Product<191>
	Producer 4 Produced: Product<199> on iteration 19
	Producer 4 is done. Shutting down.
	Consumer 1 Consumed: Product<192>
	Queue Size: 10
	Consumer 0 Consumed: Product<193>
	Consumer 7 Consumed: Product<194>
	Consumer 7 received done notification. Goodbye.
	Consumer 1 Consumed: Product<195>
	Queue Size: 6
	Consumer 0 received done notification. Goodbye.
	Queue Size: 5
	Consumer 1 Consumed: Product<196>
	Consumer 1 Consumed: Product<197>
	Queue Size: 3
	Consumer 1 Consumed: Product<198>
	Consumer 1 Consumed: Product<199>
	Queue Size: 1
	Consumer 1 received done notification. Goodbye.
	0
	1
	2
	3
	4
	5
	6
	7
	8
	9
	10
	11
	12
	13
	14
	15
	16
	17
	18
	19
	20
	21
	22
	23
	24
	25
	26
	27
	28
	29
	30
	31
	32
	33
	34
	35
	36
	37
	38
	39
	40
	41
	42
	43
	44
	45
	46
	47
	48
	49
	50
	51
	52
	53
	54
	55
	56
	57
	58
	59
	60
	61
	62
	63
	64
	65
	66
	67
	68
	69
	70
	71
	72
	73
	74
	75
	76
	77
	78
	79
	80
	81
	82
	83
	84
	85
	86
	87
	88
	89
	90
	91
	92
	93
	94
	95
	96
	97
	98
	99
	100
	101
	102
	103
	104
	105
	106
	107
	108
	109
	110
	111
	112
	113
	114
	115
	116
	117
	118
	119
	120
	121
	122
	123
	124
	125
	126
	127
	128
	129
	130
	131
	132
	133
	134
	135
	136
	137
	138
	139
	140
	141
	142
	143
	144
	145
	146
	147
	148
	149
	150
	151
	152
	153
	154
	155
	156
	157
	158
	159
	160
	161
	162
	163
	164
	165
	166
	167
	168
	169
	170
	171
	172
	173
	174
	175
	176
	177
	178
	179
	180
	181
	182
	183
	184
	185
	186
	187
	188
	189
	190
	191
	192
	193
	194
	195
	196
	197
	198
	199

The first half of the output was not available, but clearly, the program completed, the runs of producers and consumers were concurrent and the output displays the numbers 0 to 199 in the correct order.
