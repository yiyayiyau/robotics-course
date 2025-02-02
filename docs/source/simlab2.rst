============================
 SimLab Exercise 2 - Motion
============================

Note: Before new exercises, always update the repo::

  cd $HOME/git/robotics-course
  git pull
  git submodule update
  cd build
  make -j4


a) Basic IK
===========

The first example in ``course3-Simulation/03-motion`` demonstrates a
minimalistic setup to use optimization for IK. The initial example
creates a KOMO instance setup to solve a 1-time-step optimization
problem (i.e., and IK problem). Start from this example to generate
more interesting motion. The specific tasks are:

1. Vary between the left gripper and right gripper reaching for the object. Is there a difference to "the object reaching for the right gripper" vs.\ the other way around? And test the left gripper reaching for the right gripper.
2. Also constrain the gripper orientation when reaching for the object. For a start, try to add a ``FS_quaternionDiff`` constraint. Why does this not work immediately? Try to change the object pose so that the constraint can be fulfilled. Be able to explain the result.
3. There are (in my view better) alternatives to apriori fixing the gripper orientation (the full quaternion). Instead add a ``FS_scalarProductXZ`` constraint and try to understand. (Zoom into the little coordinate frame in the gripper center to understand conventions.) Play around with all combinations of ``scalarProduct??`` and understand the effect. Further, add one more argument ``target={.1}`` to the ``addObjective`` method, and understand the result. Using multiple scalar product features, how could you also impose a full orientation constraint, and how would this differ to constraining the quaternion directly?
4. Think more holistically about grasping: How could it be realized properly? For example, think about optimizing a series of two or three poses, where the first might be a so-called *pre-grasp*, and the others model approach and final grap (from which the gripper-close command could be triggered). Create such a sequence. Note: Do all of this without yet explicitly using collision (``pairCollision``) features.
5. For your information, ``tutorials/2-features.ipynb`` gives an impression about what alternative features one can use to design motion.

  
b) Path Optimization
====================

In the first exercise you learnt created basi motion using direct
Inverse Kinematics. This exercise is about using more general
optimization methods to design motion.

To design grasps and for robot manipulation, the coordinate system of
the endeffector is really important. In our convention, the
``graspCenter`` coordinate frame is attached to the endeffector, and
is aligned similarly as if it were a camera looking at the object: the
z-axis goes back, and the gripper open/closes along the x-axis. For
instance, if you want to grasp a cylinder, you need to get the gripper
x-axis orthogonal to the zylinder's axis.

To explore possible pre-defined features, check https://github.com/MarcToussaint/robotics-course/blob/master/tutorials/2-features.ipynb .


Exercise 1:
-----------

Use the KOMO to compute configurations for various objectives.

a) Compute a 2-arm robot configuration, where the graspCenter positions of both hands coincide, the two hands oppose, and their x-axes are orthogonal. (E.g., as if they would handover a little cube.) 
b) Add a box (shape type ``ssBox``, see Tip2 below)  somewhere to the scene, compute a robot configuration where one of the grippers grasps the box (centered, along a particular axis), while avoiding collisions between the box and the two fingers and gripper.
c) Propose alternatives for how to design grasps, based on any geometric feature you can think of. (Note that our collision code can compute normals and witness points for proximity queries, which can be used.)

Tip1: If you want to use hard equality objectives, introduce them first as SOS and tune their scaling so that the result is approximately ok. Then change their type to EQ.

Tip2: ``ssBox`` means sphere-swept box. This is a box with rounded corners. This should be your default primitive shape. The shape is determined by 4 numbers: x-size, y-size, z-size, radius of corners. The 2nd most important shape type is ``ssCvx`` (sphere-swept convex), which is determined by a set of 3D points, and sphere radius that is added to the points' convex hull. (E.g., a capsule can also be described as simple ssCvx: 2 points with a sweeping radius.) The sphere-swept shape primitives allow for well-defined Jacobians of collision features.


Exercise 2:
-----------

Generate nice paths between a  start configuration :math:`q_0` and a goal configuration :math:`q_1`.


a) Let :math:`q_1` be the solution to Exercise 1a) above, and let :math:`q_0` the robot start configuration. Generate a nice path from start to goal using direct joint space interpolation. Ideally, use a sine motion profile or some other means to ensure smooth acceleration and deceleration.
b) Execute the path in simulation. (E.g., send it series of position references.)
c) Use KOMO to compute an optimal path from start to goal. We have the SOS objective of minimizing sum of square accelerations, and the EQ objective on qItself to constrain the final configuration :math:`q_1`.
d) As c), but also add the ``accumulatedCollisions`` feature as inequality throughout. (Perhaps test by adding obstacles. Note that shapes need to have set ``setContact(1)`` before creating KOMO to enable their collision.)


Exercise 3 (Bonus):
-------------------

Realize a simplest possible instance of Operational Space Control using KOMO. [The python interfaces are not ready for this yet.]

a) Setup a minimal KOMO problem of order :math:`k=2`. Add a add_qControlObjective to penalize accelerations. Add another add_qControlObjective to penalize also velocities! Add a weak objective on the hand position. Solve and make a single step forward.
b) Repeat the above, always recreating KOMO from the new current configuration. [Reuse of the KOMO instance is possible - needs nicer interface.]

