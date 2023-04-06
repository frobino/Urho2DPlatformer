# How to build the engine (legacy)

Note: tested with U3D 8e9c11f7febfd36b382f4c391b3d3747e800a63e

```
sudo apt-get install libx11-dev libxrandr-dev libasound2-dev git cmake make libgl1-mesa-dev
git clone https://github.com/u3d-community/U3D
cd U3D
```

Alternative 1:

```
mkdir build & cd build & cmake ../ & make
```

Alternative 2 (similar to project build procedure):

```
// Build using classic cpp compiler
cd script & ./cmake_generic.sh ../build -DCMAKE_BUILD_TYPE=Debug
cd ../build & bear -- make
```

```
// Build to Wasm using emcc compiler
cd script & ./cmake_emscripten.sh ../build/ -DEMSCRIPTEN_ROOT_PATH=/home/${USER}/Tools/emsdk/upstream/emscripten
cd ../build & bear -- make
// To see the result in the browser you need to run http-server or similar, do not open directly the html file!
```

Before building a game linked to Urho3D do always add the following env var:

```
export URHO3D_HOME=/home/${USER}/Projects/U3D/build
```

# How to create project (legacy)


```
mkdir Urho3DSampleProject & cd Urho3DSampleProject
cp -r /home/$USER/Projects/U3D/bin .
cp -r /home/$USER/Projects/U3D/CMake .
cp -r /home/$USER/Projects/U3D/script .
```
then:

- create CMakeLists.txt (using the [template](https://urho3d.github.io/documentation/1.6/_using_library.html)).
- create src/ folder and put inside main.cpp

# How to build the project (legacy)

```
cd <pathToProject>
cd script & ./cmake_generic.sh ../build
cd ../build & bear -- make
```

To create a debuggable binary, use the following in step 7:

```
cd <pathToProject>
cd script & ./cmake_generic.sh ../build -DCMAKE_BUILD_TYPE=Debug
cd ../build & bear -- make
```

script/cmake_generic.sh "build/dockerized-web" -D WEB=1 -D URHO3D_HOME=/home/urho3d/.urho3d/install/web -D CMAKE_INSTALL_PREFIX=/home/urho3d/.urho3d/install/webemcc (Emscripten gcc/clang-like replacement) 2.0.8 (d059fd603d0b45b584f634dc2365bc9e9a6ec1dd)

Note that the above steps can be automated using rake. See [documentation](https://urho3d.github.io/documentation/1.6/_using_library.html).

# How to build the engine using DBE

DBE is a dockerized environment with all tools needed to build Urho3D
and apps for all platforms based on Urho3D.

```
git clone https://github.com/u3d-community/U3D
cd U3D
# Install U3D library to a default install location inside the container
script/dockerized.sh linux rake build install
```

or

```
URHO3D_HOME=/home/urho3d/.urho3d/install/linux URHO3D_TRACY_PROFILING=1 script/dockerized.sh linux
```

or

```
script/dockerized.sh web rake build install
```

# How to create (and build) project using DBE

Export the path to the Urho3d home (that was installed inside the DBE):

```
export URHO3D_HOME=/home/urho3d/.urho3d/install/linux
```

Create the app under the Urho3D/demo folder (or clone the app in the
Urho3D/demo folder, in this case Urho2DPlatformer) and build:

```
cd U3D
# Create a new application called Urho2DPlatformer under the U3D/demo folder
./script/dockerized.sh linux rake new[Urho2DPlatformer,demo]
cd demo/Urho2DPlatformer
# Build the newly generated UrhoApp
./script/dockerized.sh linux
```

If the application is outside of the U3D project, to build it just add the URHO3D_HOME:

```
cd <pathToYourProject>
# Build the application. NOTE: no need to create "build" folder manually.
URHO3D_HOME=/home/urho3d/.urho3d/install/linux script/dockerized.sh linux
```

or build with some special options:

```
# Build the newly generated UrhoApp
URHO3D_HOME=/home/urho3d/.urho3d/install/linux URHO3D_TRACY_PROFILING=1 script/dockerized.sh linux
```

NOTE: the command above (necessary to build an application with tracy) is currently not working.
To get around the issue, I was able to use the following "trick":

```
cd /home/user/Projects/Urho3D
export URHO3D_HOME="/home/$USER/Projects/Urho3D/build/dockerized-linux/"
# Enter in the dockerized env from the engine folder
script/dockerized.sh linux bash
cd demo/Urho2DPlatformer/script
./cmake_generic.sh ../build -DURHO3D_TRACY_PROFILING=1
cd ../build && make
```

Similarly for the web build:
```
cd /home/user/Projects/Urho2DPlatformer
# enter in the dockerized env (note, we need to build U3D in dockerized env first)
URHO3D_HOME=/home/urho3d/.urho3d/install/web script/dockerized.sh web bash
script/cmake_generic.sh "build/dockerized-web" -D WEB=1 -D URHO3D_HOME=/home/urho3d/.urho3d/install/web
cd build/dockerized-web
make
```

# How to browse the code:

The "main" can be found where the URHO3D_DEFINE_APPLICATION_MAIN(..applicationName..) macro is.
It expands to something like:

    int RunApplication()
    {
      Urho3D::SharedPtr<Urho3D::Context> context(new Urho3D::Context());
      Urho3D::SharedPtr<className> application(new className(context));
      return application->Run();
    }

src/Urho2DPlatformer:
- the "main"
- provides *Setup()* and *Start()* method to the engine, so that the loop can be initiated
- it uses much of the *Sample2D* class methods to create scene nodes (such as characters), populate the scene, and play effects.
- NOTE:
  - Urho2DPlatformer is a subclass of Sample (see below)
  - Sample is a subclass of Application, the Urho3D base class **needed** to initiate the engine

src/Sample:
- class used to setup the window in which the game will run
- e.g. **DebugHud** is instantiated and configured here (with F2 key we toggle all profilers)

src/Utilities2D/Sample2D:
- helper class, used to create scene nodes (such as characters), populate the scene, and play effects.
- has methods to instantiate sprites of characters, etc.
- *Sample2D* is used to avoid that the main *Urho2DPlatformer* gets too big.

src/Character2D:
- class used to represent the main player / hero of the game (Imp)
- derives from Urho3D::LogicComponent class
- contains some fields as boolean parameters such as _isKilled, etc.
- override Update() method to update the behavior of the character at each frame

src/Utilities2D/Mover:
- Mover logic component
  - Handles entity (enemy, platform...) translation along a path (set of Vector2 points)
  - Supports looping paths and animation flip
  - Default speed is 0.8 if 'Speed' property is not set in the tmx file objects

bin/CoreData/ (partly copied from engine folder):
- contain resources needed from the engine. See [documentation](https://urho3d.github.io/documentation/1.4/_main_loop.html).
- this is not something an end user needs to write,
  but when running the an application linked to Urho3D engine
  expects to find these files.

bin/Data:
- contains Fonts, Scenes, Sounds, Textures, characters pngs, ...

- bin/Data/Textures/
  - contains pics used for background, logos, etc.

- bin/Data/Scenes/
  - **NOTE**: this folder and the contained xml file is currently **generated** by the cpp program. See *Urho2DPlatformer::HandleSceneRendered* and *Urho2DPlatformer::CreateScene*.
  Info on how to create the inital scene are taken from *bin/Data/Urho2D/Tilesets/* tmx file,
  see below.
  - contains the main scene xml to be created/edited/opened with **Editor**

- bin/Data/Urho2D/ contains pngs for the characters, etc. NOTE: sprites and animation are generated using [Spriter](http://www.brashmonkey.com/spriter.htm).

    E.g.

        Sample2D::CreateCharacter(...)
          // Get scml file and Play "idle" anim
          auto* animationSet = cache->GetResource<AnimationSet2D>("Urho2D/imp/imp.scml");
          // scml files specifies hoe the png pieces are connected to create a character, // it describes animations (i.e. how png pieces moves in different circumstances), etc.

- bin/Data/Urho2D/Tilesets/
  - contains blocks used to create the tile (png files)
  - contains a tmx file describing how to interpret the png and **how to place the blocks to create the level**.

# How to use the tools around Urho3D engine

## Editor

The editor is found in *URHO3D_HOME/build/bin/Editor.sh*. This is a script that executes the following tool:

```
Urho3DPlayer Scripts/Editor.as
```

where *Editor.as* in an AngelScript script.

When executing Editor.sh, the editor opens and we can select the scene (e.g. *bin/Data/Urho2D/Platformer2D.xml*).

## Profiler and Tracy

Urho3D contains a profiler library, which is built by default
when the engine is built.

Results from the profiler can be shown and interpreted using
the [DebugHud](https://github.com/urho3d/Urho3D/wiki/DebugHud-and-Profiling),
an inbuilt utility for profiling, measuring and displaying scene,
memory or other runtime statistics.
To see DebugHud in action use F2 and F3 on Urho3D’s samples.

An alternative to the default profiler is [Tracy](https://github.com/wolfpld/tracy).
To use Tracy, we need first to build **both** the engine and the game
using the option URHO3D_TRACY_PROFILING:

    cd script & ./cmake_generic.sh ../build -DURHO3D_TRACY_PROFILING=1

NOTE: URHO3D_PROFILING will be automatically turned off when URHO3D_TRACY_PROFILING is on.

Then we can build and start the Tracy server (a.k.a. profiler):

```
sudo apt-get install libglfw3-dev
sudo apt-get install libfreetype-dev
sudo apt-get install libcapstone-dev
sudo apt install build-essential g++ cmake git-all \
    liblzma-dev zlib1g-dev libbz2-dev liblzma-dev \
    libboost-date-time-dev libboost-program-options-dev \
    libboost-system-dev libboost-filesystem-dev \
    libboost-iostreams-dev
sudo apt-get install libgtk-3-dev
git clone https://github.com/wolfpld/tracy.git
cd tracy/profiler/build/unix
make release
./Tracy-release
```

See [install instructions](https://www.gear-genomics.com/docs/tracy/installation/#installation-from-source)
and [pull request description](https://discourse.urho3d.io/t/tracy-profiler-integration-for-urho3d/6627/17)

# 2D Physics

## Body collision example:

Urho2D implements rigid body physics simulation using the [Box2D](http://box2d.org/manual.pdf) library.

Urho2DPlatformer.cpp:
```
void Urho2DPlatformer::SubscribeToEvents()
{
    ...
    // Subscribe to Box2D contact listeners
    SubscribeToEvent(E_PHYSICSBEGINCONTACT2D, URHO3D_HANDLER(Urho2DPlatformer, HandleCollisionBegin));
    ...
}
```

In the **engine**:

Source/Urho3D/Urho2D/PhysicsWorld2D.cpp:
```
// Implement b2ContactListener from Box2D.
// BeginContact is called when two fixtures begin to touch.
//
// See b2Contact::Update(b2ContactListener* listener), which calls BeginContact!
// How Box2D identifies collisions (raycasting or other tech is still unclear to me)
void PhysicsWorld2D::BeginContact(b2Contact* contact)
{
    ...
    beginContactInfos_.Push(ContactInfo(contact));
    ...
}

void PhysicsWorld2D::SendBeginContactEvents()
{
  for (unsigned i = 0; i < beginContactInfos_.Size(); ++i)
  {
      ContactInfo& contactInfo = beginContactInfos_[i];
      eventData[P_BODYA] = contactInfo.bodyA_.Get();
      ...
      SendEvent(E_PHYSICSBEGINCONTACT2D, eventData);
      ...
  }
}
```

## Raycast example:

Source/Urho3D/Urho2D/PhysicsWorld2D.cpp:
```
// Return in "result" parameter
void PhysicsWorld2D::RaycastSingle(PhysicsRaycastResult2D& result, const Vector2& startPoint, const Vector2& endPoint, unsigned collisionMask)
{
    ...
    SingleRayCastCallback callback(result, startPoint, collisionMask);
    world_->RayCast(&callback, ToB2Vec2(startPoint), ToB2Vec2(endPoint));
}
```

Box2D/Collision/Shapes/b2collision.cpp - Similar to the one I developed:
```
// Collision Detection in Interactive 3D Environments by Gino van den Bergen
// From Section 3.1.2
// x = s + a * r
// norm(x) = radius
bool b2CircleShape::RayCast(b2RayCastOutput* output, const b2RayCastInput& input,
							const b2Transform& transform, int32 childIndex) const
{
  ... similar to the one I developed...
}
```

# References

[Simpler instuctions than official documentation](https://github.com/urho3d/Urho3D/wiki)  
[Unofficial wiki with nice links](https://urho3d.fandom.com/wiki/Unofficial_Urho3D_Wiki)  
[Blog describing the superbeginner code and basics](https://darkdove.proboards.com/thread/30/urho-flow-1)  
[Unofficial wiki - First project (3D), very good comments in code](https://urho3d.fandom.com/wiki/First_Project)  
[Blog describing how to use the Editor](https://darkdove.proboards.com/thread/15/urho3d-editor-simple-guide)
