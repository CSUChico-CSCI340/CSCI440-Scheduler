# Writing a Scheduler
California State University - Chico

By Bryan Dixon

## Introduction
The purpose of this assignment is for you gain insight into how schedulers work on the system.

## Logistics
The only “hand-in” will be electronic. Any clarifications and revisions to the assignment will be modified here and announced to the class during class or via Canvas.

## Hand Out Instructions

I recommend you use an Ubuntu Linux virtual machine to complete this assignment. Alternatively, you can use the ecc-linux machines or your native Linux install.

Download this repository as a zip file and then extract it where you would like to store your project files. An example for downloading and extracting the zip file is below, assuming you are in your home directory (you may remove the main.zip file after unzipping it):

```bash
~$ wget https://github.com/CSUChico-CSCI340/CSCI440-Scheduler/archive/refs/heads/main.zip
~$ unzip main.zip
#Will now have folder CSCI440-Scheduler with files
$ cd CSCI440-Scheduler/
$ ls
LICENSE			images/			multilevelRR/		simpleRR/
README.md		multilevelFeedback/	simple/
```

As you can see there are 5 folders contained in this repo. The images folder you can ignore as this is a holder for the images in the README. The other four folders contain the starter API code and simulator to test your code for each of the four schedulers you will be writing in this assignment.

To start, you should add your name in a header comment at the top of each *schedule.c*.

You only need to make changes to *schedule.c*; however, if you would like to add declarations to the *schedule.h* file, you may do so. You should **not** modify any of the provided function prototypes in the header file. All corresponding function definitions (the actual implementation) must be in *schedule.c*. If you make changes to *schedule.h*, please include a header comment at the top of the file with your name and a brief note indicating how you modified the file.

Looking at each *schedule.c* file, you will see that it contains a rudimentary scheduler API to add a process, remove a process, and get the next process from your scheduler. It will be your job to figure out how each scheduler works and add the appropriate code to make the given scheduler schedule correctly.

I would recommend implementing these in the order discussed later in this document as to some extent they will build upon the concepts of the earlier schedule as they get more complicated.


## General Overview of Schedulers
Taken from Wikipedia[1]
In computer science, scheduling is the method by which threads, processes or data flows are given access to system resources (e.g. processor time, communications bandwidth). This is usually done to load balance and share system resources effectively or achieve a target quality of service.  The need for a scheduling algorithm arises from the requirement for most modern systems to perform multitasking (executing more than one process at a time) and multiplexing (transmit multiple data streams simultaneously across a single physical channel).

The scheduler is concerned mainly with:

* Throughput - The total number of processes that complete their execution per time unit.
* Latency, specifically:
	* Turnaround time - total time between submission of a process and its completion.
	* Response time - amount of time it takes from when a request was submitted until the first response is produced.
* Fairness - Equal CPU time to each process (or more generally appropriate times according to each process’ priority and workload).
* Waiting Time - The time the process remains in the ready queue.

In practice, these goals often conflict (e.g. throughput versus latency), thus a scheduler will implement a suitable compromise. Preference is given to any one of the above mentioned concerns depending upon the user’s needs and objectives. In real-time environments, such as embedded systems for automatic control in industry (for example robotics), the scheduler also must ensure that processes can meet deadlines; this is crucial for keeping the system stable. Scheduled tasks can also be distributed to remote devices across a network and managed through an administrative back end.



## Your Task
The task for this assignment is to implement the scheduler APIs provided to you in the schedule.c files for the following schedulers:

* **Simple** - A simple FCFS scheduler
    * FCFS (first come, first serve) is also known as a FIFO (first in, first out) scheduler.
    * Behaves like a queue: the first process added to the queue is the first to be removed and executed.
      
* **Simple Round Robin** - A simple Round Robin scheduler with a quanta of 4 time units.  	
    * Similar to the simple FCFS scheduler, but this one is sensitive to response time.
    * A scheduling quantum is also known as a time slice - each process runs for the given time slice before the scheduler switches to the next job. This process repeats until the job is finished.
      
* **Multi Level Round Robin** - A variant of a Multi-Level priority scheduler using Round Robin schedulers.
    * The first time the scheduler runs, it should start at the highest priority level (priority 0). Each subsequent time it runs, it should move to the next lower priority level, cycling through all levels in order. Use a global index variable to track which priority level the scheduler is currently processing.
    * The scheduler should: iterate through all priority levels in order (starting from the tracked index), skip any empty queues, and at each non-empty level, select the next process in that queue and schedule it for the appropriate time quantum.
    * Higher-priority levels should be assigned larger time quanta. Your implementation should match the quanta and number of priority levels shown in Figure 1.

Figure 1: Multi Level Round Robin Priority Scheduler

![MultiLevel Queue](https://raw.githubusercontent.com/CSUChico-CSCI340/CSCI340-Scheduler/master/images/multilevel.png "MultiLevel Queue")

* **Multi Level Feedback** - A Multi-Level priority scheduler with feedback.
    * This scheduler consists of three queues: a FCFS scheduler for the highest priority tasks and two round robin queues for lower priority tasks.
    * The highest priority is represented with 0, and higher numbers represent lower priorities. The scheduler should always check the highest priority queue first.
    * All high priority tasks (priority 0) should run to completion. All lower priority tasks (priority 1 and 2) are assigned a time quantum. Your implementation should mirror the number of priorities and implementation in Figure 2.
    * Each process maintains an age, which starts at 0 whenever it enters a queue. As long as a process stays in the same queue, its age increments over time. Age is the mechanism that provides feedback for the scheduler.
    * If a process has not been scheduled for 1000 time cycles, it should be promoted to the next higher-priority queue (its age should be reset to 0 when it is placed in the new queue).
    * In this implementation, once a process is promoted to the highest priority queue (priority 0), it remains there until it is completed.
      
Figure 2: Multi Level Feedback Priority Scheduler

![MultiLevel Feedback Queue](https://raw.githubusercontent.com/CSUChico-CSCI340/CSCI340-Scheduler/master/images/multilevelfeedback.png "MultiLevel Feedback Queue")


You are not allowed to import any libraries not already provided in the schedule.c file.

## Data Structures in C
As you can’t include any libraries for data structures, you will likely want to implement your own data structure to implement the FCFS scheduler. As such you will need to do this in C code. As an example, here is a simple implementation of a linked list in C using structs:

```C
#include <stdio.h>
#include <stdlib.h>

struct node {
	int value;
	struct node* next;
};

int main(){
	int i;
	/* This will be the unchanging first node */
	struct node* root;
	/* cur node for manipulating linked list */
	struct node* cur;
	/*
 	 * Now root points to a node struct
 	 * Dynamic allocation of memory the size of
 	 * a node in C (similar to new node in C++)
  	 */
	root = (struct node*) malloc( sizeof(struct node));
	cur = root;
	for(i=0; i<10;i++){
		cur->value = i;
		cur->next = (struct node*) malloc( sizeof(struct node));
		cur = cur->next;
	}
	//Create final values
	cur->value = 10;
	cur->next = NULL;
	/* Now let’s print it out */
	cur=root;
	while(cur){
		printf("%d ", cur->value);
		cur=cur->next;
	}
	printf("\n");
	/* Let’s free up memory */
	while(root){
		cur = root;
		root = root->next;
		free(cur); //like delete in C++
	}
}
```

## Checking Your Work
I have provided some tools to help you check your work.

* **Reference solution.** - I’ve included a reference output file with the expected solution for each scheduler you need to write. Your program should produce identical output to that of the ref.out file.
* You can:
    * use the provided Makefiles and compile your code with `make`
    * run the executable and redirect the output to your own output file, e.g. `./sim > mytest.out`
    * use `diff` or `vimdiff` to compare the ref.out file with your output file


## Hints

* Read the [CPU Scheduling](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf) and [Multi-Level Feedback Scheduling](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf) sections from the online textbook.

## Evaluation
Your solution will be tested agains the reference output. You will get full credit if your assignment reproduces the reference output. Grades for this assignment will be assigned as follows:

* 10% - Simple Scheduler working
* 20% - Simple Round Robin working
* 35% - Multi-Level Round Robin working
* 35% - Multi-Level Feedback working

So if you get all of them working you'll get 100%.

## Hand In Instructions
You only need to change the *schedule.c* files; you may have also made some additions to the *schedule.h* files. You need to upload the *schedule.c* and *schedule.h* files to the [INGInious submission pages](https://inginious.csuchico.edu/) to mark your completion time. There will be a INGInious submission for each of the different schedulers so make sure you submit to the correct scheduler submission.

## References
1. Wikipedia. “Scheduling (computing)”. Wikipedia, The Free Encyclopedia. 2012. http://en.wikipedia.org/wiki/Process_scheduler. Online; accessed 16-February-2014.
