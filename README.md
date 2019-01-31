## Amazon Rekognition Celebrity Detection Ios

iOS Swift project code for identifying celebrities using Amazon Rekognition

Check out: [AWS Mobile Blog - Using Amazon Rekognition to detect celebrities in an iOS app](https://aws.amazon.com/blogs/mobile/amazon-rekognition-detects-celebrities-in-ios-app/).

## Steps to use

1. Create a Cognito Identity Pool Id, which you will use to authenticate the app against the Amazon Rekognition APIs
2. Now you need to attach an IAM role to this Identitiy with the appropriate IAM policy. Choose the already available policy template *AmazonRekognitionReadOnly*, attach it to either unauth or auth IAM roles and attach the role to the Cognito Identity you created in step 1. 
3. Enter the Cognito Identity Pool Id in the *AppDelegate.swift* file under the *Initialize Identity Provider* section. 
4. Run `pod install --repo-update` to get the required Pods. Your Podfile already has the dependencies listed.
5. Build the app to your device and confirm that the app is working and making calls to
Amazon Rekognition. If you have a clickable “Jeff Bezos” label for the celebrity in the
image, the app is working! Now go ahead and snap a pic of a celebrity on your TV
screen, in a magazine, or in person. If everything works, you should see a label on
the face in the image that Amazon Rekognition has identified as a celebrity. Clicking
on the identified celebrity takes you to the celebrity’s IMDb profile page. Note that
the app supports a camera image or an image from the phone's photo library.

For more details on how the app works review the [Starter App tutorial](https://github.com/aws-samples/amazon-rekognition-celebrity-detection-ios/blob/starter-app/README.md).

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
