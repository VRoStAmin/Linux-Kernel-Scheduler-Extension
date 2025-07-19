Mastorakhs Emmanouhl AM:csd5255
Making of the Highest Value First scheduler. 

Everything I do is mentioned, if I found something was wrong and why it was, then Ill mention it later. 

First of all I noticed that all scheduling policies have a different sched_class that determine the way that the processes are handled. 
We also see that the sched_classes list contains all these policies and their functions of each, and the order of them in the list determine the priority of those scheduling policies. 

So I start by introducing a new sched_class inside include/linux/sched.h, which will have the number 4 and will be defined as SCHED_HIGHEST.

After that, I create a new sched_class struct for the scheduler, this will contain function pointers to the functions of our scheduler. I understand that it works in a polymorphic way where there are functions and each scheduler defines how these functions work. Everything regarding these functions will be in kernel/sched_highest.c 

I will also include the sched_highest.c inside the sched.c. 

After defining the sched_class and adding the most obvious functions needed: 

enqueue_task
dequeue_task
pick_next_task
task_tick
get_rr_interval
check_preemt_curr
put_prev_task
set_curr_task

I move to the implementation of these functions first. 

For the enqueue_task: 

/* I ended up not going with this approach. 
It should add an appropriate task to our scheduler's runqueue. Reading the rq struct, I noticed that there are rq structs for each scheduler, like cfs_rq and rt_rq each having fields that have to do with how the rq is managed. I don't like the idea of copying ready code. Instead I will create a list inside the rq struct itself. This list will be initialized and used in my functions. If I see that anything else regarding the way I manage the run queue is needed (field, value, etc.) then I will add it in the runqueue struct, instead of creating a highest_rq (for example) 

(If I see that I need a lot of fields then I will create a highest_rq and it will be part of the runqueue). 
*/

Because I see that there are init functions that are being called in a for_each_possible_cpu loop in the sched_init function. I will instead create a highest_rq struct, add it to the rq struct and in the task struct and make an init function in the sched.c code. Then I will call that function in the same loop inside sched_init. It will just initialize a list that is going to be used for the tasks.

The enqueue function will: 
append the task to the list,
initialize and update fields in the task struct and in the rq where needed, 
update the value of the task before adding it to the queue. 

I thought about the preemption too, what I will do is get the value of the task that will get added, I will also get the value of the current task
If the current task has a lower value of the one that was just added, then I will call the resched_task with the current task passed, from what I read, I think that the resched_task will call the pick_next_task.  

I will also add two fields inside the task struct for future use, one called runtime and it will be the time the task will be running and one called value, it will be the value of the task based on the 
way we need to calculate it. I will also add a list_head field that is gonna be used as a node for the list in the runqueue. I will initialize the list inside the copy_process() in fork.c .I don't like also changing the code there but I could not find a better solution with the time I have. 

For the dequeue_task:
I will just dequeue the given task from the list. If I see that other changes need to be done I will do them as well.

For the pick_next_task: 
I will just iterate the list, find the one having the maximum value and return it. If no value then it will return NULL. 

For the task_tick: 
The task tick will just update the run time of the task given by just incrementing it, it will also keep updating its value. If the value gets bigger than the computation time of the task struct then it 
should send a signal to kill it.

For the calculating of the value I will use the counter of runtime and the jiffies as the current time. Because D1 and D2 are in seconds I will convert them to jiffies by multiplying them with the HZ function before computing the final value. The computation time will also be converted to jiffies with the function msecs_to_jiffies that is available inside the jiffies. 

I had some mistakes during the compilation which I fixed and It compiled successfully. 
For the testing of the scheduler I will use the system calls from assignment three, and I also found the sched_setscheduler() system call which I will use. The code for the test as well as the way I compile it (with the .h file) will be the same as assignment three. 

The sched_setscheduler will use the defined number I have for the scheduler #define SCHED_HIGHEST 4 in sched.h, there is sched_setscheduler inside the sched.c that calls the __sched_setscheduler and inside that I will add my policy so that it prevents the returning of -EINVAL if the policy is of SCHED_HIGHEST. In the __set_scheduler I will also add a check that will make the sched_class of &sched_highest_class 
if the policy is set to that. 

From what I understand, now I will be able to change the sched_class to be of the SCHED_HIGHEST and that means that it will call the functions that are in the sched_highest_class.













