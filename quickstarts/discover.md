# Discover your Poppy robot

The Poppy robots and in particular the [pypot.robot.Robot](../pypot/doc/pypot.robot.html#pypot.robot.robot.Robot) objects from pypot make their best to allow you to easily discover and program
you robot.

## Where do I put my code?

If you are using a simulator, program directly in your computer, in a
Python console (enter `python` in your terminal) or in any editor you
like (Python IDLE...). Then launch it with the run button or use the
command line `python myprogram.py`.

Otherwise, launch the IPython notebook server (via the webapp TODO or
with this command in SSH
`ipython notebook --ip 0.0.0.0 --no-mathjax --no-browser`) then connect
with your browser to [poppy.local:8888](http://poppy.local:8888) TODO
ipython screenshot Create a new notebook and put your code into it.

Last solution, SSH into your robot (using Putty or
`ssh poppy@poppy.local`) and open an editor (`vim` by default on the
Odroid) to put your code directly in the robot. Note that if you want to
use a more advanced editor, you can use the -X option while connecting
in SSH to get XWindows back on your desktop computer.

## Create the robot object

The first (and more difficult!) step is to create this robot object.

If you have a physical Poppy Humanoid, Poppy Torso or Poppy Ergo Jr, you
can directly use the corresponding libraries and create your robot with
(let's call the robot *poppy*):

```python
from poppy_humanoid import PoppyHumanoid
poppy = PoppyHumanoid()

from poppy_torso import PoppyTorso
poppy = PoppyTorso()

from poppy_ergo_jr import PoppyErgoJr
poppy = PoppyErgoJr()
```

Otherwise, if you use a custom robot defined in a configuration file,
use (see [here](../pypot/doc/quickstart-configfile.html)):

```python
import pypot.robot
poppy = pypot.robot.from_json('my_config.json')
```

## Motors

Now that we are connected to the robot, we can list all available
motors:

```python
print(poppy.motors)
```

You get something like:

```python
[<DxlMotor name=l_elbow_y id=44 pos=-0.0>,
 <DxlMotor name=r_elbow_y id=54 pos=-0.0>,
 <DxlMotor name=r_knee_y id=24 pos=0.2>,
 <DxlMotor name=head_y id=37 pos=-22.7>,
 <DxlMotor name=head_z id=36 pos=0.0>,
 <DxlMotor name=r_arm_z id=53 pos=-0.0>,
 <DxlMotor name=r_ankle_y id=25 pos=1.6>,
 <DxlMotor name=r_shoulder_x id=52 pos=1.1>,
 <DxlMotor name=r_shoulder_y id=51 pos=0.0>,
 <DxlMotor name=r_hip_z id=22 pos=0.1>,
 <DxlMotor name=r_hip_x id=21 pos=1.2>,
 <DxlMotor name=r_hip_y id=23 pos=-0.0>,
 <DxlMotor name=l_arm_z id=43 pos=-0.0>,
 <DxlMotor name=l_hip_x id=11 pos=-0.0>,
 <DxlMotor name=l_hip_y id=13 pos=-0.1>,
 <DxlMotor name=l_hip_z id=12 pos=0.1>,
 <DxlMotor name=abs_x id=32 pos=0.0>,
 <DxlMotor name=abs_y id=31 pos=0.4>,
 <DxlMotor name=abs_z id=33 pos=0.0>,
 <DxlMotor name=l_ankle_y id=15 pos=0.8>,
 <DxlMotor name=bust_y id=34 pos=0.8>,
 <DxlMotor name=bust_x id=35 pos=0.2>,
 <DxlMotor name=l_knee_y id=14 pos=0.0>,
 <DxlMotor name=l_shoulder_x id=42 pos=0.4>,
 <DxlMotor name=l_shoulder_y id=41 pos=0.0>]
```

We can make it more readable by extracting the motor's name and
position:

```python
for m in poppy.motors:
  print('motor {} in position {}'.format(m.name, m.present_position)
```

The motors are grouped into motor groups:

```python
for group in poppy.alias:
    print ("motor group ", group)

# assuming "head" is one of the motor groups (use "top" for a ergo_jr)
for m in poppy.head:
    print('motor {} in position {}'.format(m.name, m.present_position)
```

Motors are said compliant if they do not apply torque to reach their
goal position. They are safe and can be moved by hand. To have the robot
move by itself, you must remove the compliance.

> **warning**
>
> Before removing compliance, be sure each motor is in its allowed angle
> range, otherwise, the robot may move very quickly to get back to an
> *allowed* position

You can do it for the whole robot, for a motor group or for each motor:

```python
# set the whole robot non-compliant
poppy.compliant = True
poppy.head.compliant = False

poppy.l_shoulder_x.compliant = False
```

Now that compliance is removed, let make the robot move! Put it in an
open place, far from your coffee, fingers and screen and let's go. The
`present_position` of a Dynamixel motor is its angular position (in
degree) sensed by the integrated rotationnal encoder. Its`goal_position`
is the angular position we ask it to achieve and the motor will
therefore move toward it:

```python
import time

poppy.head_z.goal_position = 20.
poppy.head_y.goal_position = 10.

time.sleep(2) # wait 2 seconds

poppy.head_y.goal_position = -10.
```

See the [motors documentation](../pypot/doc/motor.html) for more advanced control.

## Sensors

As Poppy Creatures are made of both motors and sensors, you can naturally also access those sensors.

First, you can retrieve all sensors attached to your robot via:
```python
print(poppy.sensors)
>>> [<pypot.sensor.imagefeature.marker.MarkerDetector object at 0x7fb11f05f4d0>,
     <pypot.sensor.camera.opencvcam.OpenCVCamera object at 0x7fb11f042650>]
```
You can also get their names:
```python
print([sensor.name for sensor in poppy.sensors])
>>> ['marker_detector', 'camera']
```
and then directly use this name to access the sensor:
```python
print (poppy.camera)
>>> <pypot.sensor.camera.opencvcam.OpenCVCamera object at 0x7fb11f042650>
```

The present sensors vary depending on which Poppy Creature you are using. For instance, the ErgoJr has:

* a camera
* image feature detection (marker detection in the example above)

You can then introspect each sensor to get available registers:
```python
print(poppy.camera.registers)
>>> ['frame', 'resolution', 'fps', 'index']
```

And then read the value you are interested in:
```python
print(poppy.camera.resolution)
>>> [640, 480]
```

More details on the sensors and how you can add extra sensors are given in the [extending sensors](TODO) section.

.. note:: We separate motors and sensors even if this is kind of an artificial definition here. Indeed, we used servomotors which can be seen both as motors and sensors.

## Go further

Next step is to use primitives, which is decribed in
[this tuto](../pypot/quickstart-primitive.html).