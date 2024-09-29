# Task-Scheduler
Assignment project on DSA that takes a list of tasks from the user as input and outputs an optimal schedule of tasks in order to maximize time according to utility calculations.

## Table of Contents

- [About the Project](#Appendix)
- [Built With](#Documentation)
- [Installation](#Installation)
- [Usage/Examples](#Usage/Examples)
- [Functionality](#Screenshots)
- [Acknowledgements](#Acknowledgments)


    
## About the Project

The objective of this project was to create a simple application focusing on practicality to aid users in organizing their day based on their tasks and the available time.


## Built With

* [Python](https://www.python.org/)

  
## Installation

In order to run this project, you need to: 

* Install the latest version of Python in (https://www.python.org/downloads/)

* Download the Source notebook and run OR go to GitHub Repository, get the URL and input: 
```bash
  git clone <(https://github.com/laryssacoe/Task-Scheduler.git)>
```

### Libraries needed 

The libraries included are built-in Python:

* random
* time
* matplotlib.pyplot
* numpy
* scipy.stats


## Usage/Examples

Demonstration of code snippet and its purpose. 

* Running the Scheduler case 1: 
Defines the dynamic for choosing a task to be performed based on whether there are fixed tasks or not in the total tasks of the user. 


```python  
def run_scheduler(self, starting_time, final_time=60*24):
"""
Prints the order of the schedule based on the input of the tasks and the cases. It runs the loop until all tasks have been done.

Parameters 
-----------
starting_time: int 
    Start of the day in minutes
final_time: int
    End of the day in minutes (default as midgnight)
"""

end_time = final_time
tot_utility = 0
current_time = starting_time
print("Running a simple scheduler:\n")


# checks if there are no unscheduled tasks and the heaps are still not empty
while self.check_unscheduled_tasks() or self.priority_queue.heapsize > 0 or self.fixed_priority_queue.heapsize > 0:

    self.get_tasks_ready() # inputs the tasks in the heaps if they are ready to be in progress

    # case 1: both fixed and priority heaps are not empty
    if self.priority_queue.heapsize > 0 and self.fixed_priority_queue.heapsize > 0:

	task_fixed = self.fixed_priority_queue.heap[0]
	task_regular = self.priority_queue.heap[0]
	alloted_duration = task_fixed.fixed_hour - current_time

	# case 1.1: for which the current time is equal the first task in fixed_hour heap
	# carry out with the fixed task
	if current_time == task_fixed.fixed_hour:  

	    task = self.fixed_priority_queue.heappop()    
	    print(f"🕰t={self.time_conversion(current_time)}")
	    print(f"\t started '{task.description}' for {task.duration} mins...")
	    tot_utility += task.priority  
	    current_time += task.duration
	    print(f"\t✅ t={self.time_conversion(current_time)}, task completed with {task.priority} utils!") 
	    self.remove_dependency(task.id) 
	    task.status = self.COMPLETED

	# case 1.2: for which doing the first priority task will not surpass the fixed hour task 
	# carry out with first task in priority queue
	elif current_time + task_regular.duration < alloted_duration: 

	    if current_time + task_regular.duration >= end_time: 
		print(f"🕰t={self.time_conversion(current_time)}")
		print(f"\t 😔 Sorry, you did not complete all the planned tasks for today in the alloted time, the final time was {end_time//60}h{end_time%60:02d} and the total utility is {tot_utility} utils.")
		print(f"\t The tasks still missing are: ")
		
		i = 1
		for algo in self.priority_queue.heap: 
		    print(f"\t \t {i}. '{algo.description}', task id = {algo.id}") 
		    i += 1
		break

	    task = self.priority_queue.heappop()    
	    print(f"🕰t={self.time_conversion(current_time)}")
	    print(f"\tstarted '{task.description}' for {task.duration} mins...")
	    tot_utility += task.priority  
	    current_time += task.duration
	    print(f"\t✅ t={self.time_conversion(current_time)}, task completed with {task.priority} utils!") 
	    self.remove_dependency(task.id) 
	    task.status = self.COMPLETED

	# case 1.3: for which doing the first in priority queue would surpass the time of the fixed hour
	else: 
	    
	    # searches if there is another (other than firts-priority) task of high priority 
	    # that can be done in between this time
	    boolean = self.search_heap(alloted_duration)

	    if boolean is True: # if so, increases the other task priority and carries out with task

		if current_time + self.priority_queue.heap[0].duration >= end_time: 
		    print(f"🕰t={self.time_conversion(current_time)}")
		    print(f"\t 😔 Sorry, you did not complete all the planned tasks for today in the alloted time, the final time was {end_time//60}h{end_time%60:02d} and the total utility is {tot_utility} utils.")
		    print(f"\t The tasks still missing are: ")
			    
		    i = 1
		    for algo in self.priority_queue.heap: 
			print(f"\t \t {i}. '{algo.description}', task id = {algo.id}") 
			i += 1
		    break

		task = self.priority_queue.heappop()   
		print(f"🕰t={self.time_conversion(current_time)}")
		print(f"\tstarted '{task.description}' for {task.duration} mins...")
		tot_utility += (task.priority/1000)  
		current_time += task.duration
		print(f"\t✅ t={self.time_conversion(current_time)}, task completed with {(task.priority/1000)} utils!") 
		self.remove_dependency(task.id) 
		task.status = self.COMPLETED

	    # if no task is found that matches this duration, take a break before the fixed hour task
	    else: 

		print(f"🕰t={self.time_conversion(current_time)}")
		current_time += alloted_duration
		if alloted_duration > 60: 
		    print(f"\t✨ Great! You have time to take a nap for {self.time_conversion(alloted_duration)} mins until {self.time_conversion(task_fixed.fixed_hour)}")
		else: 
		    print(f"\t Resting (Pomodoro break) before task: {task_fixed.description} for {alloted_duration} mins until {self.time_conversion(task_fixed.fixed_hour)}")
```


## Functionality

The scheduler receives information from the user to organize their day and the list of tasks they have in an optimal order for the user. The information given to the scheduler should contain the starting time of the day and at least four characteristics (attributes) of each task among the seven possible characteristics of a task described below.


    1. id: identifier of the task
    2. description: simple description of the task in text, for example, "Go to the gym"
    3. duration: how long, in minutes, a task takes to be completed
    4. dependencies: list of the tasks that have to be done before this one can be done
    5. fixed_hour: predetermined time of a task to start (constrained)
    6. type: category of a task, between academic ('A'), essential ('E'), and personal ('P' - default)
    7. latest_start_time: the latest starting time of a task 
    8. status: default = 'N' (user will not input)
    9. priority: priority value of the task (computed based on other attributes - user will not input)

As the scheduler is initiated to give the order of the tasks the user gives, each task is initialized with its given attributes. After each task is initialized, the scheduler dynamically computes the priority values of each task based on its type, duration, and number of dependencies and the given conditional utility function, explained in detail in the priority value section. As the initial priorities are computed, the scheduler takes all of them and initializes two empty heaps as the priority queues. In the priority queue, we have the characteristics of a min-heap, which will take the tasks that have fixed_hour as one of their attributes; then, on the second priority queue, we have a max-heap, which will take the 'flexible' tasks, meaning those that do not have a predetermined starting time. Finally, the scheduler runs, and it enters a loop until all the tasks are completed, which in the scheduler will be represented by not having any unscheduled tasks (those that have not been inserted into any heap) or any of the heaps not being empty. The initial current time will be the starting time of the user's day. 

For each iteration, as the loop continues until there are no tasks to be done (the termination condition is reached), the scheduler checks if any of the tasks with a status of 'not started' can now be inserted into the heap. The condition for this to happen is that the task has no dependencies; then, the scheduler will check if it has a predetermined initial time so the task is inserted in the correct heap. Then, as the tasks are ready to be executed in the iteration, it moves on to the possible cases that a task will fit, which determines the final result as if each task fits, they are output in order.


![Imgur](https://i.imgur.com/TlgVAdN.jpeg)
This image exemplifies the functionality of the Task Scheduler in steps.



## Demonstration

In this video, I showcase the functionality of the algorithm and the reasoning behind its constructions as well as its advantages and limitations.

Link to video: [https://youtu.be/5YHttbBpcvU](https://youtu.be/pX4nkPS-MK0)

## Acknowledgements

- [Min/Max-Heap Structure + Priority Queues](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/). 
- [Utility values calculation](https://www.amazon.de/-/en/Daniel-Kahneman/dp/0374533555).
- Acknowledgement of Professor R. for being an insightful resource in the construction of this project.
