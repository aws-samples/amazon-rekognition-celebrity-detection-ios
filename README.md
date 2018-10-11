## Amazon Rekognition Celebrity Detection Ios

Starter iOS Swift project code for identifying celebrities using Amazon Rekognition

### Overview of the starter app

The starter app includes the basic plumbing for capturing a picture using
the device camera, or selecting a photo from the device photo library. It also
includes components that can load a webpage inside a Safari-based
webview.
Also, the starter app includes a Celebrity.swift class that’s used to create
a Celebrity object for each 'face' that’s identified in the provided image. This
object uses results from the Amazon Rekognition API to populate its
properties and identify the location of each identified 'face' in the image. In
particular, it uses the bounding box data for each face that’s identified in
the image, and draws a button at the position of the face. Choosing the
button takes you to a webpage for the celebrity that’s loaded within a Safari
web view.

## Steps

### Step 1
The first step is to configure access for the iOS app to call Amazon Rekognition. We
do this by creating an Amazon Cognito identity pool, and an associated IAM role
with an attached policy that allows our mobile app to call Amazon Rekognition
directly.
The quick start option is an AWS CloudFormation template. The template creates a
CloudFormation stack that provisions an Amazon Cognito identity pool, IAM role,
and policy. After the stack has completed, choose the output tab to view your
identity pool ID. You use this value for "<your-cognito-pool-id>" in step 4.

#### Set up an Amazon Cognito identity pool
1. Go to the Amazon Cognito console, go to Manage Identity Pools, and then
choose Create new Identity Pool.
1. Enter a name that’s recognizable later, such as “My Rekognition App”.
1. Select the check box for Enable access to unauthenticated identities, so
that you don't need to sign in for this demo app.
1. Choose Create Pool, which takes you to the Your Cognito identities require
access to your resources page for assigning a role to your identity pool.

#### Set up an IAM role and policy
The next step is to create a role and attach an IAM policy to this role that gives
access to the Amazon Rekognition API.
To do this, you edit the IAM policy on the Your Cognito identities require access to your
resources page. Choose View Details, choose View Policy Document, choose View
Policy Document, and then choose Edit. You do this for the unauth role, but you
can also do the same thing for the auth role if you’re going to use the app as an
authenticated user.
You can also use a readily available policy template for access to the Amazon
Rekognition API.

1. To do this, choose Allow on the Your Cognito identities require access to
your resources page and head over to the IAM console.
1. In the IAM console, go to the Roles section, and find your unauth role from
the previous step.
1. Choose the role link to go to the role's page.
1. Under the Permissions tab, choose Attach policies, and search for a policy
titled AmazonRekognitionReadOnly. Select the check box next to this
policy, and choose Attach policy.

### Step 2: Download this starter app
The starter app comes with the basic code for capturing a photo and displaying an
image. This allows you to focus on the components that are related to Amazon
Rekognition right away.

### Step 3: Set up the AWS Mobile SDK
1. We use CocoaPods to set up the AWS Mobile SDK for iOS dependencies. Go to
the pod file in the project directory and enter the following under #Pods for
<project name>:
`pod 'AWSRekognition'`
1. Open the Terminal app, and then use cd to change the directory to the
location of the unpacked files from the previous step.
1. Run the following:
`pod install --repo-update`
1. If you go to the Pods folder in your project, you should now see the pods
AWSCore and AWSRekognition there. These are the AWS Mobile SDK
components that enable you to talk to Amazon Rekognition. You can also see
that a new workspace file was created for this project.
1. Open the project workspace in Xcode and choose Build to build the project.



### Step 4: Set up credentials for the AWS Mobile SDK
Having set up the Amazon Cognito identity pool with the required role and policy to
access the Amazon Rekognition API, we now provide these credentials to the AWS
Mobile SDK in the app.
1. Open the sample app in Xcode.
1. Go to AppDelegate.swift.
1. Add the following line at the top:
`import AWSCore`
1. In the app’s didFinishLaunchingWithOptions function, add the following lines: <br>
`let credentialsProvider = AWSCognitoCredentialsProvider(
            regionType: .USEast1,
            identityPoolId: "<your-cognito-pool-id>")`
`let configuration = AWSServiceConfiguration(
            region: .USEast1,
            credentialsProvider: credentialsProvider)`
`AWSServiceManager.default().defaultServiceConfiguration = configuration`

In the steps here, we created a credentialsProvider object and assigned the
Amazon Cognito identity pool that we created. Then we created an
AWSServiceConfiguration object with these credentials. Finally, we set the default
AWS configuration for the AWS Mobile SDK in this app to the
AWSServiceConfiguration object. This allows the AWS Mobile SDK to talk to AWS for
all service resources that are included in the IAM policy that’s attached to the
identity pool.

### Step 5: Ask Amazon Rekognition to identify faces and celebrities
This section is at the core of what we’re trying to accomplish—talking to the
Amazon Rekognition API to detect faces and celebrities in images that are captured
by the app.
1. At the top of the ViewController.swift file, add the import line for
Amazon Rekognition: <br>
`import AWSRekognition`

1. Create a class variable of type AWSRekognition. We use this to talk to the
Amazon Rekognition API. <br>
`var rekognitionObject:AWSRekognition?`

1. Go to the sendImageToRekognition function, and add the following lines
after the snippet to 'Delete older labels or buttons': <br>
```
rekognitionObject = AWSRekognition.default()
let celebImageAWS = AWSRekognitionImage()
celebImageAWS?.bytes = celebImageData
let celebRequest = AWSRekognitionRecognizeCelebritiesRequest()
celebRequest?.image = celebImageAWS
```

In this code, we first initialize the rekognitionObject with a default configuration. We
then create an instance of AWSRekognitionImage and assign the bytes property,
which is the image data that’s captured by the camera or chosen from the photo
library. Next, we create an AWSRekognitionRecognizeCelebritiesRequest,
and assign the chosen image to the image property of the request.

It’s now time to send this image data to Amazon Rekognition to detect faces and
identify celebrities.

1. Add the following lines to the sendImageToRekognition function after the
lines in the previous step: <br>
```
rekognitionObject?.recognizeCelebrities(celebRequest!){
    (result, error) in
    if error != nil{
        print(error!)
        return
    }  
    if result != nil{
        print(result!)
    }
    else{
        print("No result")
    }
}
```

Here, we passed the AWSRekognitionRecognizeCelebritiesRequest from
the previous step to the Rekognition recognizeCelebrities API method.

1. Inside the if statement for `result != nil`, add the following lines to extract
celebrities and faces from the Amazon Rekognition response. Review the
code comments to get a better understanding of each task. <br>
```
//1. First we check if there are any celebrities in the response 
if ((result!.celebrityFaces?.count)! > 0){

    //2. Celebrities were found. Lets iterate through all of them 
    for (index, celebFace) in result!.celebrityFaces!.enumerated(){
        //Check the confidence value returned by the API for each celebirty identified 
        if(celebFace.matchConfidence!.intValue > 50){ //Adjust the confidence value to whatever you are comfortable with

            //We are confident this is celebrity. Lets point them out in the image using the main thread                 
            DispatchQueue.main.async {
                [weak self] in

                //Create an instance of Celebrity. This class is availabe with the starter application you downloaded
                let celebrityInImage = Celebrity()
                
                celebrityInImage.scene = (self?.CelebImageView)!
                
                //Get the coordinates for where this celebrity face is in the image and pass them to the Celebrity instance
                celebrityInImage.boundingBox = ["height":celebFace.face?.boundingBox?.height, "left":celebFace.face?.boundingBox?.left, "top":celebFace.face?.boundingBox?.top, "width":celebFace.face?.boundingBox?.width] as! [String : CGFloat]

                //Get the celebrity name and pass it along
                celebrityInImage.name = celebFace.name!
                //Get the first url returned by the API for this celebrity. This is going to be an IMDb profile link
                if (celebFace.urls!.count > 0){
                    celebrityInImage.infoLink = celebFace.urls![0]
                }
                //If there are no links direct them to a search page
                else{
                    celebrityInImage.infoLink = "https://www.imdb.com/search/name-text?bio="+celebrityInImage.name
                }
                //Update the celebrity links map that we will use next to creat buttons
                self?.infoLinksMap[index] = "https://"+celebFace.urls![0]
                
                //Create a button that will take users to the IMDb link when tapped
                let infoButton:UIButton = celebrityInImage.createInfoButton()
                infoButton.tag = index
                infoButton.addTarget(self, action: #selector(self?.handleTap), for: UIControlEvents.touchUpInside)
                self?.CelebImageView.addSubview(infoButton)
            }
        }
        
    }
}
//If there were no celebrities in the image, lets check if there were any faces (who, granted, could one day become celebrities)
else if ((result!.unrecognizedFaces?.count)! > 0){
//Faces are present. Point them out in the Image (left as an exercise for the reader)
    /*
        
    */
}
else{
//No faces were found (presumably no people were found either)
    print("No faces in this pic")
}
```

Let's quickly walk through what we did above:

* The response has three components to it - celebrities, faces, nothing found. We start by extracting celebrities.
* Photos can contain multiple faces, so we need to iterate through all of the identified celebrities.
* We use the bounding box info returned by the Rekognition to determine where each celebrity's face is in the image and place a label with the returned name in the center of the celebrity's face. We then create a button that will serve both as a label and a clickable link to the celebrity's IMDb page.  
* After we are done with celebrities, we identify other unknown faces in the image and point them out as well. 
* Note that the API also has information about face landmarks like eyes, nose, etc. and the pose (3D orientation of the face) conveyed via pitch, roll and yaw. We are not using these in this application, but feel free to explore enhancements using this information. 

### Step 6: Test your application

That's it! The hard work is done and it is now time to try out the application. Build the app to your device and confirm that the app is working and making calls to Amazon Rekognition. If you have a clickable “Jeff Bezos” label for the celebrity in the image, the app is working! Now go ahead and snap a pic of a celebrity on your tv screen, in a magazine, or in person. If everything works, you should see a label on the face(s) in the image that Rekognition as identified as a celebrity. Clicking on the identified celebrity will take you to the celebrities' IMDb profile page. Note that the application supports a camera image or an image from the phone's photo library. 







## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
