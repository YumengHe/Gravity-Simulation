# Gravity-Simulation

## Week 1 Findings (04/13 - 04/19)
Did a bit of reading about how Unity itself manages physics, as well as how the whole  pipeline can be handled by a custom engine through scripts and Unity API with the free [Okay Dynamics Engine](https://assetstore.unity.com/packages/tools/physics/okay-dynamics-engine-68022) as an example. 

**TL;DR**: Unity doesn't allow global physics manager access. We would need our own. To simulate gravity, we need: 1) a global empty object with a script component that keeps track of the gravity grid and does the update step, and 2) a script component on each celestial body that handles the data passing and local transform updates.

### Unity Physics

Unity uses the opensource [PhysX engine](https://github.com/NVIDIA-Omniverse/PhysX) as its native physics engine, capable of handling rigid bodies, joint constraints, and cloth simulation; but nothing further. To make use of it on any game object, we can simply attach a rigid body component and a collider type component to the desired game object. The former holds object properties like mass, dampening factors, constraints, gravity toggles, etc; while the latter holds collider information that captures its size and friction factors. Additionally, a constant force component can be added to the object to apply external forces and torques. 

When writing custom scripts for a game object that has the aforementioned physics component, information from these components can be accessed through these function calls:
```
gameObject.GetComponent<Collider>();

gameObject.GetComponent<Rigidbody>();
```

*Did not explore how joints work as that seems to be entirely tangential to our overall goal.*

### ODE structure

To simulate gravitational forces on astronomical scale, or any physics simulation for that matter, we would need globally keep track of all participating parties in the simulation. Unity does not open its own global physics manager to user edits, so we will have to write our own global manager. We can find a parallel of this in Sebastian Lague's [project](https://github.com/SebLague/Solar-System/blob/Episode_01/Assets/Scripts/Game/EndlessManager.cs). 

However, since I am *stupid* and forgot about this repository's existance until the writing of this journal, I mostly dug around ODE's demo scene demo_boxstack_bowl to study this. Notably, it has an empty object "Engine" to keep track of all the global variables for the physics simulation. It has a few script component that comes from the ODE package but nothing else. In the ```ODEEngine.cs``` script module, it keeps a list of collisions that is filled each frame by function calls from ```RegisterCollision()``` method under the Collider type, which in turn uses ```OnTriggerEnter()``` (defined in ```ODETrigger.cs```) to seemingly extract collision events from the Unity PhysX engine*. We will need to do the same for our gravity simulation, keeping track of the global gravity field like in kavan's project.

The colliding objects has a script component that helps handling the aformentioned information passing process from each collider to our own physics manager. A similar script component should be maintained for each celestial body in our simulation so that we can update each body's transformation in the global manager each frame. 

*This comprehension contradicts what is described in the documentation on ODE's asset store page. Further reading is desirable but tangential to this project. 

## Links

### Video Walkthroughs
Simulating Gravity in C++: [YouTube](https://www.youtube.com/watch?v=_YbGWoUaZg0&ab_channel=kavan)

Coding Adventure: Solar System: [YouTube](https://www.youtube.com/watch?v=7axImc1sxa0) | [Github](https://github.com/SebLague/Solar-System/)

### Unity Related
ODE package lite: [Unity AssetStore](https://assetstore.unity.com/packages/tools/physics/okay-dynamics-engine-68022)

Unity Physics Documentation: [UnityDocs](https://docs.unity3d.com/6000.0/Documentation/Manual/PhysicsOverview.html)

Basics to Unity Object: [UnityDocs](https://docs.unity3d.com/6000.0/Documentation/Manual/GameObjects.html)

Scripting in Unity: [UnityDocs](https://docs.unity3d.com/6000.0/Documentation/Manual/scripting-get-started.html)

