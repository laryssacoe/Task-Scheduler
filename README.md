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

The objective of this project was to develop a small Flask web application that integrates an IoT device simulated by an IoT platform (Blynk was used). 

The web app serves as a control server to the theoretical device, meaning in the 'Streaming' Page, the user has access to the eletronic device that intends to simulate a robot, in this case, a LED was used that would get the commands from both the web app server as well as the Blynk server to connect to the device by Wi-Fi and be able to turn the light ON and OFF through the computer.



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

Demonstration of code snippets and their purpose. 

* Sign Up functionality: 
Defines the route to sign up page that will request the user to create an username and password and will save the the values in a tuple in the table that will be used to match the credentials to the user in the redirecting of login page. 

```python
@app.route('/signup', methods=['GET', 'POST'])
def signup():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        conn = sqlite3.connect('users.db')
        c = conn.cursor()

        # asks for the insertion of username and password values as a tuple stored in users
        c.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, password))

        conn.commit()
        conn.close()

        # after sign up, redirects the user to do their login
        return redirect(url_for('login'))

    return render_template('signup.html')
```


* Streaming Commands: 
Defines the commands that will send information to the Blynk server to turn the light ON and OFF and creates a function that will update the status in the streaming page by getting this information from the Blynk server.

```JavaScrpit
	// defines the function that turns the light on in IoT integration
	function turnLightOn() {
		// uses XML Http and the GET request to update data from the blink server API and make value of light = 1
		var xhttp = new XMLHttpRequest();
		xhttp.open("GET", "https://blynk.cloud/external/api/update?token=9htotNtrt1lrBSfxk8gwZNA5S8iMlMkY&v0=1", true);
		// sends new value
		xhttp.send();
		// calls the update light status function to automatically change the status to 1 or 'on'
		updateLightStatus();
	}

    // defines the function that turns light off in IoT integration
	function turnLightOff() {
		var xhttp = new XMLHttpRequest();
		// uses XML Http and the GET request to update data from the blynk server API and make value of light = 0
		xhttp.open("GET", "https://blynk.cloud/external/api/update?token=9htotNtrt1lrBSfxk8gwZNA5S8iMlMkY&v0=0", true);
		// sends the new value
			xhttp.send();
		// calls the update light status function to automatically change the status to 0 or 'off'
		updateLightStatus();
	}

	// defines the function that will update constantly and display the status of the light
		function updateLightStatus() {

	// uses XML Http to get the new request of data from the Blynk API server to update light status
    var xhttp = new XMLHttpRequest();

    // defines a function that will only be called by the change of light state in the XMLHttp request and display its status
    xhttp.onreadystatechange = function() { // calls the function upon change using "xhttp.onreadystate" property of XML HTTP

        // checks if the "readystate" xmlhttp property is done (4) and the request of data was successful (200)
        if (this.readyState == 4 && this.status == 200) {

            // if so, assigns the variable response to a response text that will be used to display status
            var responseText = this.responseText;

            // checks the status of the light by the response text, if the response is '1', attatches the value "on" to the string lightStatus
            // if not, assigns the value "off" to be displayed
            var lightStatus = responseText == "1" ? "on" : "off";

            // attributes an identifier to the HTML element light-status that will return the content setup
            // in this case, the content is the status of the light in string plus the actual value define in variable "lightStatus"
            document.getElementById("light-status").innerHTML = "The status of the light is currently " + lightStatus + "!";

			// checks the status of the light to assign icon id to the specific png link
            if (lightStatus == "on") {
                document.getElementById("light-icon").src = "https://toppng.com/uploads/preview/light-bulb-on-off-png-11553940319kdxsp3rf0i.png";
            } else {
                document.getElementById("light-icon").src = "https://toppng.com/uploads/preview/light-bulb-on-off-png-11553939967conkckniqn.png";
            }
        }
    };

    // initializes a new GET request of data to the Blynk server and sends the request to get the new value of the light
    xhttp.open("GET", "https://blynk.cloud/external/api/get?token=9htotNtrt1lrBSfxk8gwZNA5S8iMlMkY&v0", true);
    xhttp.send();
}
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

- [Min/Max-Heap Structure + Priority Queues] Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). Introduction to algorithms (3rd ed.). 
- [Utility values calculation]Kahneman, D. (2013). Chapters 26 and 29, In Thinking, fast and slow. New York: Farrar, Straus, and Giroux.
 - [Blynk Documentation](https://docs.blynk.io/en/blynk.cloud/get-datastream-value)
 - [CSS Resources](https://freefrontend.com/css-code-examples/)
