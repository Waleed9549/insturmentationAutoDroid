## Why
To generate JaCoCo integration teast coverage reports for Android projrcts, you need to create a report of code coverage. This 
needs to configure JaCoCo plugin, providing paths to source code and build the application. This can be troublesome since Android 
projects can have different build types thus requiring additional paths to be set. This document describes how to configure the 
applications to produce an instrumented APK that can be used for testing purposes. Please make sure to check following things 
before you get started:
1. Follow the setup instructions.
2. Make sure that APK can be built without any errors and it runs properly on and Android device/Emulator. 

## Usage
#### 1. Configure build.gradle (app) to add plugin
```
apply plugin: 'jacoco'
jacoco {
    toolVersion = "0.8.5"
}
```
#### 2. Inside buildTypes enable the code coverage
```
builType{

   debug{ 

     testCoverageEnabled true

}}
```
#### 3. Add class path to build.gradle (project) and make sure to use the same version as app build.gradle
```
classpath "org.jacoco:org.jacoco.core:0.8.5" 
```
#### 4.1 Add a onStop() function to MainActivity.java
```
@Override

protected void onStop()
{
 Log.d("StorageSt", Environment.getExternalStorageState());
 String coverageFilePath = "/mnt/sdcard/<USE APP DIRECTORY NAME>/coverage.ec"
 File coverageFile = new File(coverageFilePath);
 super.onStop();
 if(BuildConfig.DEBUG)
 {
    try
    {
      Class<?> emmaRTClass = Class.forName("com.vladium.emma.rt.RT");
      Method dumpCoverageMethod = emmaRTClass.getMethod("dumpCoverageData",coverageFile.getClass(),boolean.class,boolean.class);
      dumpCoverageMethod.invoke(null, coverageFile,true,false);
    }
    catch (Exception e) {}
  }
 }
```
#### 4.2 Add this inside onCreate() function:
```
if(ContextCompat.checkSelfPermission(MainActivity.this,
        Manifest.permission.READ_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED){
    // Toast.makeText(MainActivity.this,"Read Granted");
} else{
    requestWritePermission();
}
File directory = new File(Environment.getExternalStorageDirectory() + java.io.File.separator +"<USE APP DIRECTORY NAME>");
if (!directory.exists()) {
    try {
        directory.mkdir();
        Toast.makeText(this, "Directory Created", Toast.LENGTH_SHORT).show();

    } catch (Exception e) {
        Toast.makeText(this, "Create Directory Failed", Toast.LENGTH_SHORT).show();
    }

}
else
    Toast.makeText(this, "Directory Exists", Toast.LENGTH_SHORT).show();
```
#### 4.3 Add write permission outside onCreate() function:
In some Android versions, you must provide dynamic write permission. To be on a safer side, you may provide write permission in all the cases. First add a new variable in the mainactivity class and add the following lines inside onCreate Method:

```
private void requestWritePermission(){

    ActivityCompat.requestPermissions(this, new String[] {Manifest.permission.WRITE_EXTERNAL_STORAGE}, WRITE_STORAGE_PERMISSION_CODE);

}
```
Initialise variable WRITE_STORAGE_PERMISSION_CODE by adding :
```
private int WRITE_STORAGE_PERMISSION_CODE = 1;
```
#### 5. Add write permission to AndroidManifest.xml file

``` <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> ```

#### 6.   Create a new broadcast receiver class "CoverageReceiver"
        Right click package name>New>Other>Broadcast Receiver
#### 6.1    Inside onReceive method of the CoverageReceiver class add the folowing lines
    if("com.context.FINISH_TESTING".equals(intent.getAction())){
        String coverageFilePath = "/mnt/sdcard/<USE APP DIRECTORY NAME>/coverage.ec";
        File coverageFile = new File(coverageFilePath);
        try {
            Class<?> emmaRTClass = Class.forName("com.vladium.emma.rt.RT");
            Method dumpCoverageMethod = emmaRTClass.getMethod("dumpCoverageData",coverageFile.getClass(),boolean.class,boolean.class);
            dumpCoverageMethod.invoke(null, coverageFile,true,false);
            Toast.makeText(context,"File created",Toast.LENGTH_SHORT).show();
        }
        catch (Exception e) {
            Toast.makeText(context,"File not created",Toast.LENGTH_SHORT).show();
            Toast.makeText(context, e.getCause().toString(),Toast.LENGTH_LONG).show();
        }
    }
#### 6.2     Create an instance of the Coverage Receiver in the launcher activity
    CoverageReceiver coverageReceiver = new CoverageReceiver();
#### 6.3     Register Coverage Receiver inside onCreate Method of launcher activity
    IntentFilter filter = new IntentFilter("com.context.FINISH_TESTING");
    registerReceiver(coverageReceiver, filter);
   

### Build the project after finishing these steps and generate the instrumented APK file.

###  


## Test the APK

Install the APK on an emulator and after running tests send a broadcast using terminal: adb shell am broadcast -a com.context.FINISH_TESTING. This will create a coverage file with extension ec inside /mnt/sdcard/<package-name>/coverage.ec
which can be pulled using command  ```adb pull <source_path> (/mnt/sdcard/<project-package-name>/coverage.ec) <destination_path> (app/build/output/<project-package-name>/coverage.ec) ```
Make sure that the file size is non zero.

Multiple ec files can be created using this process and the Android jar can help in obtaining code coverage report. 






