# Facebook Integration into Cocos2d-X Project
Facebook is implemented as a Cocos2d-X Plugin. 

## Common
1. Create Facebook Application

## IOS
1. XCode.
  1. Add "PluginProtocol.xcodeproject" and "PluginFacebook.xcodeproject".
  2. Add "FacebookSDK.framework" to Build Phases.
  3. Add "libPluginProtocol.a" and "libPluginFacebook.a" static libraries to "Link Binary With Libraries" section.
2. Info.Plist
  1. Add `FacebookAppID` and `URL Schemes` in Info.plist.
3. Add `-ObjC` flag in `Other Linker Flags`.
4. Add path to "PluginProtocol"/include in `User Header search` paths.
4. AppController Modifications

```
...
#import <FacebookSDK/FacebookSDK.h>
...
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    return [FBSession.activeSession handleOpenURL:url];
}
...
- (void)applicationDidBecomeActive:(UIApplication *)application {
    [FBAppCall handleDidBecomeActive];
    cocos2d::Director::getInstance()->resume();
}
```

## Android
1. Generate Publish Folder for Plugins.
  1. Ensure you have both android-7 and android-21 in your SDK. If not, fire up SDK Manager and download android-7. Step 2 fails unless this is done.
  2. Go to `/cocos2d/plugin/tools` and run `publish.sh`. 
2. Add JARS from 'plugin-x' (now visible in eclipse) to BuildPath.
3. Add same jars into build.xml
4. Create "DependProject/src" folder manually.
5. Modify AppActivity.java
6. Make sure all JARS have export ON.
```
...
import org.cocos2dx.lib.Cocos2dxGLSurfaceView;
import org.cocos2dx.plugin.PluginWrapper;
import org.cocos2dx.plugin.FacebookWrapper;
import android.content.Intent;
import android.os.Bundle;
...
public class AppActivity extends Cocos2dxActivity {
    @Override
    public Cocos2dxGLSurfaceView onCreateView() {
        Cocos2dxGLSurfaceView glSurfaceView = new Cocos2dxGLSurfaceView(this);
        // TestCpp should create stencil buffer
        glSurfaceView.setEGLConfigChooser(5, 6, 5, 0, 16, 8);

        PluginWrapper.init(this);
        PluginWrapper.setGLSurfaceView(glSurfaceView);
        FacebookWrapper.onCreate(this);
        return glSurfaceView;
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if(!PluginWrapper.onActivityResult(requestCode, resultCode, data))
        {
            super.onActivityResult(requestCode, resultCode, data);
        }
        FacebookWrapper.onAcitivityResult(requestCode, resultCode, data);
    }

    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        FacebookWrapper.onSaveInstanceState(outState);
    }
}
```

