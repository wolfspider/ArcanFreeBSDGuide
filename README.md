# Installing Arcan on FreeBSD

[Arcan](https://arcan-fe.com/) from the beginning has had good FreeBSD support. This is a guide for installing it on your FreeBSD system and additional tips for getting it configured for graphical acceleration without utilizing Xorg/X11. For the most part the build instructions *just work* as described on the project page and [Wiki](https://github.com/letoram/arcan/wiki/bsd) for Arcan. 

#### Disclaimer: This is a work in progress and will continue to receive updates as things are added

The first thing which needs to be considered is setting up a FreeBSD system with compatible hardware. The main site makes reference to i965 Intel and Nvidia graphics drivers being the main ones tested. The machine which was used for these instructions is a [Dell Optiplex 990](http://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/optiplex-990-spec-sheet.pdf).

FreeBSD was installed from the standard .iso image for the AMD64 platform it is [FreeBSD Release 11.2](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/11.2/)

There are two components we will be looking at regarding setting up Arcan. The first is Arcan itself which can be thought of as a "Frame Server". The Frame Server runs and attaches rendering nodes through it's applications (or appls). We need to build this first and then afterward we will retrieve [Durden](https://github.com/letoram/durden). Durden can be thought of as the application which draws the interface on the screen and communicates with Arcan. Durden doesn't require any compilation at all amazingly and is accessed by Arcan as a parameter at run time. At the time of this writing it is more convenient to install X11/Xorg first to bootstrap the SDL backend in order to set up the keybindings before activating Arcan as a standalone system. My choice initially was to just install [GNOME 3](https://www.freshports.org/x11/gnome3) from ports. For the sake of brevity we will assume X11/Xorg has already been set up. The important item here which needs to be added is the [kernel mode settings package](https://www.freshports.org/graphics/drm-next-kmod/) in order to get up to date drivers for the GPU.  

Instructions for configuring this are located [here](https://www.freebsd.org/doc/handbook/x-config.html#x-config-kms).  

The first thing which is needed is to install dependencies necessary to build from source.

    pkg install cmake sdl openal-soft freetype libGL lua51
    
Now clone the Arcan repo, create the build folder, and generate the build settings with the following commands:

    git clone https://github.com/letoram/arcan.git
    cd arcan
    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE="Debug" -DVIDEO_PLATFORM=sdl
    -DSTATIC_SQLITE3=ON -DSTATIC_OPENAL=ON -DSTATIC_FREETYPE=ON -DDISABLE-JIT=ON ../src
    make -j 12
    
For now, Lua JIT is disabled due to stability issues on FreeBSD reportedly. Dependencies can be statically compiled as well by running the clone.sh file located in the arcan/external/git folder and then building the application.

While building Arcan you should see output in the terminal which verifies the dependencies it has successfully found. There is a CMakeCache.txt file which gets created in the build folder and if there are any questions about whether all the dependencies have been located this is the place to look.

Go back to your user's home directory and clone the Durden repo:

    git clone https://github.com/letoram/durden.git

Make sure you are in X11/Xorg before starting Arcan with the SDL backend. I recommend doing this first before running it with the EGL-DRI backend to make sure it starts up fine before moving forward. To start it run this command:

    ./arcan -p /home/user /home/user/durden/durden -T /home/user/arcan/data/scripts
    
This command should start Durden for the first time and it will attempt to begin binding keys to actions. For information about configuring Durden go [here](http://durden.arcan-fe.com/). You will find an instructional video and information about how to get around.

There are several options for creating windows on the screen for the compositor to arrange like "Browse", "Terminal", "Media" and so on. This interface allows the user to compose a desktop using multiple sources.

Once your settings are saved select the "System" menu option and select "Shutdown". Now that entries have been saved into Arcan's database we can rebuild Arcan for EGL-DRI. This is the optimal mode to run in for executing Arcan-LWA nested instances.

Now that it's verified everything is working shut down X11/Xorg and rebuild Arcan with the egl-dri backend.

    cd /home/user/arcan/build
    cmake -DCMAKE_BUILD_TYPE="Debug" -DVIDEO_PLATFORM=egl-dri
    -DSTATIC_SQLITE3=ON -DSTATIC_OPENAL=ON -DSTATIC_FREETYPE=ON -DDISABLE-JIT=ON ../src
    make -j 12
    
Start Arcan again and now you should have a new desktop environment without X11/Xorg! 

#### Known Issues

At this time Arcan is not detecting the mouse with the egl-dri backend. I'm using a typical logitech wireless keyboard and mouse, however I tried running the Arcan startup command with "sudo" and it did detect the mouse and the sensitivity is too strong. There is code for the mouse in the arcan/src/platform/freebsd folder specific to this which will probably need to be looked at.

Arcan apparently uses /dev/sysmouse in order to access it. In order to make it work as a user disable HALD in /etc/rc.conf, add the user to both groups "wheel" and "operator", and change permissions for the mouse for now.

    sudo chmod 777 /dev/sysmouse
    sudo chmod 777 /dev/ums0

Your mouse may have a different address than "ums0" just substitute that for the second command. Most likely this can be narrowed down and not all of this is needed.

For more information about changing the acceleration of the mouse read "man sysmouse".

 




