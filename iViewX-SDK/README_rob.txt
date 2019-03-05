build in build folder (subfolder of lsl git root called "build_rob")

cmake .. -G "Visual Studio 15 2017 Win64" -DQt5_DIR="C:/Qt/Qt5.11.1/5.11.1/msvc2015_64/lib/cmake/Qt5" -DLSLAPPS_iViewX=ON


Currently: Complicated build instructions, with visual studio and command line.

1. Run in command line:

cmake --build . --config Release --target install

(it will fail)

2.1. Open the project file (LabStreamingLayer.sln) in Visual studio
2.2. Right-Click "SMIEyetracker" in the Project Explorer Window -> Nuget Pakete
2.3. In the "Browse" tab, search for "allegro" and install it (little crocodile smybol, allegro5)
2.4. Right-Click "SMIEyetracker" in the Project Explorer Window -> Properties / Eigenschaften,
     Select Allegro 5, Open it and set the "Library Type" to "Static Monolith Release"
2.5. Compile in Visual Studio (will not fail)

3. compile again in command line:
cmake --build . --config Release --target install

works this time!!



For deployment on the notebook we need some 32-Bit libraries that must be placed inside the "SMIEyetracker" folder.

- All dlls (no 64 in name) from the include subfolder in this folder
- dlls from the iViex SDK install location on C:/Programm FIles x86   (bin folder)
- msvcp140.dll and vcruntime140.dll from C:/Windows/SysWOW64/



good to go.
