# Plugin-X Integration Guide for Android

# Plugin-X integration in the android

[Integrate third-party SDK] (http://www.cocos2d-x.org/wiki/Third_Party_SDK_Integration)

## Environmental needs

1. python 2.7. [http://www.python.org/](http://www.python.org/)
2. Apache Ant build tool. Http://ant.apache.org/
3. If your operating system is Windsws, you need to install Cygwin environment.

## Compilation Plugin-X project
1. Run publish.sh script path for plugin-x / tools / publish.sh. This script in Mac OSX Mountain Lion v10.8.2 and Cygwin on Windows 7 test well in the first run when the script, it will allows you to enter some parameters to set up the environment, as follows

![xx] (res / plugin-x-setting-environment.jpg)

On Windows, the path you enter the path should be entered in the form of Linux Example: You can enter C: / adt-bundle-windows / sdk, instead of C: \ adt-bundle-windows \ sdk

2. After compilation is complete, a file named publish folder in the root directory will be generated plugin-x project. If you publish a folder in the following figure, that means the success of your compilation.

! [xx] (res / plugin-x-publish-folder.jpg)

3. publish folder contains the following files:
- Plugin header files (* .h)
- C ++ static library (* .a)
- Java libraries (* .jar)
- Makefiles for android (* .mk)
- Some plug-android project needs

## Before using the plug-in editor game project
### What do we need?

Plugin-x android SDK for third-party package for C / C ++ interface to the game so we need to edit the project settings in order to use it:

- Static library editor .mk file (Android.mk & Application.mk) to link created by the plugin-x.
- Edit Project Settings android sdk library and third-party libraries to link created by the plugin-x (.jar file).
- Edit AndroidManifest.xml, add the statement of activities, and games use rights.
- For individual plug-ins, we need to add additional settings and resource files.

### Game developers guide tool
There is a script called gameDevGuide.sh path plugin-x / tools / gameDevGuide.sh.
This script will help you edit your game project.

1. Run the script in a terminal (on Windows, you should use Cygwin to run it) UI interface is as follows:
! [xx] (res / plugin-x-guide-UI.jpg)

Enter the path in the edit-box inside your game android project, please ensure that the path is no space (Translator's Note: The path under windows "\" to "/", otherwise, it can be normal to the next step, but to continue to be wrong) and then click on 'Next' button.

2. Select the plug-in you need a UI interface is as follows:

! [xx] (res / plugin-x-guide-UI2.jpg)
After you select the plug-in, click 'Finish' button. The script will edit your android projects in several configuration files.

The following configuration file will be edited:

- Android.mk -------------- Add game dependent C ++ library
- Application.mk ----------- add parameters for ndk build
- .project ----------------- Publish a link to the game works as a resource directory
- .classpath --------------- Add java game dependent libraries
- AndroidManifest.xml ------ Add the declaration required plug-game activity and user rights

## Manually edit
1. Edit ndk-build command parameters: Adding publish directory to NDK_MODULE_PATH parameters, as follows:
`NDK_MODULE_PATH = $ {PLUGIN_ROOT} / publish`

2. Add code to the JNI_OnLoad method, as follows:

`` `
#include "PluginJniHelper.h"
jint JNI_OnLoad
{
  JniHelper :: setJavaVM;
  PluginJniHelper :: setJavaVM; // for plugins
  return JNI_VERSION_1_4;
}
`` `

3. When the game's main activity is created, call the PluginWrapper.init () method:

`` `
import org.cocos2dx.plugin.PluginWrapper;
import org.cocos2dx.lib.Cocos2dxGLSurfaceView;
public class HelloIAP extends Cocos2dxActivity {
  protected void onCreate {
  super.onCreate ();
  PluginWrapper.init (); // for plugins
  // If you want your callback function can be invoked in GL thread, add this line:
  PluginWrapper.setGLSurfaceView ();
  }
  static {
  System.loadLibrary;
  }
}
`` `

4. A special plug-modified item
If you look at the game using plug-ins, you may need some special modifications.
We have plans to add these items to the developer to modify the guidance tools inside.

- For nd91
1. Import android project `91SDK_LibProject_complete` to the eclipse. Engineering path is` plugin-x / publish / nd91 / andriod / DependProject`

2. Set your game project relies on `91SDK_LibProject_complete` project, as shown below:

! [xx] (res / plugin-x-nd91-depend.jpg)

## Using the plugin-x in your C ++ code
### Loading and unloading plugins

All plug-ins by class PluginManager management class name you can plug load / unload plugins, sample code:

`` `
     // Load plugin AnalyticsFlurry
     s_pFlurry = dynamic_cast
     (PluginManager :: getInstance () -> loadPlugin ("AnalyticsFlurry"));

     // Unload plugin AnalyticsFlurry
     PluginManager :: getInstance () -> unloadPlugin ("AnalyticsFlurry");
     s_pFlurry = NULL;
`` `

Recommendation: When the game starts loading plugins, uninstall the plug end of the game.
### Plug-ins
You can call the methods declared in the protocol sample code:

`` `
     // Enable the debug mode
     s_pFlurry-> setDebugMode (true);

     // Log an event
     s_pFlurry-> logEvent ("music");

     // Log an event with params
     LogEventParamMap paramMap;
     paramMap.insert (LogEventParamPair ("type", "popular"));
     paramMap.insert (LogEventParamPair ("artist", "JJLin"));
     s_pFlurry-> logEvent ("music", ¶mMap);
`` `

Some plugins have custom method. We implemented a simple reflection of the plugin-x.
You can call a custom method name. Like this:

`` `
     // Invoke setAge () method in plugin Flurry
     PluginParam pParam3 (20);
     s_pFlurry-> callFuncWithParam ("setAge", & pParam3, NULL);

     // Invoke logTimedEventBeginWithParams () method in plugin Flurry
     PluginParam event3 ("music-kv");
     LogEventParamMap paramMap;
     paramMap.insert (LogEventParamPair ("type", "popular"));
     paramMap.insert (LogEventParamPair ("artist", "JJLin"));
     PluginParam mapValue (paramMap);
     s_pFlurry-> callFuncWithParam ("logTimedEventBeginWithParams", & event3, & mapValue, NULL);
`` `

Also, here are some basic types of reflective interface defines the return value. Methods listed below:

`` `
     // Return string value for developers
     const char * callStringFuncWithParam (const char * funcName, PluginParam * param, ...);

     // Return int value for developers
     int callIntFuncWithParam (const char * funcName, PluginParam * param, ...);

     // Return bool value for developers
     bool callBoolFuncWithParam (const char * funcName, PluginParam * param, ...);

     // Return float value for developers
     float callFloatFuncWithParam (const char * funcName, PluginParam * param, ...);
`` `

Note:

- You can pass parameters in the method, the type parameter must be `PluginParam` pointer.
- Plug-in parameters must be passed in order
- If the method is not found in the plug-in, nothing will happen.


## Reference
1. screenshots and sample code examples from the plugin-x project HelloIAP
2. If you have any questions, please do not want to comment on this wiki, or create issues on github.com/chukong/plugin-x/ project.
