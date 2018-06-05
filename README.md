### Overview
HyperSnapSDK is HyperVerge's documents + face capture framework that captures images at a resolution appropriate for our proprietary Deep Learning OCR and Face Recognition Engines.

### Prerequisites
- Gradle Version: 4.1 (Recommended)
- Tested with Gradle Plugin for Android Studio - version 3.0.1 
- minSdkVersion 19
- targetSdkVersion 26

### Sample
- Please refer to the sample app provided in the repo to get an understanding of the implementation process.
- To run the app, clone/download the repo and open **sample** using latest version of Android Studio
- Open project build.gradle and replace **aws_access_key** and **aws_secret_pass** with the credentials provided by HyperVerge
- Build and run the app

### Integration Steps
- Add the following set of lines to your `app/build.gradle`

  ```groovy
  android {
      defaultConfig {
          renderscriptTargetApi 19
          renderscriptSupportModeEnabled true
	  }
  }
  dependencies {
      implementation('co.hyperverge:hypersnapsdk:1.0.8@aar', {
          transitive=true
          exclude group: 'com.android.support'
      })
  }
  ```
- Add the following set of lines to the Project (top-level) `build.gradle`

  ```groovy
  allprojects {
      repositories {
          maven {
              url "s3://hvsdk/android/releases"
              credentials(AwsCredentials) {
                  accessKey aws_access_key
                  secretKey aws_secret_pass
              }
          }
      }
  }
  ```
  where **aws_access_key** and **aws_secret_pass** will be given by HyperVerge
- **Permissions**: The app requires the following permissions to work.
    - *Camera*
    - *Autofocus*
    - *Read and Write external storage*

    Kindly note that for android v23 (Marshmallow) and above, you need to handle the runtime permissions inside your app.

- Add the following line to your Application class (the class which extends android.app.Application) for initializing our Library. This must be run only once. Check [this](https://guides.codepath.com/android/Understanding-the-Android-Application-Class) link if you are unsure of what an Application class is. Here appId and appKey are the credentials that have been supplied by us.
  ```java
  HyperSnapSDK.init(context, APP_ID, APP_KEY);
  ``` 
- **Capturing Face**: For capturing face image, following method should be called:
  ```java
  FaceCaptureActivity.start(context, LivenessMode.MODE, myCaptureCompletionListener);  
  ```
  where:
  - **context** is the context of the current Activity being displayed
  - **myCaptureCompletionListener** is an object of `CaptureCompletionHandler` and has been described later
  - **LivenessMode.MODE** is an enum with 3 values.
  - **LivenessMode.NONE** No liveness test is performed. The selfie that is captured is simply returned. If successful, the 	result dictionary in the completion handler has one key-value pair. 
	-**imageUri** : local path of the image capture

  -**LivenessMode.TEXTURELIVENESS** : Texture liveness test is performed on the selfie captured.  If successful, the result dictionary has two key-value pairs. 
	- **imageUri** : local path of the image capture 
	- **isLive** : Boolean value. Tells whether the selfie is that of a real person or a photograph


-**LivenessMode.TEXTUREANDGESTURELIVENESS** : In this mode, based on the results of the texture Liveness call, the user might be asked to do a series of gestures to confirm liveness. The user performing the gestures is arbitrarily matched with the selfie captured. If  one or more of these matches fail, a 'faceMatch' error is returned (refer to 'Error Codes' section). If all the gestures are succefully performed and the face matches are sucessful, a result similar to the .textureLiveness mode is returned.

- **Capturing Document**: For capturing document crop image(based on Aspect Ratio), following method should be called:

  ```java
  DocumentActivity.start(context, document, myCaptureCompletionListener);
  ```
  where:
  - **context** is the context of the current Activity being displayed
  - **document** is an enum variable of type co.hyperverge.hypersnapsdk.objects.Document. It is described in detail later.
  - **myCaptureCompletionListener** is an object of `CaptureCompletionHandler` and has been described later
- **Document Parameter**: In order to specify the document that needs to be captured using DocumentActivity of the SDK, a document needs to be passed with the start method of DocumentActivity. The parameter can be initialized as:
  ```java
  Document document = Document.CARD;
  ```
  
  Following are the type of documents supported by Document enum:
    - **CARD**: Aspect ratio : 0.625. Example: Vietnamese National ID, Driving License, Motor Registration Certificate
    - **PASSPORT**: Aspect ratio: 0.67. Example: Passports
    - **A4**: Aspect ratio: 1.4. Example: Bank statement, insurance receipt
    - **OTHER**: This is for aspect ratios that don't fall in the above categories. In this case, the aspect ratio should be set in the next line by calling `document.setAspectRatio(aspectRatio);`
      where `aspectRatio` is a float specifying the aspectRatio of the document
  
  Also, Document supports following customizations:
  - **message**: Message is the text displayed at the top section of the Camera Preview in DocumentActivity screen. Message is meant to educate user about the document type. The text can be altered by calling following method:
    ```java
    document.setMessage("Driving Licence Front Side");
    ```
  - **instruction**: Instruction is the text displayed at the bottom end of the Camera Preview in DocumentActivity screen. Instructions are meant to educate user about the box present in the camera screen of Document Activity and its significance. The text can be altered by calling following method:
    ```java
    document.setInstruction("Place your Driving License inside the Box");
    ```
- **CaptureCompletionHandler**: CaptureCompletionHandler is an interface whose object needs to be passed with start method of both FaceCaptureActivity and DocumentActivity. It has methods which has to be implemented by the object to handle both the responses of document capture and the errors that occured during capture. Following is a sample implementation of CaptureCompletionHandler:
  ```java
  CaptureCompletionHandler myCaptureCompletionListener = new CaptureCompletionHandler() {
    @Override
    public void onResult(CaptureError error, JSONObject result) {
        if(error != null) {
            Log.e("LandingActivity", error.getError() + " :: " + error.getErrMsg());
        }
        else{
            Log.i("Landing Activity", result.toString());
            //result will have following keys:
            //    • imageUri: String path of the captured image
        }
    }
  }
  ```
  Following are the errors that can occur during capture process:
  
  |Description|Explanation|Action|
  |-----------|-----------|------|
  |INTERNAL_SDK_ERROR|Occurs when an unexpected error has happened with the HyperSnapSDK.|Notify HyperVerge|
  |OPERATION_CANCELLED_BY_USER_ERROR|When the user taps on cancel button before capture|Try again.|
  |PERMISSIONS_NOT_GRANTED_ERROR|Occurs when the relevant Android permissions are not given.|Ensure that the necessary permissions are given by user.|
- **Customizations**: Some text fields, element colors, font styles, and button icons are customizable so that the look and feel of the components inside SDK can be altered to match the look and field of the app using the SDK. **Kindly note that this step is optional**. Below is the list of items that are customizable grouped by the resource file/type where the customized value/file(s) should be placed in.
    - **strings.xml**:
      ```xml
      <string name="document_screen_title_text">Document Scanner</string>
      <string name="face_screen_title_text">Face Scanner</string>
      <string name="place_face">Place your face within circle</string>
      <string name="stay_still"> Capture Now </string>
      ```
### Contact Us
If you are interested in integrating this SDK, please do send us a mail at [contact@hyperverge.co](mailto:contact@hyperverge.co) explaining your use case. We will give you the `aws_access_key` & `aws_secret_pass` so that you can try it out.
      
