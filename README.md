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
6. create main.cpp
7. cd script & ./cmake_generic.sh ../build
8. cd ../build & bear make

To create a debuggable binary, use the following in step 7:

7. cd script & ./cmake_generic.sh ../build -DCMAKE_BUILD_TYPE=Debug

# How to browse the code:

The "main" can be found where the URHO3D_DEFINE_APPLICATION_MAIN(..applicationName..) macro is.
It expands to something like:

    int RunApplication()
    {
      Urho3D::SharedPtr<Urho3D::Context> context(new Urho3D::Context());
      Urho3D::SharedPtr<className> application(new className(context));
      return application->Run();
    }

Urho2DPlatformer:
- the "main"

Mover:
- TODO

Sample2D:
- helpers for 2D games, including sprites of characters, etc.

Character2D:

Sample:
- class used to setup the window in which the game will run
- e.g. DebugHud is instantiated and configured here (with F2 key we toggle all profilers)

bin/ (partly copied from engine folder) contains:

- Data/Scenes/ contains the main scene xml to be created/edited/opened with Editor

- Data/Urho2D/ contains pngs for the characters, etc.

  E.g.

      Sample2D::CreateCharacter(...)
        // Get scml file and Play "idle" anim
        auto* animationSet = cache->GetResource<AnimationSet2D>("Urho2D/imp/imp.scml");
        // scml files specifies png, etc.

# References

[Simpler instuctions than official documentation](https://github.com/urho3d/Urho3D/wiki)  
[Unofficial wiki with nice links](https://urho3d.fandom.com/wiki/Unofficial_Urho3D_Wiki)  
[Blog describing the superbeginner code and basics](https://darkdove.proboards.com/thread/30/urho-flow-1)