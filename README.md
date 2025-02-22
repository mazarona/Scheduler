
This project is part of [Udacity's Embedded Systems Advanced Nanodegree](https://github.com/mazarona/embedded-systems-advanced-nanodegree)

# Thesis Implementation

### 1. Define Macro
- In FreeRTOSConfig.h

![Screenshot](images/Pasted%20image%2020220923234008.png)

### 2. Define The New EDF List
- In tasks.c

![Screenshot](images/Pasted%20image%2020220923221401.png)

### 3. Initialize The New EDF List
- In tasks.c in `prvInitialiseTaskLists()`

![Screenshot](images/Pasted%20image%2020220923221925.png)

### 4. Modify The Method That Adds A Task To The Ready List
- In tasks.c

![Screenshot](images/Pasted%20image%2020220923224300.png)
- Note: When adding a new task using `vListInsert()` function it inserts this new `xStateListItem` node in the `xReadyTasksListEDF` list at a position according to the value inside the member variable `xStateListItem.xItemValue` in such a way so that the nodes inside the list are sorted in ascending order according to this value. So We should make `xItemValue` of each task node hold the task deadline


### 5. Modify TCB Struct
- In tasks.c

![Screenshot](images/Pasted%20image%2020220923224935.png)

### 6. Create A New Task Initialization Method
- In tasks.c

![Screenshot](images/Pasted%20image%2020220923231453.png)
![Screenshot](images/Pasted%20image%2020220923231417.png)

### 7. Modify Initialization of IDLE Task
- In tasks.c in `vTaskStartScheduler()`

![Screenshot](images/Pasted%20image%2020220923231857.png)
- Note: We will have to make sure that the IDLE tasks stays at the end of the EDF list. This is just initialization if we didn't do anything else when the system starts running for a while the IDLE task will eventually preempt other application tasks.
> Every time IDLE task executes (i.e. no other tasks are in the Ready List), it calls a method that increments its deadline in order to guarantee that IDLE task will remain in the last position of the Ready List.


### 8. Choose The Task At The Head Of The EDF List When Context Switching
- In tasks.c in `vTaskSwitchContext()`

![Screenshot](images/Pasted%20image%2020220923233123.png)
- Note:  All That this function `vTaskSwitchContext()` does is it selects the task that will run and assigns it to `pxCrruntTCB`


# Messing Changes In Thesis


### 1. Make Sure Idle Task Stays At The End Of The EDF List

- In tasks.c in `prvIdleTask()`

![Screenshot](images/Pasted%20image%2020220924023633.png)
- Note: `pxCurrentTCB` inside of the IDLE task code points to the IDLE task itself.
- Note: `xMaxTaskDeadLine` is a variable that holds the maximum deadline value of all created tasks. So The IDLE task will always be late from the maximum task deadline by `configINIT_IDLE_PERIOD` as it always updates with the current tick.
- In tasks.c 

![Screenshot](images/Pasted%20image%2020220924031208.png)
- In tasks.c in `xTaskPeriodicCreate()`

![Screenshot](images/Pasted%20image%2020220924031043.png)

### 2. Modifiy xTaskIncrementTick To Revaluate Awakened Tasks Deadline. And Insert It At The Right Place In EDF List
- In tasks.c in `xTaskIncrementTick()`

![Screenshot](images/Pasted%20image%2020220924025044.png)

### 3. Change The Logic Of When A Context Switch Should Happen.

- In tasks.c in `xTaskIncrementTick()`

![Screenshot](images/Pasted%20image%2020220924030036.png)
- Note: Instead of comparing priorities, I changed it to compare and see if the just awakened task has lower deadline than the current running task. If that happen then a context switch should take place. No need to worry about what node will the contextswitch method will choose since we have already modified it to choose the head node at the EDF list as said in the Thesis. All we had to do is to signal that a context switch needs to happen when a task of lower deadline value than the current running task awakens.

