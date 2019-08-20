# GDNative based OpenVR plugin for Godot

This is a GDNative based plugin that adds OpenVR support to Godot.

The leading version of this repository now lives at:
https://github.com/GodotVR/godot_openvr

**note** The master on this repository is now kept in sync with the Godot master.
To build versions for official releases of Godot please check the branches that are named in sync with the Godot release.

Submodules
----------
This project references two submodules.
If you do not already have these repositories downloaded somewhere you can execute:
```
git submodule init
git submodule update
```
To download the required versions.

You may need to run these commands in the `godot-cpp` subfolder to obtain the correct version of `godot_headers`

Godot_cpp is a git repository that implements C++ bindings to Godots internal classes.

OpenVR is a git repository maintained by Valve that contains the OpenVR SDK used to interact with the OpenVR/SteamVR platform.

Alternatively you can use the switch openvr to the location where you have downloaded a copy of this SDK by adding `openvr_path=<path>` when running scons.

Compiling
---------
Scons is used for compiling this module. I made the assumption that scons is installed as it is also used as the build mechanism for Godot and if you are building from source you will likely need to build Godot as well.

You must compile `godot-cpp` first by executing:
```
cd godot-cpp
scons platform=windows target=release generate_bindings=yes bits=64
cd ..
```

You can compile this module by executing:
```
scons platform=windows target=release
```

Platform can be windows, linux or osx. OSX is untested.

Deploying
---------
Note that besides compiling the GDNative module you must also include valves openvr_api.dll (windows), libopenvr_api.so (linux) or OpenVR.framework (Mac OS X). See platform notes for placement of these files.
The godot_openvr.dll or libgodot_openvr.so file should be placed in the location the godot_openvr.gdnlib file is pointing to (at the moment bin).

Mac notes
---------
Mac is currently untested, I unfortunately do not have the required hardware. If anyone wants to hold up their hands, please contact me :)

Linux notes
-----------
On Linux, Steam will not automatically add the SteamVR libraries to your $LD_LIBRARY_PATH variable. As such, when starting Godot (for development) or your game outside of Steam, it will fail to load the required libraries.

There are a couple of ways to fix this:

1) Launch Godot or your game from within Steam

2) Run Godot or your game through the steam runtime manually (change the path to suit your Steam installation):

```
/home/<user>/.steam/steam/ubuntu12_32/steam-runtime/run.sh <your command>
```

3) Adjust your $LD_LIBRARY_PATH variable accordingly:

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:"/path/to/libgodot_openvr.so/dir/":"/home/<user>/.steam/steam/steamapps/common/SteamVR/bin/"
```

You can place the libopenvr_api.so file alongside the libgodot_openvr.so file in the bin folder. You can find this file in: openvr/bin/linux64

Windows notes
-------------

Windows generally works without any special needs. I've tested compiling with MSVC 2017, others have tested 2015. With 2017 be aware Microsoft now lets you pick and choose which components you install and by default it may not install the SDK you need. Make sure to install both the Windows SDK and build tools.

Also when deploying users may need to first install the correct redistributable you can find here: https://support.microsoft.com/en-au/help/2977003/the-latest-supported-visual-c-downloads
I am not 100% sure this is a requirement as it automatically installs this when installing MSVC but past experiences and such... :)

For Windows you need to supply a copy of openvr_api.dll along with your executable which can be found in openvr/bin/win64

HDR support
-----------
OpenVR does not accept Godot's HDR color buffer for rendering, so your scene may receive position and rotation information and display correctly on the desktop but won't render anything inside your headset. 

Valve have a fix in the works which is currently in testing. 

For now you will have to set `hdr` to `false` on your viewport in addition to enabling `arvr`: 

```
func _ready():
    var interface = ARVRServer.find_interface("OpenVR")
    if interface and interface.initialize():
        get_viewport().arvr = true
        get_viewport().hdr = false
```

Note that once SteamVR is updated it expects linear color buffers while Godot converts the render buffer to sRGB. You can replace `get_viewport().hdr = false` with `get_viewport().keep_linear = true` to work around this problem. Everything will look right inside of the headset but the preview on screen will be too dark. We're still working on this.

Shader hickup
-----------------
There are a few moment where OpenVR has a hickup.

One is around the teleporter function which can be solved by adding the `VR_Common_Shader_Cache.tscn` as a child scene to our ARVRCamera. `ovr_first_person.tscn` does this.

For the controllers they use a standard material. Adding a mesh instance with a standard material will ensure the shader is pre-compiled. Again we do this in `ovr_first_person.tscn`. 

However there is still an issue with loading the texture. We need to redo loading of the controller mesh by handing it off to a separate thread.

GLES2 support
-------------
The new GLES2 renderer in Godot 3.1 renders directly to RGBA8 buffers and thus doesn't need the HDR workaround. The GLES2 renderer is also much more lightweight then the GLES3 renderer and thus more suited for VR.

Using the main viewport
-----------------------
The ARVR server module requires a viewport to be configured as the ARVR viewport. If you chose to use the main viewport an aspect ratio corrected copy of the left eye will be rendered to the viewport automatically.

You will need to add the following code to a script on your root node:

```
var interface = ARVRServer.find_interface("OpenVR")
if interface and interface.initialize():
	# turn to ARVR mode
	get_viewport().arvr = true

	# turn HDR off, not needed with the GLES2 renderer
	get_viewport().hdr = false

	# make sure vsync is disabled or we'll be limited to 60fps
	OS.vsync_enabled = false
	
	# up our physics to 90fps to get in sync with our rendering
	Engine.target_fps = 90
```

Using a separate viewport
-------------------------
If you want control over the output on screen so you can show something independent on the desktop you can add a viewport to your scene.

Make sure that you turn the ARVR property of this viewport to true and the HDR property to false.
Also make sure that both the clear mode and update mode are set to always.

You can add a normal camera to your scene to render a spectator view or turn the main viewport into a 2D viewport and save some rendering overhead.

You can now simplify you initialisation code on your root node to:

```
var interface = ARVRServer.find_interface("OpenVR")
if interface:
	interface.initialize()

	# make sure vsync is disabled or we'll be limited to 60fps
	OS.vsync_enabled = false
	
	# up our physics to 90fps to get in sync with our rendering
	Engine.target_fps = 90
```

License
-------
Note that the source in this repository is licensed by the MIT license model. This covers only the source code in this repository. 
Both Godot and OpenVR have their own license requirements. See their respective git repositories for more details.

About this repository
---------------------
This repository was created by and is maintained by Bastiaan Olij a.k.a. Mux213

You can follow me on twitter for regular updates here:
https://twitter.com/mux213

Videos about my work with Godot including tutorials on working with VR in Godot can by found on my youtube page:
https://www.youtube.com/BastiaanOlij
