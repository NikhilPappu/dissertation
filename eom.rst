.. _eom:

===========================
Bicycle Equations Of Motion
===========================

Preface
=======

Attempting to derive the equations of motion of the Whipple bicycle model was
the trigger which solidified my graduate research topic. I attempted the
derivation for my class project in Mont Hubbard's winter 2006 multi-body
dynamics class and struggled with it well into the summer before finally
getting a mostly correct answer. After the fact, I realized much of my pain was
caused by a single missing apostrophe in my Autolev source code\ [#apostrophe]_.
Luckily another student in the class, Thomas Englehardt, had also derived the
equations and helped me debug by sharing his code and going over his methods.
Even then, it turned out that my original equations weren't "exactly" correct
and it wasn't until Luke Peterson joined our lab and got the bicycle dynamics
itch did I get the bugs sorted out in my derivation with his help.
Conversations and collaboration with Luke have improved the derivation
significantly and influence much of what follows. Luke has also continued to
improve the derivation with the goal of printing the first compact symbolic
result. With careful parameterization and selection of generalized speeds, the
non-linear equations may be able to be put into human readable form.

Non-linear Equations of Motion
==============================

The Whipple Model is the foundation of all the models presented in this
dissertation. This section details derivation of the non-linear equations of
motion using Kane's method :cite:`Kane1985`\ [#kane]_. The non-linear equations of
motion are algebraically unwieldy and no one so far has publicly printed them
in a form compact enough to print on reasonably sized paper, and certainly not
in a form suitable for any in-depth analytical understanding as pointed out in
:cite:`Meijaard2007`. My methodology relies heavily on computer aided algebra to do
the bookkeeping in the derivation, so I will only describe the necessary
details to correctly derive the equations, leaving the algebra, trigonometry
and calculus to the computer. The symbolic equations of motion herein were
originally developed using Autolev :cite:`Kane2000`, a proprietary and now defunct
software package for symbolically deriving equations of motion for multi-body
systems. I've since used the open source software `SymPy <http://sympy.org>`_
to derive the equations with the help of the included ``mechanics`` package
which was developed in our lab to provide a software package suitable for
academia with capabilities similar to Autolev\ [#autolev]_. The input code for
both software packages are available in the ``src/eom`` directory of the
dissertation source files.

Model Description
-----------------

The *Whipple Bicycle Model* :cite:`Whipple1899` is defined by this set of
assumptions:

* The model is made up of four rigid bodies: the rear frame (the main
  bicycle frame which may or may not include a rigid rider), the front frame
  (typically the fork and handlebar assembly), and two wheels.
* The bodies are connected to each other by frictionless revolute joints.
* The wheels have knife edges and contact the ground under pure rolling with no
  side-slip.
* The complete bicycle is assumed to be laterally symmetric.
* The bicycle rolls on flat ground.

:cite:`Astrom2005`, :cite:`Limebeer2006`, and :cite:`Meijaard2007` all provide excellent
overviews of the model, its history, and its features.

Unfortunately the word "model" is often ambiguous. I will attempt to be as
precise as possible with my wording. For this chapter, I consider a dynamic
model, such as the *Whipple Bicycle Model*, to be equivalent to another
dynamic model if at least the minimal set of equations of motions are the same
(i.e. they give the same result when evaluated at a particular configuration
and state of interest). This implies that the Whipple Bicycle Model linearized
about the nominal configuration is a different model than the non-linear
*Whipple Bicycle Model*. I will try to be explicit when discussing the various
models.

I will use the following terminology and labels for the four rigid bodies, see
:ref:`Figure 3.2<figBicycleGeometry>`:

Rear Frame, :math:`C`
   The main bicycle frame which may include parts or all of the rider.
Rear Wheel, :math:`D`
   The rear wheel of the bicycle.
Front Frame, :math:`E`
   The fork and handlebar assembly.
Front Wheel, :math:`F`
   The front wheel of the bicycle.

.. _parameterization:

Parameterization
----------------

The benchmark derivation of the linear Whipple model :cite:`Meijaard2007` about the
nominal configuration uses a non-minimal set of parameters based on typical
geometric parameters and inertia definitions using an inertially fixed
reference frame in which the :math:`x` axis points forward and the :math:`z`
axis points down, :ref:`Figure 3.1<figTypicalBicycleGeometry>`. The *nominal
configuration* is defined as the configuration when the steering angle is zero
and the bicycle is upright with respect to gravity and the ground plane. The
parameters presented in :cite:`Meijaard2007` are not necessarily the best choice of
parameters, especially when looking at the model from a non-linear perspective,
as they are not the simplest set nor a minimal set. For example, the benchmark
parameters can be reduced in number by making use of gyrostats, see
:cite:`Sharp2008a` for an implementation. Choosing a minimum, constant set of
parameters can certainly reduce the complexity of the resulting non-linear
equations. In this derivation, I use a parameterization with different geometry
and inertial definitions to facilitate a more intuitive non-linear derivation,
but do not make use of gyrostats to reduce the number of parameters.

.. _geometry:

Geometry
--------

The geometry of the Whipple model can be parameterized in an infinite number of
ways. It is typical and often natural to define the geometry with respect to
the descriptions of bicycle geometry used in the bicycle fabrication industry,
such as wheel diameter, head tube angle, trail and or rake, :ref:`Figure
3.1<figTypicalBicycleGeometry>`. Choices of parameterizations like these create
unnecessary complications when developing the non-linear equations of motion
because they are typically defined with respect to only the nominal
configuration of the bicycle and are not constant with respect to the system
configuration.

.. _figTypicalBicycleGeometry:

.. figure:: figures/eom/typical-bicycle-geometry.*
   :align: center
   :width: 2.36in
   :target: _images/typical-bicycle-geometry.png

   The typical parameterization of the fundamental bicycle's geometry given in
   :cite:`Meijaard2007`. The wheelbase :math:`w`, trail :math:`c`, steer axis tilt
   :math:`\lambda`, front wheel radius :math:`r_F`, and rear wheel radius
   :math:`r_R` are shown.

With that in mind and after trying various parameterizations, I use geometric
formulation presented by :cite:`Psiaki1979`\ [#psiaki]_. The wheels are described by
their radius :math:`\left(r_{F,R}\geq0\right)` and the remaining geometry is
defined by three distances, all of which are configuration invariant. The
distance :math:`d_1` is the offset to the center of the rear wheel from the
steer axis and :math:`d_3` is the offset of the front wheel from the steering
axis. :math:`d_2` is then the distance between the wheel centers as measured
along the steer axis. :ref:`Figure 3.2<figBicycleGeometry>` gives a complete
visual description of the bodies and the geometry.

.. _figBicycleGeometry:

.. figure:: figures/eom/bicycle-geometry.*
   :align: center
   :width: 3.75in
   :target: _images/bicycle-geometry.png

   The bicycle in the nominal configuration. The rigid bodies are the rear
   frame :math:`C`, rear wheel :math:`D`, front frame :math:`E`, and front
   wheel :math:`F`. The geometric parameters and important points are also
   shown.

Generalized Coordinates
-----------------------

The bicycle is completely configured in a Newtonian reference frame by nine
generalized coordinates: six coordinates locate and orient the rear frame in
space and three additional coordinates are needed for the three revolute joints
connecting the front frame and wheels to the rear frame. I made use of the SAE
vehicle dynamics reference frame standard and all rotations are are defined as
positive right-handed which makes the configuration identical to that in
:cite:`Meijaard2007`\ [#standard]_. The rotation matrices are defined as

.. math::
   :label: eqRotationDefinition

   ^a \bar{v} = {}^N\mathbf{R}^A\ {}^n \bar{v}

where :math:`^n\bar{v}` is a vector expressed in the :math:`N` frame and
:math:`^a\bar{v}` is the same vector expressed in the :math:`A` frame.

.. _figBicycleCoordinates:

.. figure:: figures/eom/bicycle-coordinates.*
   :width: 3.6in
   :align: center
   :target: _images/bicycle-coordinates.png

   The bicycle in a general configuration showing each of the eight generalized
   coordinates.

To configure the bicycle, begin by locating the point that follows the rear
wheel contact in the ground plane of the Newtonian reference frame, :math:`N`,
with longitudinal and lateral coordinates :math:`q_1` and :math:`q_2`,
respectively, see :ref:`Figure 3.3<figBicycleCoordinates>`. Then orient the
rear frame, :math:`C`, with respect to the Newtonian reference frame through a
body-fixed 3-1-2 rotation defining the yaw angle, :math:`q_3`, the roll angle,
:math:`q_4`, and the pitch angle, :math:`q_5`. The intermediate frames yaw,
:math:`A`, and roll, :math:`B`, are implicitly generated. The rotation matrix of
:math:`C` relative to :math:`N` is then

.. math::
   :label: eqNtoC

   ^N\mathbf{R}^C =
   \begin{bmatrix}
     c_3c_5 - s_3s_4s_5 & s_4s_5c_3 + s_3c_5 & -s_5c_4\\
     -s_3c_4 & c_3c_4  & s_4\\
     s_5c_3 + s_3s_4c_5 & s_3s_5 - s_4c_3c_5 & c_4c_5
   \end{bmatrix}

where the notation :math:`s_i = \operatorname{sin}(q_i)` and :math:`c_i =
\operatorname{cos}(q_i)` in :eq:`eqNtoC` and also all subsequent equations in
this chapter.

The rear wheel, :math:`D`, rotates with respect to the rear frame about the
:math:`\hat{c}_2` axis through :math:`q_6`.

.. math::
   :label: eqCtoD

   ^C\mathbf{R}^D =
   \begin{bmatrix}
     c_6 & 0 & -s_6\\
     0 & 1 & 0\\
     s_6 & 0 & c_6
   \end{bmatrix}

The front frame, :math:`E`, rotates with respect to the rear frame about the
:math:`\hat{c}_3` axis through the steering angle, :math:`q_7`.

.. math::
   :label: eqCtoE

   ^C\mathbf{R}^E =
   \begin{bmatrix}
     c_7 & s_7 & 0\\
     -s_7 & c_7 & 0\\
     0 & 0 & 1
   \end{bmatrix}

Finally, the front wheel, :math:`F`, rotates with respect to the front frame
through :math:`q_8` about the :math:`\hat{e}_2` axis.

.. math::
   :label: eqEtoF

   ^E\mathbf{R}^F =
   \begin{bmatrix}
     c_8 & 0 & -s_8\\
     0 & 1 & 0\\
     s_8 & 0 & c_8
   \end{bmatrix}

The first two coordinates locate the rear wheel contact point in the Newtonian
reference frame and the remaining six coordinates orient the four rigid bodies
within the Newtonian reference frame.

The positions of the various points on the bicycle must be defined with respect
to the Newtonian reference frame. There are six primary points of interest: the
four mass centers, :math:`d_o,c_o,e_o,f_o`, and the two points fixed on the
wheels which are instantaneously in contact with the ground, :math:`d_n,f_n`\
[#wheelcontact]_.

The mass center of the rear wheel, :math:`d_o`, is assumed to be at the center
of the wheel and is located by

.. math::
   :label: eqRearWheelMassCenter

   \bar{r}^{d_o/n_o} = q_1\hat{n}_1 + q_2\hat{n}_2 - r_R\hat{b}_3

The rear frame mass center, :math:`c_o`, is located by two additional
parameters

.. math::
   :label: eqFrameMassCenter

   \bar{r}^{c_o/d_o} = l_1\hat{c}_1 + l_2\hat{c}_3

For convenience, define an additional point on the steer axis, :math:`c_e`,
such that

.. math::
   :label: eqDoToCe

   \bar{r}^{c_e/d_o} = d_1\hat{c}_1

The mass center of the front wheel, :math:`f_o`, is then located by:

.. math::
   :label: eqFrontWheelMassCenter

   \bar{r}^{f_o/c_e} =  d_2\hat{c}_3 + d_3\hat{e}_1

The front frame mass center, :math:`e_o`, is located by two more additional
parameters:

.. math::
   :label: eqForkMassCenter

   \bar{r}^{e_o/f_o} = l_3\hat{e}_1 + l_4\hat{e}_3

The location of the point on the rear wheel instantaneously in contact with the
ground in the Newtonian frame is then defined by

.. math::
   :label: eqRearWheelContact

   \bar{r}^{d_n/d_o} = r_F\hat{b}_3

The location of the front wheel contact point is less trivial. The vector from
the front wheel center to the contact point is defined as

.. math::
   :label: eqFrontWheelContact

   \bar{r}^{f_n/f_o} = r_F\left(\frac{(\hat{e}_2\times\hat{a}_3)\times\hat{e}_2}
   {||(\hat{e}_2\times\hat{a}_3)\times\hat{e}_2||}\right)

   \bar{r}^{f_n/f_o} =
   r_F(s_4s_7-s_5c_4c_7)/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2}\hat{e}_1 +
   r_Fc_4c_5/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2}\hat{e}_3

Here the triple cross product divided by its magnitude represents the unit
vector pointing from the front wheel center to the point on the front wheel
instantaneously in contact with the ground. :cite:`Basu-Mandal2007` give a good
explanation and diagram. Another description of this vector is in terms of dot
products, i.e. subtract the :math:`\hat{n}_3` component of :math:`\hat{e}_2`
from :math:`\hat{n}_3` to get a vector that points from the front wheel center
to the contact point.

.. math::
   :label: eqFrontWheelContactDot

   \bar{r}^{f_n/f_o} =
   r_F\left(\frac{\hat{a}_3 - (\hat{e}_2 \cdot\hat{a}_3)\hat{e}_2}
   {||\hat{a}_3 - (\hat{e}_2 \cdot\hat{a}_3)\hat{e}_2||}\right)

This is easily shown to be equivalent to :eq:`eqFrontWheelContact` by writing
the triple cross product as sum of dot products.

Holonomic Constraints
---------------------

Two holonomic configuration constraints, arising from the fact that both wheels
must touch the ground, complicate the model derivation. The first holonomic
equation is obviated by definition of the rear wheel center
:eq:`eqRearWheelMassCenter`. This enforces that the rear wheel contact point
cannot have an displacement in the :math:`\hat{n}_3` direction\ [#holonomic]_.
The second holonomic constraint is enforced by requiring the front wheel to
touch the ground plane. The constraint is characterized by a non-linear
relationship between the roll angle :math:`q_4`, steer angle :math:`q_7` and
pitch angle :math:`q_5`.

.. math::
   :label: eqHolonomicConstraint

   \bar{r}^{f_n/d_n}\cdot\hat{a}_3 =
   &d_2c_4c_5 - d_1s_5c_4 + r_Rc_4 +
   r_Fc_4^2c_5^2/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2} -\\
   &(s_4s_7-s_5c_4c_7)(d_3+r_F(s_4s_7-s_5c_4c_7)/(c_4^2c_5^2+
   (s_4s_7-s_5c_4c_7)^2)^{1/2}) = 0

In this equation, pitch, :math:`q_6`, is defined as the dependent coordinate.
The choice of pitch has to do with the fact that for "normal" bicycle
configurations, pitch is practically constant. This is not universal and it may
be smart to choose the dependent coordinate differently for other cases. The
constraint equation can actually be formulated into a quartic in the sine of
the pitch (:cite:`Psiaki1979`, :cite:`Peterson2008`) which does have an analytic, albeit
lengthy, solution. I do not opt for the analytical solution, so care is needed
when simulating and linearizing to properly choose the proper roots associated
with this dependent coordinate.

Kinematical Differential Equations
----------------------------------

The choice of generalized speeds can significantly reduce the length of the
equations of motion :cite:`Mitiguy1996`. This is beneficial for both working with
the analytical forms of the equations of motion and the efficiency in
computation. Even though this is true, little effort was spent in selecting
optimal generalized speeds, as the analytical form of the equations of motion
of this system would be difficult to interpret regardless of the choice and
because computational speed was of little concern. For :math:`i=1,\dotsc,8`
simply choose the generalized speeds to be equal to the time derivatives of the
generalized coordinates

.. math::
   :label: eqGenerlizedSpeeds

   u_i = \dot{q}_i

Velocity
--------

The angular and linear velocities of each rigid body are required for computing
the partial velocities and accelerations used in Kane's method. Also, the
velocities of the points on the wheels at the ground contact points are needed
for the development of the nonholomic constraints. The angular velocity of the
rear frame, :math:`C`, in :math:`N` is

.. math::
   :label: eqOmegaCinN

   ^N\bar{\omega}^C =
   (c_5u_4-s_5c_4u_3)\hat{c}_1 +
   (u_5+s_4u_3)\hat{c}_2 +
   (s_5u_4+c_4c_5u_3)\hat{c}_3

Both the front frame and the rear wheel are connected to the bicycle rear frame
by simple revolute joints, so the angular velocities are simply

.. math::
   :label: eqOmegaDinC

   ^C\bar{\omega}^D = u_6 \hat{c}_2

and

.. math::
   :label: eqOmegaEinC

   ^C\bar{\omega}^E = u_7 \hat{c}_3

The front wheel has angular velocity relative to the front frame

.. math::
   :label: eqOmegaFinE

   ^E\bar{\omega}^F = u_8 \hat{e}_2

The angular velocity of any of the bodies can now be computed with respect to
the Newtonian reference frame. For example,

.. math::
   :label: eqOmegaFinN

   ^N\bar{\omega}^F = {}^N\bar{\omega}^C + {}^C\bar{\omega}^E + {}^E\bar{\omega}^F

which expands to

.. math::
   :label: eqOmegaFinNExpanded

   ^N\bar{\omega}^F =
   &(s_7c_8u_5-s_8u_7-(s_5s_8-c_5c_7c_8)u_4-
   (s_8c_4c_5-c_8(s_4s_7-s_5c_4c_7))u_3)\hat{f}_1 + \\
   &(u_8+c_7u_5+(s_4c_7+s_5s_7c_4)u_3-s_7c_5u_4)\hat{f}_2 + \\
   &(c_8u_7+s_7s_8u_5+(s_5c_8+s_8c_5c_7)u_4+
   (c_4c_5c_8+s_8(s_4s_7-s_5c_4c_7))u_3)\hat{f}_3

Using the angular velocities and the position vectors, the velocities of the
mass centers can be computed. Starting with mass center of the rear wheel

.. math::
   :label: eqDoInN

   ^N\bar{v}^{d_o} = \frac{d}{dt}\left(\bar{r}^{d_o/n_o}\right)
   = u_1\hat{n}_1 + u_2\hat{n}_2 - r_Rs_4u_3\hat{b}_1 + r_Ru_4\hat{b}_2

The remaining velocities can be computed by taking advantage of the fact that
various pairs of points are fixed on the same rigid body. The mass centers of
the rear wheel, :math:`d_o` and the rear frame, :math:`c_o`, and the steer axis
point, :math:`c_e`, (:ref:`Figure 3.2<figBicycleGeometry>`) all lie on the rear
frame.

.. math::
   :label: eqCoInN

   ^N\bar{v}^{c_o} = {}^N\bar{v}^{d_o} + {}^N\bar{\omega}^C\times\bar{r}^{c_o/d_o}

where

.. math::

   ^N\bar{\omega}^C\times\bar{r}^{c_o/d_o} =
   l_2(u_5+s_4u_3)\hat{c}_1 +
   (l_1(s_5u_4+c_4c_5u_3)-l_2(c_5u_4-s_5c_4u_3))\hat{c}_2 -
   l_1(u_5+s_4u_3)\hat{c}_3

and

.. math::
   :label: eqCeInN

   ^N\bar{v}^{c_e} = {}^N\bar{v}^{d_o} + {}^N\bar{\omega}^C\times\bar{r}^{c_e/d_o}

where

.. math::

   ^N\bar{\omega}^C\times\bar{r}^{c_e/d_o} = d_1(s_5u_4+c_4c_5u_3)\hat{c}_2 -
   d_1(u_5+s_4u_3)\hat{c}_3

The velocity of the front wheel mass center is computed with respect to the
steer axis point as they both lie on the front frame

.. math::
   :label: eqFoInN

   ^N\bar{v}^{f_o} = {}^N\bar{v}^{c_e} + {}^N\bar{\omega}^E\times\bar{r}^{f_o/c_e}

where

.. math::

   ^N\bar{\omega}^E\times\bar{r}^{f_o/c_e} =
   &-d_2(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)\hat{e}_1 + \\
   &(d_3(u_7+s_5u_4+c_4c_5u_3)-d_2(s_7u_5+c_5c_7u_4+
   (s_4s_7-s_5c_4c_7)u_3))\hat{e}_2 + \\
   &d_3(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)\hat{e}_3

Then the velocity of the front frame mass center is similarly

.. math::
   :label: eqEoInN

   ^N\bar{v}^{e_o} = {}^N\bar{v}^{f_o} + {}^N\bar{\omega}^E\times\bar{r}^{e_o/f_o}

where

.. math::

   ^N\bar{\omega}^E\times\bar{r}^{e_o/f_o} =
   &-l_4(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)\hat{e}_1 +\\
   &(l_3(u_7+s_5u_4+c_4c_5u_3)-l_4(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3))\hat{e}_2 +\\
   &l_3(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)\hat{e}_3

The velocity of the contact points on the wheel are needed to enforce the
no-slip condition and can be computed with respect to the rear and front wheel
centers. The rear contact point is

.. math::
   :label: eqDnInN

   ^N\bar{v}^{d_n} = {}^N\bar{v}^{d_o} + {}^N\bar{\omega}^D\times\bar{r}^{d_n/d_o}

where

.. math::

   ^N\bar{\omega}^D\times\bar{r}^{d_n/d_o} = r_R(u_5+u_6+s_4u_3)\hat{b}_1 - r_Ru_4\hat{b}_2

which simplifies to

.. math::
   :label: eqSimpleDnInN

   ^N\bar{v}^{d_n} = r_R(u_5+u_6)\hat{b}_1 + u_1\hat{n}_1 + u_2\hat{n}_2

The front wheel contact velocity is

.. math::
   :label: eqFnInN

   ^N\bar{v}^{f_n} = {}^N\bar{v}^{f_o} + {}^N\bar{\omega}^F\times\bar{r}^{f_n/f_o}

where

.. math::

   ^N\bar{\omega}^F\times\bar{r}^{f_n/f_o} =
   &r_F/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2}\\
   &[ -c_4c_5(s_7c_5u_4-u_8-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)\hat{e}_1 - \\
   &(c_4c_7u_4+s_7c_4c_5u_5-s_4s_5s_7u_4-(s_4s_7-s_5c_4c_7)u_7)\hat{e}_2 + \\
   & (s_4s_7-s_5c_4c_7)(s_7c_5u_4-u_8-c_7u_5-(s_4c_7+
   s_5s_7c_4)u_3)\hat{e}_3 ]

Acceleration
------------

The angular acceleration of each body along with the linear acceleration of
each mass center are required to form :math:`F_r^*` in Kane's equations. The
angular acceleration of the rear frame, :math:`C`, in :math:`N` is

.. math::
   :label: alphaCinN

   ^N\bar{\alpha}^C =
   &(s_4s_5u_3u_4+c_5\dot{u}_4-s_5u_4u_5-c_4c_5u_3u_5-s_5c_4\dot{u}_3)\hat{c}_1 + \\
   &(c_4u_3u_4+\dot{u}_5+s_4\dot{u}_3)\hat{c}_2 +\\
   &(c_5u_4u_5+s_5\dot{u}_4+c_4c_5\dot{u}_3-s_4c_5u_3u_4-s_5c_4u_3u_5)\hat{c}_3

The remaining bodies' angular accelerations follow from simple rotations

.. math::
   :label: alphaDinC

   ^C\bar{\alpha}^D = \dot{u}_6\hat{c}_2

.. math::
   :label: alphaEinC

   ^C\bar{\alpha}^E = \dot{u}_7\hat{c}_3

.. math::
   :label: alphaFinE

   ^E\bar{\alpha}^F = \dot{u}_8\hat{e}_2

The linear acceleration of each mass center can then be computed. The
acceleration of the rear wheel center of mass is

.. math::
   :label: aDoInN

   ^N\bar{a}^{d_o} = \frac{d}{dt}\left(^N\bar{v}^{d_o}\right)

   ^N\bar{a}^{d_o} = \dot{u}_1\hat{n}_1 + \dot{u}_2\hat{n}_2 -
   r_Rs_4u_3^2\hat{a}_2 - r_R(2c_4u_3u_4+s_4\dot{u}_3)\hat{b}_1 +
   r_R\dot{u}_4\hat{b}_2 + r_Ru_4^2\hat{b}_3

The remaining accelerations are computed using the analogous two point
relationship utilized for the velocities. The acceleration of the rear frame
center of mass is

.. math::
   :label: aCoinN

   ^N\bar{a}^{c_o} = {}^N\bar{a}^{d_o} +
   {}^N\bar{\omega}^C\times(^N\bar{\omega}^C\times\bar{r}^{c_o/d_o}) +
   {}^N\bar{\alpha}^C\times\bar{r}^{c_o/d_o}

where

.. math::

   ^N\bar{\omega}^C\times(^N\bar{\omega}^C\times\bar{r}^{c_o/d_o}) =
   &(-l_1(u_5+s_4u_3)^2-(s_5u_4+c_4c_5u_3)(l_1(s_5u_4+c_4c_5u_3)-\\
   &l_2(c_5u_4-s_5c_4u_3)))\hat{c}_1 +\\
   &(u_5+s_4u_3)(l_2(s_5u_4+c_4c_5u_3)+l_1(c_5u_4-s_5c_4u_3))\hat{c}_2 + \\
   &((c_5u_4-s_5c_4u_3)(l_1(s_5u_4+c_4c_5u_3)-l_2(c_5u_4-s_5c_4u_3))-\\
   &l_2(u_5+s_4u_3)^2)\hat{c}_3

and

.. math::

   ^N\bar{\alpha}^C\times\bar{r}^{c_o/d_o} =
   &l_2(c_4u_3u_4+\dot{u}_5+s_4\dot{u}_3)\hat{c}_1 + \\
   &(-l_1(s_4c_5u_3u_4+s_5c_4u_3u_5-c_5u_4u_5-s_5\dot{u}_4-c_4c_5\dot{u}_3)-\\
   &l_2(s_4s_5u_3u_4+c_5\dot{u}_4-s_5u_4u_5-c_4c_5u_3u_5-s_5c_4\dot{u}_3))\hat{c}_2 - \\
   &l_1(c_4u_3u_4+\dot{u}_5+s_4\dot{u}_3)\hat{c}_3

The acceleration of the steer axis point is

.. math::
   :label: aCeInN

   ^N\bar{a}^{c_e} = {}^N\bar{a}^{d_o} +
   {}^N\bar{\omega}^C\times(^N\bar{\omega}^C\times\bar{r}^{c_e/d_o}) +
   {}^N\bar{\alpha}^C\times\bar{r}^{c_e/d_o}

where

.. math::

   ^N\bar{\omega}^C\times(^N\bar{\omega}^C\times\bar{r}^{c_e/d_o}) =
   &-d_1((u_5+s_4u_3)^2+(s_5u_4+c_4c_5u_3)^2)\hat{c}_1 + \\
   &d_1(u_5+s_4u_3)(c_5u_4-s_5c_4u_3)\hat{c}_2 +\\
   &d_1(s_5u_4+c_4c_5u_3)(c_5u_4-s_5c_4u_3)\hat{c}_3

and

.. math::

   ^N\bar{\alpha}^C\times\bar{r}^{c_e/d_o} =
   &-d_1(s_4c_5u_3u_4+s_5c_4u_3u_5-c_5u_4u_5-
   s_5\dot{u}_4-c_4c_5\dot{u}_3)\hat{c}_2 - \\
   &d_1(c_4u_3u_4+\dot{u}_5+s_4\dot{u}_3)\hat{c}_3

The acceleration of the front wheel center of mass is

.. math::
   :label: aFoInN

   ^N\bar{a}^{f_o} = {}^N\bar{a}^{c_e} +
   {}^N\bar{\omega}^E\times(^N\bar{\omega}^E\times\bar{r}^{f_o/c_e}) +
   {}^N\bar{\alpha}^E\times\bar{r}^{f_o/c_e}

where

.. math::

   ^N\bar{\omega}^E\times(^N\bar{\omega}^E\times\bar{r}^{f_o/c_e}) =
   &(-d_3(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)^2-\\
   &(u_7+s_5u_4+c_4c_5u_3)(d_3(u_7+s_5u_4+c_4c_5u_3)-\\
   &d_2(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3)))\hat{e}_1 - \\
   &(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)(d_2(u_7+s_5u_4+c_4c_5u_3)+\\
   &d_3(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3))\hat{e}_2 + \\
   &((s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3)(d_3(u_7+s_5u_4+c_4c_5u_3)-\\
   &d_2(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3))-\\
   &d_2(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)^2)\hat{e}_3

and

.. math::

   ^N\bar{\alpha}^E\times\bar{r}^{f_o/c_e} =
   & -d_2(s_7u_5u_7+c_5c_7u_4u_7+u_3(s_4s_7u_7+s_4s_5s_7u_4-c_4c_7u_4-
   s_5c_4c_7u_7-s_7c_4c_5u_5)+\\
   & s_7c_5\dot{u}_4-s_5s_7u_4u_5-
   c_7\dot{u}_5-(s_4c_7+s_5s_7c_4)\dot{u}_3)\hat{e}_1 + \\
   & (d_2(s_5c_7u_4u_5+s_7c_5u_4u_7-c_7u_5u_7-
   u_3(s_4c_7u_7+s_7c_4u_4+s_4s_5c_7u_4+s_5s_7c_4u_7-\\
   & c_4c_5c_7u_5)-s_7\dot{u}_5-
   c_5c_7\dot{u}_4-(s_4s_7-s_5c_4c_7)\dot{u}_3)-\\
   & d_3(s_4c_5u_3u_4+s_5c_4u_3u_5-c_5u_4u_5-\dot{u}_7-
   s_5\dot{u}_4-c_4c_5\dot{u}_3))\hat{e}_2 + \\
   & d_3(s_7u_5u_7+c_5c_7u_4u_7+u_3(s_4s_7u_7+s_4s_5s_7u_4-c_4c_7u_4-
   s_5c_4c_7u_7-s_7c_4c_5u_5)+\\
   & s_7c_5\dot{u}_4-
   s_5s_7u_4u_5-c_7\dot{u}_5-(s_4c_7+s_5s_7c_4)\dot{u}_3)\hat{e}_3

The acceleration of the front frame center of mass is

.. math::
   :label: aEoInN

   ^N\bar{a}^{e_o} = {}^N\bar{a}^{f_o} +
   {}^N\bar{\omega}^E\times(^N\bar{\omega}^E\times\bar{r}^{e_o/f_o}) +
   {}^N\bar{\alpha}^E\times\bar{r}^{e_o/f_o}

where

.. math::

   ^N\bar{\omega}^E\times(^N\bar{\omega}^E\times\bar{r}^{e_o/f_o}) =
   &(-(d_3+l_3)(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)^2-\\
   &(u_7+s_5u_4+c_4c_5u_3)((d_3+l_3)(u_7+s_5u_4+c_4c_5u_3)-\\
   &d_2(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3)-\\
   &l_4(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3)))\hat{e}_1 - \\
   &(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)((d_2+l_4)(u_7+s_5u_4+
   c_4c_5u_3)+\\
   &(d_3+l_3)(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3))\hat{e}_2+ \\
   &((s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3)((d_3+l_3)(u_7+s_5u_4+c_4c_5u_3)-\\
   &d_2(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3)-\\
   &l_4(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3))-\\
   &(d_2+l_4)(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)^2)\hat{e}_3

and

.. math::

   ^N\bar{\alpha}^E\times\bar{r}^{e_o/f_o} =
   & -(d_2+l_4)(s_7u_5u_7+c_5c_7u_4u_7+u_3(s_4s_7u_7+s_4s_5s_7u_4-c_4c_7u_4-
   s_5c_4c_7u_7-\\
   &s_7c_4c_5u_5)+s_7c_5\dot{u}_4-s_5s_7u_4u_5-c_7\dot{u}_5-
   (s_4c_7+s_5s_7c_4)\dot{u}_3)\hat{e}_1+ \\
   & (d_2(s_5c_7u_4u_5+s_7c_5u_4u_7-c_7u_5u_7-u_3(s_4c_7u_7+s_7c_4u_4+
   s_4s_5c_7u_4+s_5s_7c_4u_7-\\
   &c_4c_5c_7u_5)-s_7\dot{u}_5-
   c_5c_7\dot{u}_4-
   (s_4s_7-s_5c_4c_7)\dot{u}_3)+l_4(s_5c_7u_4u_5+s_7c_5u_4u_7-c_7u_5u_7-\\
   & u_3(s_4c_7u_7+s_7c_4u_4+s_4s_5c_7u_4+s_5s_7c_4u_7-c_4c_5c_7u_5)-
   s_7\dot{u}_5-\\
   & c_5c_7\dot{u}_4-(s_4s_7-s_5c_4c_7)\dot{u}_3)-
   (d_3+l_3)(s_4c_5u_3u_4+s_5c_4u_3u_5-c_5u_4u_5-\\
   &\dot{u}_7-s_5\dot{u}_4-c_4c_5\dot{u}_3))\hat{e}_2 + \\
   & (d_3+l_3)(s_7u_5u_7+c_5c_7u_4u_7+u_3(s_4s_7u_7+s_4s_5s_7u_4-c_4c_7u_4-
   s_5c_4c_7u_7-\\
   &s_7c_4c_5u_5)+s_7c_5\dot{u}_4-
   s_5s_7u_4u_5-c_7\dot{u}_5-
   (s_4c_7+s_5s_7c_4)\dot{u}_3)\hat{e}_3

.. _nonholonomic:

Motion Constraints
------------------

Motion constraints reduce the dimensions of the locally achievable
configuration space from nine to three. The first four constraints are
introduced to enforce the pure rolling, no side-slip, contact of the knife-edge
wheels with the ground plane and are nonholonomic. This sets the components of
velocity of the contact points on the wheels in the :math:`\hat{a}_1` and
:math:`\hat{a}_2` directions equal to zero, producing the following
relationships

.. math::
   :label: eqNonholonomic1

   ^N\bar{v}^{d_n}\cdot\hat{a}_1 = s_3u_2 + c_3u_1 + r_R(u_5+u_6) = 0

.. math::
   :label: eqNonholonomic2

   ^N\bar{v}^{d_n}\cdot\hat{a}_2 = c_3u_2 - s_3u_1 = 0

.. math::
   :label: eqNonholonomic3

   ^N\bar{v}^{f_n}\cdot\hat{a}_1 =
   & s_3u_2 + c_3u_1 + d_2c_5u_5 + d_2s_4c_5u_3 - r_Rs_4u_3 -
   d_3s_7c_4u_3 - d_1s_5(u_5+s_4u_3)\\
   & r_F c_4c_7(u_8+c_7u_5+
   (s_4c_7+s_5s_7c_4)u_3)/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2} -\\
   & s_7c_5(d_3u_7-
   r_F (s_7c_4c_5u_5-(s_4s_7-s_5c_4c_7)u_7)/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2}) - \\
   &s_5(d_3c_7(u_5+s_4u_3)+r_Fs_4s_7(u_8+c_7u_5+(s_4c_7+
   s_5s_7c_4)u_3)/(c_4^2c_5^2+\\
   &(s_4s_7-s_5c_4c_7)^2)^{1/2}) = 0

.. math::
   :label: eqNonholonomic4

   ^N\bar{v}^{f_n}\cdot\hat{a}_2 =
   &c_3u_2 + d_1c_5u_3 + r_Rc_4u_4 + d_1s_4c_5u_5 + d_1s_5c_4u_4 +\\
   &(c_4c_7-s_4s_5s_7)(d_3(u_7+s_5u_4+c_4c_5u_3)-\\
   &d_2(s_7u_5+c_5c_7u_4+(s_4s_7-s_5c_4c_7)u_3)-r_F(c_4c_7u_4+
   s_7c_4c_5u_5-s_4s_5s_7u_4-\\
   &(s_4s_7-s_5c_4c_7)u_7)/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2}) - \\
   &s_3u_1 - (s_7c_4+s_4s_5c_7)(d_2(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)+\\
   &r_Fc_4c_5(s_7c_5u_4-u_8-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2}) - \\
   &s_4c_5(d_3(s_7c_5u_4-c_7u_5-(s_4c_7+s_5s_7c_4)u_3)+\\
   &r_F(s_4s_7-s_5c_4c_7)(s_7c_5u_4-u_8-c_7u_5-\\
   &(s_4c_7+s_5s_7c_4)u_3)/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1/2}) = 0

The fifth motion constraint is used to enforce the constraint on the velocities
imposed by the holonomic constraint, Equation :eq:`eqHolonomicConstraint`. This
is strictly introduced to ease numerical integration and linearization. By
differentiating the holonomic constraint equation we arrive at an equation that
is linear in the generalized speeds and can be treated as any other motion
constraint

.. math::
   :label: eqHolonomicDerivative

   \frac{d}{dt}(\bar{r}^{G_n/d_n}\cdot\hat{a}_3) =
   &r_Rs_4u_4 + d_1s_4s_5u_4 + (d_3+r_F(s_4s_7-s_5c_4c_7)/(c_4^2c_5^2+
   (s_4s_7-s_5c_4c_7)^2)^{1/2})\\
   &(s_4c_7u_7+s_7c_4u_4+s_4s_5c_7u_4+s_5s_7c_4u_7-c_4c_5c_7u_5) -
   d_1c_4c_5u_5 - \\
   &d_2s_4c_5u_4 -d_2s_5c_4u_5 -
   r_Fc_4c_5(s_4c_4^2c_5^3u_4+s_5c_4^3c_5^2u_5+\\
   &(s_4s_7-s_5c_4c_7)^2(s_4c_5u_4+s_5c_4u_5))/(c_4^2c_5^2+(s_4s_7-s_5c_4c_7)^2)^{1.5} = 0

These five equations are linear in the eight generalized speeds. Following
convention, I choose the roll rate, :math:`u_4`, the rear wheel rate,
:math:`u_6`, and steer rate, :math:`u_7`, as independent generalized speeds.

Now we calculate the dependent speeds as functions of the independent speeds by
solving the linear system of equations and differentiating the resulting
equations to find the dependent :math:`\dot{u}`'s. The dependent speeds take
the form

.. math::
   :label: eqDependentSpeeds

   u_1 = f(u_4, u_6, u_7, q_3, \ldots, q_8)

   u_2 = f(u_4, u_6, u_7, q_3, \ldots, q_8)

   u_3 = f(u_4, u_6, u_7, q_4, \ldots, q_8)

   u_5 = f(u_4, u_7, q_4, \ldots, q_8)

   u_8 = f(u_4, u_6, u_7, q_4, \ldots, q_8)

At this point, the equations become analytically long and it is not necessarily
trivial to reduce the length of the equations. A smarter choice of generalized
speeds could certainly help, but no effort was spent searching for a good set.
From this point on in the derivation, the analytical results of the equations
of motion will not be shown. I will only outline the remainder of the theory,
as all of the kinematical building blocks are in place to derive the equations
with Kane's method (or any other method). I highly recommend the use of
computer aided algebra to continue on, but the die-hard could certainly write
them by hand.

Generalized Active Forces
-------------------------

The three expressions for the nonholomonic generalized active forces,
:math:`\tilde{F}_r` can now be formed. For the four body system with three
degrees of freedom :math:`(r=4,6,7)` they take the form

.. math::
   :label: eqTotalGeneralizedActiveForces

   \tilde{F}_r = (\tilde{F}_r)_C + (\tilde{F}_r)_D + (\tilde{F}_r)_E +
   (\tilde{F}_r)_F

   (\bar{F}_r)_X= {}^N\bar{V}^{x_o}_r\cdot\bar{R}^{x_o} +
   {}^N\bar{\omega}^X_r\cdot\bar{T}^X

where :math:`^N\bar{V}_r^{x_o}` is the r\ :sup:`th` partial velocity of the
mass center corresponding to the generalized speed :math:`u_r`,
:math:`\bar{R}^{x_o}` is the resultant force on the mass center (excluding
non-contributing forces), :math:`^N\bar{\omega}_r^X` is the r\ :sup:`th`
partial angular velocity of the body corresponding to the generalized speed
:math:`u_r`, and :math:`\bar{T}^X` is the resultant torques on the body where
:math:`X` is one of the four bodies. The partial velocities are simply the
partial derivative of the velocities in question with respect to the
generalized speeds and can be found systematically :cite:`Kane1985`. We assume that
the only contributing force acting on the system is the gravitational force,
:math:`g`. The resultant forces are

.. math::
   :label: eqContributingForces

   \bar{R}^{c_o} = m_Cg\hat{n}_3

   \bar{R}^{d_o} = m_Dg\hat{n}_3

   \bar{R}^{e_o} = m_Eg\hat{n}_3

   \bar{R}^{f_o} = m_Fg\hat{n}_3

We also assume that three generalized active torques act on the system which
correspond to the three independent generalized speeds found in Section
:ref:`nonholonomic`.

The roll torque, :math:`T_4`, acts between the rear frame and the Newtonian
frame about :math:`\hat{a}_1`. The rear wheel torque, :math:`T_6`, acts between
the rear frame and the rear wheel about :math:`\hat{c}_2` and the steer torque,
:math:`T_7`, acts between the rear frame and the front frame about
:math:`\hat{c}_3`.

.. math::
   :label: eqContributingTorques

   \bar{T}^C = T_4\hat{a}_1-T_6\hat{c}_2-T_7\hat{c}_3

   \bar{T}^D = T_6\hat{c}_2

   \bar{T}^E = T_7\hat{c}_3

   \bar{T}^F = 0

Generalized Inertia Forces
--------------------------

The nonholonomic generalized inertia forces, :math:`\tilde{F}^*_r`, are formed
using the accelerations and the inertial properties of the bodies.

.. math::
   :label: eqTotalGeneralizedInertiaForces

   \tilde{F}^*_r = (\tilde{F}^*_r)_C + (\tilde{F}^*_r)_D + (\tilde{F}^*_r)_E +
   (\tilde{F}^*_r)_F

   (\bar{F}^*_r)_X= {}^N\bar{V}^{X_o}_r\cdot\bar{R}^*_{X_o} +
   {}^N\bar{\omega}^X_r\cdot\bar{T}^*_X

where :math:`^N\bar{V}_r^{x_o}` is once again the r\ :sup:`th` partial velocity
of the mass center with corresponding to the generalized speed :math:`u_r`,
:math:`\bar{R}^*_{x_o}` is the inertia force for body :math:`X` in frame
:math:`N` and is defined as

.. math::
   :label: eqRStar

   \bar{R}^*_{x_o} = -m_X\ ^N\bar{a}^{x_o}

where the mass of each rigid body is a constant (:math:`m_C`, :math:`m_D`,
:math:`m_E` and :math:`m_F`) which is multiplied by the acceleration of the
body's center of mass. :math:`^N\bar{\omega}_r^X` is the partial angular
velocity of the body corresponding to :math:`u_r`, and :math:`\bar{T}^*_X` is
the inertia torque on the body and is defined as

.. math::
   :label: eqTStar

   \bar{T}^*_X =
   -(^N\bar{\alpha}^X\cdot I_X+{}^N\bar{\omega}^X\times I_X\cdot\bar{\omega}^X)

where :math:`I_X` is the central inertia dyadic for the body in question which
corresponds to the following tensor definitions for the inertia of each rigid
body. The inertia tensor for each body is defined with respect to the mass
center and the body's local reference frame. The bicycle wheels are assumed to
be symmetric about both their 1-3 and 1-2 planes.

.. math::
   :label: ID

   \mathbf{I}_D =
   \begin{bmatrix}
   I_{D11} & 0 & 0\\
   0 & I_{D22} & 0\\
   0 & 0 & I_{D11}
   \end{bmatrix}

.. math::
   :label: IF

   \mathbf{I}_F =
   \begin{bmatrix}
   I_{F11} & 0 & 0\\
   0 & I_{F22} & 0\\
   0 & 0 & I_{F11}
   \end{bmatrix}

The rear frame and front frame are assumed to be symmetric about their 1-3 planes.

.. math::
   :label: IC

   \mathbf{I}_C =
   \begin{bmatrix}
   I_{C11} & 0 & I_{C13}\\
   0 & I_{C22} & 0\\
   I_{C13} & 0 & I_{C33}
   \end{bmatrix}

.. math::
   :label: IE

   \mathbf{I}_E =
   \begin{bmatrix}
   I_{E11} & 0 & I_{E13}\\
   0 & I_{E22} & 0\\
   I_{E13} & 0 & I_{E33}
   \end{bmatrix}

Dynamical Equations of Motion
-----------------------------

Kane's equations are

.. math::

   \tilde{F}_r + \tilde{F}^*_r = 0 \qquad r=4,6,7

and are a vector function of three coupled equations which are linear in the
roll, steer, and rear wheel accelerations. The linear system can be solved to
give the first order equations for :math:`\dot{u}_r`, where :math:`r=4,6,7`.
These can then be  paired with the essential kinematical differential equations
to form the complete set of dynamic equations of motion in the form

.. math::

   \dot{u}_i=f_i(u_4, u_6, u_7, q_4, q_5, q_7)

   \dot{q}_j=u_j

where :math:`i=4,6,7` and :math:`j=4,5,6,7`. Keep in mind that the pitch angle,
:math:`q_5`, is in fact a dependent coordinate, :math:`q_5=f(q_4,q_7)`,
selected when dealing with the holonomic constraint,
:eq:`eqHolonomicConstraint`. Special attention during simulation and
linearization will have to be paid to accommodate the coordinate and will be
described in the following sections.

Model discussion
----------------

:cite:`Astrom2005`, :cite:`Meijaard2007`, :cite:`Basu-Mandal2007`,
:cite:`Limebeer2006`, and others do excellent jobs describing the essential
nature of both the non-linear Whipple model and various linearized models.
Notable concepts include the fact that many of the coordinates are *ignorable*,
that is they do not show up in the essential dynamical equations of motion.
These are typically the location of the ground contact point, :math:`q_1` and
:math:`q_2`, the yaw angle, :math:`q_3`, and the wheel angles, :math:`q_6` and
:math:`q_8`. The model is also energy conserving, because the contact points do
no work. The model has many equilibrium points and when linearized about the
nominal configuration at constant forward speed the open loop model (i.e. with
no inputs) can exhibit stability within various speed regimes. The system
exhibits non-minimum phase behavior and this is clearly identified in the
linear models by the right half plane zeros in various transfer functions. The
previously mentioned references are recommended for a more detailed description
of the model.

Simulation
----------

The non-linear model can be simulated with various initial conditions. In the
presented formulation all initial conditions can be set independently except
for the roll, steer, and pitch angles. Once two of the three are chosen, the
third is determined. Typically one chooses a steer and roll angle and then
solves the holonomic constraint equation numerically for the pitch angle,
:math:`q_5`, to provide the correct initial condition. :ref:`Figure
3.4<figFigFour>` is an example open loop simulation of the non-linear model.

.. _figFigFour:

.. figure:: figures/eom/meijaard2007-figure-four.*
   :width: 3in
   :align: center
   :target: _images/meijaard2007-figure-four.png

   A reproduction of Figure 4 from :cite:`Meijaard2007`. For these initial
   conditions the model demonstrates stability. It also shows the energy
   conserving nature of the non-linear model (i.e. the forward speed settles to
   a higher value than the initial speed as the energy associated with lateral
   motion is transferred to the forward speed). Generated by
   `src/eom/nonlinear_simulation.py`.

Non-linear Validation
---------------------

It is often difficult to validate that two independently derived multibody
dynamical systems are equivalent. This due to mostly to the complexity of large
analytical expressions which can be derived by different methodologies. A
common method of validation is to evaluate the symbolic equations numerically
for non-trivial inputs and compare the results to high precision. These type of
numerical benchmarks are invaluable. They provide a baseline for model
comparison allowing for scientific reproducibility and error checking.

:cite:`Basu-Mandal2007` present the non-linear Whipple model derived with both the
Newton-Euler and Lagrange methods, along with a numerical benchmark. Table 1 in
their paper gives the derivatives of all the coordinates and speeds to high
precision for both of their derivations at a single state. They claim that if
one can compute the values to machine precision to ~10 significant figures then
the models can be concluded to be the same\ [#validity]_.

The Basu-Mandal et al. derivations are based on different coordinates than
those used here, but regardless of the coordinates they provide numerical
values which can be benchmarked if the correct coordinate transformations are
applied\ [#eqconv]_. In :ref:`Table 3.1<tabNonLinValidation>`\ [#nonlincompare]_
I present the values computed from the present model in comparison to the
values presented in :cite:`Basu-Mandal2007`. I've presented the same number of
significant digits as provided by Basu-Mandall for each variable\ [#sigfig]_.
My model produces the same result to at least eleven significant figures, thus
verifying the derivation is correct.

.. _tabNonLinValidation:

.. tabularcolumns:: lll
.. include:: tables/eom/nonlin.rst

Linearized Equations of Motion
==============================

The non-linear equations of motion can be linearized about various equilibrium
points. The obvious one is about the nominal configuration :cite:`Meijaard2007` but
there are are an infinity of other associated with steady turns
:cite:`Basu-Mandal2007`. The equations can be linearized by computing the Taylor
series expansion of the non-linear equations of motion about the equilibrium
point of interest and disregarding the terms higher than first order. For the
nominal configuration, this amounts to calculating the Jacobian of the system
of equations with respect to the coordinates, speeds, and inputs to obtain the
state matrix, :math:`\mathbf{A}`, and the input matrix, :math:`\mathbf{B}`.
Output, :math:`\mathbf{C}`, and feed-forward, :math:`\mathbf{D}`,  matrices are
computed in the same fashion from the non-linear equations of the desired
outputs. The partial derivatives of each equation were evaluated at the
following fixed point: :math:`q_i=0` where :math:`i=4,5,7`, :math:`u_i=0` where
:math:`i=4,7`, and :math:`u_5=-v/r_R` where :math:`v` is the magnitude of the
component of velocity of the rear wheel center in the direction of travel. Care
has to be taken when linearizing as :math:`q_5` is a dependent coordinate which
still appears on the right hand side of the equations. In general, the chain
rule applies, but for the nominal configuration, the terms due to the implicit
definition of pitch equal zero. The linearization transforms the system into
four linear first order differential equations in the form

.. math::
   :label: eqStateSpace

   \begin{bmatrix}
     \dot{q_4}\\
     \dot{q_7}\\
     \dot{u_4}\\
     \dot{u_7}
   \end{bmatrix}
   = \mathbf{A}
   \begin{bmatrix}
     q_4\\
     q_7\\
     u_4\\
     u_7
   \end{bmatrix}
   + \mathbf{B}
   \begin{bmatrix}
     T_4\\
     T_7
   \end{bmatrix}

I calculate the equations symbolically to reach the same results presented in
:cite:`Meijaard2007`, but my equations are much lengthier because the simplification
routines available in Autolev and SymPy where not that effective.
:cite:`Meijaard2007` assumes linearity in their derivation from the start of the
derivation and avoids many of the simplification issues. The accuracy of the
linearized model was checked by comparing the numerical results to the
benchmark bicycle in two ways. First the linearized equations of motion,
Equation :eq:`eqStateSpace`, were rearranged into two second order differential
equations in the canonical form (Equation :eq:`eqCanonical`) presented in
:cite:`Meijaard2007`. They present the values for the coefficient matrices
(:math:`\mathbf{M}`, :math:`\mathbf{C}_1`, :math:`\mathbf{K}_0` and
:math:`\mathbf{K}_2`) for the benchmark parameter set to at least 13
significant figures and the linearization presented here matched all of the
significant figures. :cite:`Meijaard2007` also provide the eigenvalues of the state
matrix at various speeds. :ref:`Table 3.2<tabLinCompare>`\ [#lincompare]_ shows
the eigenvalues computed at 5 m/s compared to the values in Table 2 of
:cite:`Meijaard2007`. These match to at least 13 significant figures.

.. math::
   :label: eqCanonical

   \mathbf{M}\ddot{\bar{q}} + v\mathbf{C}_1\dot{\bar{q}} +
   \left[ g \mathbf{K}_0 + v^2 \mathbf{K}_2 \right] \bar{q} = 0

   \bar{q} = [\phi \quad \delta]^T = [q_4 \quad q_7]^T

.. _tabLinCompare:

.. raw:: latex

   \begin{footnotesize}

.. tabularcolumns:: lllll

.. include:: tables/eom/linear-compare.rst

.. raw:: latex

   \end{footnotesize}

The lateral dynamics of this linear model are remarkably similar to the
non-linear model especially around the regime of a typical bicycle's operating
point. :ref:`Figure 3.5<figLinSim>` gives an example simulation with the same
initial conditions as the  :ref:`Figure 3.4<figFigFour>`. Notice that the
lateral dynamics, steer and roll, are almost identical, but that the constant
forward speed equilibrium condition destroys the conservative property
demonstrated in the non-linear model.

.. _figLinSim:

.. figure:: figures/eom/meijaard2007-figure-four-linear.*
   :width: 3in
   :align: center
   :target: _images/meijaard2007-figure-four-linear.png

   Simulation of the linear model given the same initial conditions as
   :ref:`Figure 3.4<figFigFour>`. Generated by ``src/eom/linear_comparison.py``.

It turns out that speed has profound effect on the lateral dynamics of the
bicycle. It is useful to plot the root locus, :ref:`Figure 3.6<figRootLocus>`,
of the characteristic equation with respect to the change in the equilibrium
forward speed to visualize the time constants, damping, frequency, and
stability of each of the modes of motion. The modes can be clearly identified
along with the speed dependent stability.

.. _figRootLocus:

.. figure:: figures/eom/root-locus-complex.*
   :width: 5in
   :align: center
   :target: _images/root-locus-complex.png

   The root locus with respect to forward speed. The color variation signifies
   speed as shown in the right side color bar. Two eigenvalues are real, one
   being stable but increasingly fast and the other slowing to a marginally
   unstable location at high speed. The other two are initially real and
   unstable, but these coalesce into a complex pair and eventually become
   stable, at a higher moderately well damped frequency. Generated by
   `src/eom/linear_comparison.py`.

Another useful and popular way to visualize the root locus is by plotting the
real and imaginary eigenvalue components separately versus forward speed,
:ref:`Figure 3.7<figEigenvalues>`. This view gives a clearer view of the stable
speed range.

.. _figEigenvalues:

.. figure:: figures/eom/root-locus.*
   :width: 5in
   :align: center
   :target: _images/root-locus.png

   The real (solid) and imaginary (dashed) eigenvalue components versus speed
   for an example parameter set. The four modes of motion are identified.
   *Caster* is stable and real for all positive values of speed. It describes
   the tendency for the front wheel to right itself in forward motion.
   *Capsize* is always real and stable at low speeds but becomes marginally
   unstable at a higher speed. It describes the roll of the rear frame. *Weave*
   is real at very low speeds and describes an inverted pendulum-like motion
   i.e. the bicycle falls over. As speed increases the eigenvalues coalesce
   into a complex conjugate pair at the weave bifurcation that describes an
   exponentially increasing sinusoidal motion of roll and steer, with steer
   lagging the roll. This mode becomes stable at a higher speed. The weave and
   capsize critical speeds bound the stable speed range. Generated by
   `src/eom/linear_comparison.py`.

The behavior of the modes of motion can be visualized by plotting the
eigenvector components in the complex plane for various speeds. For the speeds
shown in the root locus plots, there are four distinct speed ranges of
interest: below the weave bifurcation, between the weave bifurcation and the
weave critical speed, the stable speed range, and above the capsize critical
speed. At a forward speed of 0.5 m/s, all eigenvalues are real, with two
unstable and two stable. In :ref:`Figure 3.8<figEVecHalf>`, notice that the
unstable eigenvalues predict two modes where the roll and steer states
exponentially increase and with roll and steer 180 degrees out of phase. These
modes describes a simple unstable inverted pendulum motion.

.. _figEVecHalf:

.. figure:: figures/eom/eVecHalf.*
   :width: 6in
   :align: center
   :target: _images/eVecHalf.png

   Normalized eigenvector components plotted in the real/imaginary plane for
   each mode at 5.0 m/s. Only the roll angle, :math:`q_4`, and steer angle,
   :math:`q_7`, components are shown. Generated by
   ``src/eom/plot_eigenvectors.py``.

For speeds above the weave bifurcation speed, this linear model exhibits three
distinct modes of motion, which have been named *weave*, *capsize*, and
*caster*. As the bicycle is brought up to a speed of 3 m/s the two unstable
eigenvalues coalesce into a complex pair. In :ref:`Figure 3.9<figEVecHalf>`, the
left-most graph depicts the caster mode which simply shows a rapidly decaying
steer angle. The unstable weave mode eigenvector components show that the steer
amplitude is about 25% larger than the roll amplitude and roll angle leads the
steer angle by about 50 degrees. Finally, the capsize mode is a decays in roll
and steer, with the roll at about twice the amplitude as steer.

.. _figEVecThree:

.. figure:: figures/eom/eVecThree.*
   :width: 6in
   :align: center
   :target: _images/eVecThree.png

   Normalized eigenvector components plotted in the real/imaginary plane for
   each mode at 3.0 m/s. Only the roll angle, :math:`q_4`, and steer angle,
   :math:`q_7`, components are shown. Generated by
   ``src/eom/plot_eigenvectors.py``

At 5 m/s all modes are stable with the weave mode showing that the roll now
only leads the steer by about 10 degrees. The caster and capsize are similar to
3 m/s.

.. _figEVecFive:

.. figure:: figures/eom/eVecFive.*
   :width: 6in
   :align: center
   :target: _images/eVecFive.png

   Normalized eigenvector components plotted in the real/imaginary plane for
   each mode at 5.0 m/s. Only the roll angle, :math:`q_4`, and steer angle,
   :math:`q_7`, components are shown. Generated by
   ``src/eom/plot_eigenvectors.py``

Finally, at 7 m/s the capsize mode becomes unstable, with a slow exponential
increase primarily in roll.

.. _figEVecSeven:

.. figure:: figures/eom/eVecSeven.*
   :width: 6in
   :align: center
   :target: _images/eVecSeven.png

   Normalized eigenvector components plotted in the real/imaginary plane for
   each mode at 7.0 m/s. Only the roll angle, :math:`q_4`, and steer angle,
   :math:`q_7`, components are shown. Generated by
   ``src/eom/plot_eigenvectors.py``

The eigenvalues and eigenvectors describe the complete motion of the linear
bicycle model where the motion from each mode is sum to gather the whole motion
at a given speed. Typical diamond frame bicycle designs with an average sized
rigid rider exhibit this basic behavior, but even small variations in some of
the parameters can change the system behavior drastically, especially with
regards to stability.

Parameter Conversion
====================

This section details the conversion from the benchmark parameter set in
:cite:`Meijaard2007` to the presented parameter set, :ref:`Figure
3.2<figBicycleGeometry>`. As mentioned earlier, the Meijaard parameters are not a
good choice when working with the non-linear derivation because most are not
constant with respect to configuration. So the parameters must be converted to
a configuration independent set. When the bicycle is in the nominal
configuration the parameters can be converted with the following relationships.

Starting with the geometry, the wheel radii are defined the same, but the
remaining geometry is calculated as

.. math::
   :label: eqGeometryConversion

   d_1 = \operatorname{cos}(\lambda) (c + w - r_R \operatorname{tan}(\lambda))

   d_3 = -\operatorname{cos}(\lambda) (c - r_F \operatorname{tan}(\lambda))

   d_2 = \frac{(r_R + d_1 \operatorname{sin}(\lambda) - r_F + d_3
   \operatorname{sin}(\lambda))}{\operatorname{cos}(\lambda)}

The mass center locations are

.. math::
   :label: eqMassCenterConversion

   l_1 = x_B  \operatorname{cos}(\lambda) - z_B  \operatorname{sin}(\lambda) -
   r_R  \operatorname{sin}(\lambda)

   l_2 = x_B  \operatorname{sin}(\lambda) + z_B  \operatorname{cos}(\lambda) +
   r_R \operatorname{cos}(\lambda)

   l_4 = (z_H + r_F)  \operatorname{cos}(\lambda) + (x_H - w)
   \operatorname{sin}(\lambda)

   l_3 = \frac{x_H - w - l_4  \operatorname{sin}(\lambda)}
   {\operatorname{cos}(\lambda)}

The masses are equivalently defined. Here the left side gives the present
variables and the right gives the benchmark variables.

.. math::
   :label: eqMassConversion

   m_C = m_B

   m_D = m_R

   m_E = m_H

   m_F = m_F

The moments of inertia of the wheels are also equivalently defined.

.. math::
   :label: eqWheelInertiaConversion

     \mathbf{I}_D =
     \begin{bmatrix}
       I_{D11} & 0 & 0\\
       0 & I_{D22} & 0\\
       0 & 0 & I_{D11}
     \end{bmatrix}
     = \mathbf{I}_R =
     \begin{bmatrix}
       I_{Rxx} & 0 & 0\\
       0 & I_{Ryy} & 0\\
       0 & 0 & I_{Rxx}
     \end{bmatrix}

     \mathbf{I}_F =
     \begin{bmatrix}
       I_{F11} & 0 & 0\\
       0 & I_{F22} & 0\\
       0 & 0 & I_{F11}
     \end{bmatrix}
     = \mathbf{I}_F =
     \begin{bmatrix}
       I_{Fxx} & 0 & 0\\
       0 & I_{Fyy} & 0\\
       0 & 0 & I_{Fxx}
     \end{bmatrix}

The moments and products of inertia for the frame and fork require the
direction cosine matrix with respect to rotation through the steer axis tilt,
:math:`\lambda`.

.. math::
   :label: eqConversionRotation

   \mathbf{R} =
   \begin{bmatrix}
     c_\lambda & 0 & -s_\lambda\\
     0 & 1 & 0\\
     s_\lambda & 0 & c_\lambda
   \end{bmatrix}

.. math::
   :label: eqFrameInertiaConversion

    \mathbf{I}_B =
    \begin{bmatrix}
      I_{Bxx} & 0 & I_{Bxz}\\
      0 & I_{Byy} & 0\\
      I_{Bxz} & 0 & I_{Bzz}
    \end{bmatrix}

    \mathbf{I}_C =  \mathbf{R} \mathbf{I}_B \mathbf{R}^T

.. math::
   :label: eqForkInertiaConversion

   \mathbf{I}_H =
   \begin{bmatrix}
     I_{Hxx} & 0 & I_{Hxz}\\
     0 & I_{Hyy} & 0\\
     I_{Hxz} & 0 & I_{Hzz}
   \end{bmatrix}

   \mathbf{I}_E =  \mathbf{R} \mathbf{I}_H \mathbf{R}^T

Notation
========

:math:`A,B,C,D,E,F`
   Reference frames and/or rigid bodies.
:math:`d_1,d_2,d_3`
   Distances that describe the rear and front frame essential geometry.
:math:`l_1,l_2,l_3,l_4`
   In plane distances to the rear and front frame centers of mass.
:math:`m_C,m_D,m_E,m_F`
   Mass of each rigid body.
:math:`\mathbf{I}_C,\mathbf{I}_D,\mathbf{I}_E,\mathbf{I}_F`
   Local inertia tensor of each rigid body.
:math:`^Y\mathbf{R}^X`
   Rotation matrix of body or reference frame :math:`X` with respect to body
   or reference frame :math:`Y`.
:math:`q_i`
   Generalized coordinates.
:math:`u_i`
   Generalized speeds.
:math:`\hat{x}_1,\hat{x}_2,\hat{x}_3`
   Three orthogonal unit vectors in reference frame :math:`X`.
:math:`x_o`
   Mass center of body :math:`X`.
:math:`c_e`
   Point on the steer axis in both bodies :math:`C` and :math:`E`.
:math:`d_n,f_n`
   Points fixed in the rear and front wheels which are instantaneously in
   contact with the ground.
:math:`\bar{r}^{y_o/x_o}`
   Vector from point :math:`x_o` to point :math:`y_o`.
:math:`^X\bar{\omega}^Y`
   Angular velocity of frame :math:`Y` in frame :math:`X`.
:math:`^Y\bar{v}^{x_o}`
   Velocity of point :math:`x_o` in reference frame :math:`Y`.
:math:`^X\bar{\alpha}^Y`
   Angular acceleration of frame :math:`Y` in frame :math:`X`.
:math:`^Y\bar{a}^{x_o}`
   Acceleration of point :math:`x_o` in reference frame :math:`Y`.
:math:`s_1,c_1`
   Shorthand notation for :math:`\operatorname{sin}q_1` and
   :math:`\operatorname{cos}q_1`.
:math:`s_\lambda,c_\lambda`
   Shorthand notation for :math:`\operatorname{sin}\lambda` and
   :math:`\operatorname{cos}\lambda`.
:math:`\tilde{F}_r`
   Nonholonomic generalized active forces. The subscript :math:`r` denotes
   one of the independent generalized speeds.
:math:`^N\bar{V}_r^{x_o}`
   r\ :sup:`th` partial velocity of point :math:`x_o` in reference frame
   :math:`N` with respect to the independent generalized speed, :math:`u_r`.
:math:`\bar{R}^{x_o}`
   Resultant forces on the point :math:`x_o`.
:math:`^N\bar{\omega}_r^C`
   r\ :sup:`th` partial angular velocity of reference frame :math:`C` in
   reference frame :math:`N` with respect to the independent generalized speed,
   :math:`u_r`.
:math:`\bar{T}^C`
   Resultant torques acting on body :math:`C`.
:math:`\tilde{F}^*_r`
   Nonholonomic generalized inertia forces. The subscript :math:`r` denotes
   one of the independent generalized speeds.
:math:`\bar{R}^*_{x_o}`
   Generalized inertia force of point :math:`x_o`.
:math:`\bar{T}^*_X`
   Generalized inertia torque of body :math:`X`.
:math:`I_X`
   Central inertia dyadic of body :math:`X`.
:math:`\mathbf{A},\mathbf{B},\mathbf{C},\mathbf{D}`
   The state, input, output and feed-forward matrices.

The following benchmark parameters are all defined in :cite:`Meijaard2007`.

:math:`\lambda`
   Steer axis tilt.
:math:`c`
   Trail.
:math:`w`
   Wheelbase.
:math:`r_F,r_R`
   Front and rear wheel radii.
:math:`x_B,z_B,x_H,z_H`
   Mass center locations of the rear frame and front frame.
:math:`B,R,H,F`
   Rigid bodies as defined in.
:math:`m_B,m_R,m_H,m_F`
   Mass of each rigid body.
:math:`\mathbf{I}_B,\mathbf{I}_R,\mathbf{I}_H,\mathbf{I}_F`
   Inertia tensor of each rigid body.
:math:`v`
   Forward speed of the linear bicycle model.
:math:`\mathbf{M},\mathbf{C}_1,\mathbf{K}_0,\mathbf{K}_2`
   Velocity and gravity independent mass, damping, and stiffness matrices of
   the linearized Whipple model.
:math:`\bar{q}`
   Essential coordinates.

.. rubric:: Footnotes

.. [#apostrophe] These programming growing pangs were especially harsh with
   Autolev. The program is unfortunately missing virtually all of the useful
   tools and paradigms found in full-featured programming languages.

.. [#kane] Kane himself has done work on bicycle and motorcycle models and made
   use of his method to derive the equations of motion :cite:`Kane1975`,
   :cite:`Kane1977`, :cite:`Kane1977a`, :cite:`Kane1978`, :cite:`Man1979`. Furthermore,
   :cite:`Zytveld1975` derived linear equations of motion of the bicycle and models
   developed with AutoSim also employ Kane's method :cite:`Sayers1990`,
   :cite:`Sharp1999`.

.. [#autolev] Luke and I have dreamed of developing an open source version of
   Autolev for years and that has finally culminated through primarily Luke and
   Gilbert Gede's efforts in the creation of `sympy.physics.mechanics
   <http://www.sympy.org>`_.

.. [#psiaki] The coordinates are also used by :cite:`Franke1990`, among others.

.. [#standard] I don't necessarily agree that this is a great standard to follow,
   because it creates much unessary confusion when defining and mapping between
   parameterizations. The three axis pointing upward would be less error prone
   because a bicycle is never below the ground plane, except maybe in an
   academic.

.. [#wheelcontact] The bicycle wheels' points of contact are abstract points in
   dynamics. At any instant there are two coincident points at
   the contact location: one on the wheel and one on the ground. If no-slip
   rolling is assumed, both of these points are motionless, i.e. their
   velocities are zero. These points are distinctly different from the
   points that trace out the path on the ground of the wheel contact
   locations through time. Those points are not motionless and therefore do have a
   velocity.

.. [#holonomic] This constraint can readily be modified to accomodate a
   non-flat ground.

.. [#validity] The very first model I developed in 2006 would not have held up
   to this test. I owe the validity of my model to my lab mate, Luke, as his
   persistence and interest in minute detail helped me bring my model up to
   par. Several issues plagued both of our derivations. The first was an simple
   missing apostrophe in my Autolev code, the second was that we had defined
   our rear wheel rotation with respec to the :math:`B` frame instead of the
   :math:`C` frame, the third was incorrect parameter conversions from the
   :cite:`Meijaard2007` paper to our parameters, and the fourth was a bug in
   my original code that I never found, with an entire code rewrite finally
   giving me matching eigenvalues.

.. [#eqconv] The conversion equations from the :cite:`Basu-Mandal2007` coordinates to
   the present set and vice versa can be found in the source code for the
   DynamicistToolKit bicycle module.

.. [#sigfig] Basu-Mandall presents varying significant digits from 10 to 14.

.. [#nonlincompare] The variable names correspond to the convention provided in
   :cite:`Basu-Mandal2007`. This table is generated with ``src/eom/basu_comparison.py``.

.. [#lincompare] This table is generated with ``src/eom/linear_comparison.py``.
