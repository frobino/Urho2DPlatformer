# How to build the engine

    sudo apt-get install libx11-dev libxrandr-dev libasound2-dev git cmake make libgl1-mesa-dev
    git clone https://github.com/urho3d/Urho3D
    cd Urho3D

Alternative 1:

    mkdir build & cd build & cmake ../ & make

Alternative 2 (similar to project creation):

    cd script & ./cmake_generic.sh ../build -DCMAKE_BUILD_TYPE=Debug
    cd ../build & bear make

Before building a game linked to Urho3D do always add the following env var:

    export URHO3D_HOME=/home/${USER}/Projects/Urho3D/build

# How to create project:

1. mkdir Urho3DSampleProject & cd Urho3DSampleProject
2. cp -r /home/user/Projects/Urho3D/bin .
3. cp -r /home/user/Projects/Urho3D/CMake .
4. cp -r /home/user/Projects/Urho3D/script .
5. create CMakeLists.txt
6. create src/ folder and put inside main.cpp
7. cd script & ./cmake_generic.sh ../build
8. cd ../build & bear make

To create a debuggable binary, use the following in step 7:

7. cd script & ./cmake_generic.sh ../build -DCMAKE_BUILD_TYPE=Debug

TODO: add info on building for different backends

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
- TODO

src/Utilities2D/Mover:
- TODO

bin/CoreData/ (partly copied from engine folder):
- TODO

bin/Data:
- contains Fonts, Scenes, Sounds, Textures, characters pngs, ...

- bin/Data/Textures/
  - contains pics used for background, logos, etc.

- bin/Data/Scenes/
  - contains the main scene xml to be created/edited/opened with **Editor**
  - NOTE: this xml file is cuurently **generated** by the cpp program. See Urho2DPlatformer::HandleSceneRendered.

- bin/Data/Urho2D/ contains pngs for the characters, etc. NOTE: sprites and animation are generated using [Spriter](http://www.brashmonkey.com/spriter.htm).

    E.g.

        Sample2D::CreateCharacter(...)
          // Get scml file and Play "idle" anim
          auto* animationSet = cache->GetResource<AnimationSet2D>("Urho2D/imp/imp.scml");
          // scml files specifies hoe the png pieces are connected to create a character, // it describes animations (i.e. how png pieces moves in different circumstances), etc.

- bin/Data/Urho2D/Tilesets/
  - contains blocks used to create the tile (png files)
  - contains a tmx file describing how to interpret the png and **how to place the blocks to create the level**.

# References

[Simpler instuctions than official documentation](https://github.com/urho3d/Urho3D/wiki)  
[Unofficial wiki with nice links](https://urho3d.fandom.com/wiki/Unofficial_Urho3D_Wiki)  
[Blog describing the superbeginner code and basics](https://darkdove.proboards.com/thread/30/urho-flow-1)  
[Unofficial wiki - First project (3D), very good comments in code](https://urho3d.fandom.com/wiki/First_Project)  
[Blog describing how to use the Editor](https://darkdove.proboards.com/thread/15/urho3d-editor-simple-guide)