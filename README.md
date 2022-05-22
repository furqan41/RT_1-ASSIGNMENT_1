Python Robotics Simulator
================================

This is a simple, portable robot simulator developed by [Student Robotics](https://studentrobotics.org).
Some of the arenas and the exercises have been modified for the Research Track I course

The project aims to make a holonomic robot move inside a maze without hitting walls made of golden boxes. Furthermore, inside the maze, there are many silver tokens that the robot have to grab, move them behind him and then start again with the search for the next tokens.

![](https://raw.githubusercontent.com/furqan41/RT_1-ASSIGNMENT_1/main/IMAGES/simulator.PNG)

here the robot detacts the SILVER BOXEs pick them and GOLDEN BOXES to avoid them and the robot will pick the Silver boxes after one and other in the maze countinously 

Installing and running 
----------------------

The simulator requires a Python 2.7 installation, the [pygame](http://pygame.org/) library, [PyPyBox2D](https://pypi.python.org/pypi/pypybox2d/2.1-r331), and [PyYAML](https://pypi.python.org/pypi/PyYAML/).

Pygame, unfortunately, can be tricky (though [not impossible](http://askubuntu.com/q/312767)) to install in virtual environments. If you are using `pip`, you might try `pip install hg+https://bitbucket.org/pygame/pygame`, or you could use your operating system's package manager. Windows users could use [Portable Python](http://portablepython.com/). PyPyBox2D and PyYAML are more forgiving, and should install just fine using `pip` or `easy_install`.

## Troubleshooting

When running `python run.py <file>`, you may be presented with an error: `ImportError: No module named 'robot'`. This may be due to a conflict between sr.tools and sr.robot. To resolve, symlink simulator/sr/robot to the location of sr.tools.

On Ubuntu, this can be accomplished by:
* Find the location of srtools: `pip show sr.tools`
* Get the location. In my case this was `/usr/local/lib/python2.7/dist-packages`
* Create symlink: `ln -s path/to/simulator/sr/robot /usr/local/lib/python2.7/dist-packages/sr/`

## Exercise
-----------------------------

To run one or more scripts in the simulator, use `run.py`, passing it the file names. 

python2 run.py exercise3_solution.py
```

Robot API
---------

The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api].

### Motors ###

The simulated robot has two motors configured for skid steering, connected to a two-output [Motor Board](https://studentrobotics.org/docs/kit/motor_board). 
The left motor is connected to output `0` and the right motor to output `1`.

The Motor Board API is identical to [that of the SR API](https://studentrobotics.org/docs/programming/sr/motors/), except that motor boards cannot be addressed by serial number.
So, to turn on the spot at one quarter of full power, one might write the following:


```python
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25
```

**_TO DRIVE THE ROBOT FORWARD_**

def drive(speed , seconds):

	
	'''
	Function that gives the robot the ability of move straight for a certain time with a defined speed 
	
	Arguments:
	
		speed = speed of the motors, that will be equal on each motor to move straight
		
		seconds = time interval in which the robot will move straight 
		
	This function has no returns
		
	'''

	R.motors[0].m0.power = speed
	R.motors[0].m1.power = speed 
	time.sleep(seconds) 
	R.motors[0].m0.power = 0
	R.motors[0].m1.power = 0
 
 here the both moter have same speed in the same direction that makes the robot move forward 
 
 **_TO TURN THE ROBOT _**
 
 def turn(speed , seconds):

	'''
	Function that gives the robot the ability to turn on its axis 
	
	Arguments:
	
		speed = speed of the motors, which will be positive on one and negative on the other in order to create the rotation
		
		seconds = time interval in which the robot will rotate 
	
	This function has no returns 
	
	'''

	R.motors[0].m0.power = speed 
	R.motors[0].m1.power = -speed 
	time.sleep(seconds)
	R.motors[0].m0.power =0
	R.motors[0].m1.power =0
 
 here the one MOTOR will have positive speed and the other motor will have (-speed) that makes the robot to rotate 

****WHEN THE ROBOT FIND THE SILVER TOCKEN**


(WHICH MEANS THE REQUIRED THRESHOLD HAS BEEN ACHIVED THAN THE ROBOT ALLIGNS ITSELF WITH THE POSITION AND ANGEL OF THE SILVER TOCKEN)

def Routine():

	'''
	Function that activates the Routine ( Grab , turn , release , turn ) if and only if the robot is close enough to the silver token (distance threshold: 0.4)
	if the the robot is far from the silver token it will drive.
	
	This function has no return
	
	'''

	d_th = 0.4
	
	dist , rot_y = find_silver_token()
	
	if dist > d_th or dist == -1:
	
		drive(75,0.1)



 # and then robot grib the silver box put it behind and then robot return to its privious position  
	
		R.grab()
		print("Gotta catch'em all!")
		turn(20,3)
		R.release()
		drive(-20,0.9)
		turn(-20,3)
  
  **_IF THE SILVER BOX IS MISALLIGNED_ **

(WHICH THE ROBOT NEEDS TO  ALLIGNS ITSELF WITH THE POSITION AND ANGEL OF THE SILVER TOCKEN[IF IT IS NOT ALLIGNED])


def Silver_Approach(dist, rot_y):

	'''
	This Function approach the silver token when the robot detect one. At first The robot will verify if it is in the right direction of the silver token and, if it's not, it will adjust itself turning right and left. Secondly it will call the function Routine().
	
	Arguments:
	
		dist = the distance of the silver token that the robot has detected 
		
		rot_y = the angle of the silver token that the robot has detected 
	
	This function has no returns
	
	'''
	
	a_th = 2
	
	while rot_y < -a_th or rot_y > a_th :
	
		if rot_y < -a_th:
		
			turn(-2,0.5)
			
			print("left a bit!!")
			
		if rot_y > a_th:
		
			turn(2,0.5)
			
			print("right a bit!!")
			
		dist , rot_y = find_silver_token()
		
	Routine()
 
 
### The Grabber ###

The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the `R.grab` method:

```python
success = R.grab()
```

The `R.grab` function returns `True` if a token was successfully picked up, or `False` otherwise. If the robot is already holding a token, it will throw an `AlreadyHoldingSomethingException`.

To drop the token, call the `R.release` method.

Cable-tie flails are not implemented.

A function has been created to clean the main function of the code from the routine that the robot does when it has to grab a silver token.

Routine() : When the robot is close enough to a silver token (in particular at a distance of 0.4), thanks to this function, it will grab the token (method: R.grab ()), it will turn 180 ° ( function: turn ()), will release the token (method: R.release ()), go back for a while (function: drive ()) and finally turn 180 ° again to continue the ride in the maze.

This function has no Returns .

    R.grab()
    print("Gotcha!!")
    turn(20,3)
    R.release()
    drive(-20,0.9)
    turn(-20,3)
    
   **_WHEN ROBOT COME CLOSER TO THE WALL(GOLED BOXES)_**
  The rotation function will be called out and this function will make the decision weather the robot should move left of right without touch the golden boxes 
  
   When the robot is close to the wall it calculates (using the R.see() method) the distance between it and the nearest golden box, respectively, to its right and left
  
  def Rotation():   # When the robot is close to the wall it calculates (using the R.see() method) the distance between it and the nearest golden box, respectively,                         to its right and left

	'''
	Function to calculate the distance between the robot and the nearest golden box, respectively, to its right and left, each at an angle of 30 degrees (between 75 and 105 degrees for its right and between -105 and -75 degrees for its left).
	
	The robot will rotate towards the furthest golden box until it no longer sees any golden box in cone of 91 degrees at a distance of 1 in front of it. 
	
	The angle and the distance of the cone are settable by passing the arguments to the function find_golden_token(..,..).
	
	Thanks to this feature the robot will always turn counter-clockwise.
	
	
	This function has no returns 
	
	'''

	loc = {'right_distance':7, 'left_distance':7}																																																																																																																																																																																																		
	
	for token in R.see():
	
		if 75 < token.rot_y < 105:
		
			if token.info.marker_type is MARKER_TOKEN_GOLD and token.dist < loc['right_distance']:
				
				loc['right_distance'] = token.dist
				
		if -105 < token.rot_y < -75:
		
			if token.info.marker_type is MARKER_TOKEN_GOLD and token.dist < loc['left_distance']:
			
				loc['left_distance'] = token.dist 
				
	if loc ['right_distance'] > loc['left_distance']:
	
		while find_golden_token(1,45.5):
		
			turn(10,0.1)
			
		
			
	else:
	
		while find_golden_token(1,45.5):
			
			turn(-10,0.1) 

### Vision ###

To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The `R.see` method returns a list of all the markers the robot can see, as `Marker` objects. The robot can only see markers which it is facing towards.

Each `Marker` object has the following attributes:

* `info`: a `MarkerInfo` object describing the marker itself. Has the following attributes:
  * `code`: the numeric code of the marker.
  * `marker_type`: the type of object the marker is attached to (either `MARKER_TOKEN_GOLD`, `MARKER_TOKEN_SILVER` or `MARKER_ARENA`).
  * `offset`: offset of the numeric code of the marker from the lowest numbered marker of its type. For example, token number 3 has the code 43, but offset 3.
  * `size`: the size that the marker would be in the real game, for compatibility with the SR API.
* `centre`: the location of the marker in polar coordinates, as a `PolarCoord` object. Has the following attributes:
  * `length`: the distance from the centre of the robot to the object (in metres).
  * `rot_y`: rotation about the Y axis in degrees.
* `dist`: an alias for `centre.length`
* `res`: the value of the `res` parameter of `R.see`, for compatibility with the SR API.
* `rot_y`: an alias for `centre.rot_y`
* `timestamp`: the time at which the marker was seen (when `R.see` was called).

For example, the following code lists all of the markers the robot can see:

```python
markers = R.see()
print "I can see", len(markers), "markers:"

for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} metres away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} metres away".format( m.info.offset, m.dist )
```

[sr-api]: https://studentrobotics.org/docs/programming/sr/


**_FLOW CHAT OF THE SYSTEM_**

![](https://github.com/furqan41/RT_1-ASSIGNMENT_1/blob/main/IMAGES/flow%20chat.png)

**_OVERALL REVIEW ABOUT THE ASSIGNENT_ **

https://user-images.githubusercontent.com/105802251/169558461-c287938b-ec8b-4b38-a580-8b0f698716c1.mp4

