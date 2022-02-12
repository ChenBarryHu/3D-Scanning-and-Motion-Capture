# 3D-Scanning-and-Motion-Capture
Mainly for Structure from Motion (SfM) and Multiview Stereo (MVS).

## Debug Edition
Debug edition contains some original test code,containing the full workflow of making the wheel for SfM calculation.  
However, after comparison, we finally choose OpenCV lib as our tool to finish the release edition.  
Even in this kind of situation, I still consider this part would have the educational meaning.  
 1. Fundamential matrix calculation  
 2. Essential matrix calculation  
 3. RANSAC  

## Release Edition
Remember to config the CMakeLists right, as we have not used the regular way to config it.  
However, we should clarify our workflow and some tricky components in our code. As to workflow:  
  

1. Find the right sequence of the image pairs, as the DTU dataset only provides *PAIR* information instead of *STEREO-SEQUENCE* information. So, for each scan, we could only use a `while-loop` to search the sequence, which makes us have no ability to give a universal answer for how many images or pairs we use during the bundle adjustment. However we can ensure that more than 10 images participating the BA workflow. The related code is [here](https://github.com/hinczhang/3D-Scanning-and-Motion-Capture/blob/be94566c63d8db5dcffb68a6462646c31f565806/releaseEdition/MVStereo/mv_stereo.cpp#L45);  
  
2. Initialization. We try to initialize the first two images first to obtain 3D points, R, T and corresponding index structures that could connect the keypoints with 3D points. Like [here](https://github.com/hinczhang/3D-Scanning-and-Motion-Capture/blob/be94566c63d8db5dcffb68a6462646c31f565806/releaseEdition/SfM/Stereo.cpp#L115) in the `RTresponse` function;  
  
3. For the `RTresponse` function, the common step for initialization and further fusion is to calculate the keypoints and find the match, in the following [code](https://github.com/hinczhang/3D-Scanning-and-Motion-Capture/blob/be94566c63d8db5dcffb68a6462646c31f565806/releaseEdition/SfM/Stereo.cpp#L143). After initialization, the new image would compared with the original right image. From the function `get_objpoints_and_imgpoints` ([here](https://github.com/hinczhang/3D-Scanning-and-Motion-Capture/blob/be94566c63d8db5dcffb68a6462646c31f565806/releaseEdition/SfM/Stereo.cpp#L203)), we can obtain 3D points generated by the previous stereo pair, along with image 2D points from keypoints in the current right image (which means that we try to choose the matches, if they contain the left keypoints that are same as the right keypoints from the previous matches) (**Tricky!**);  
  
4. We than use the `PnP` algorithm to get R and T of the current right image, as we have the keypoints and the corresponding 3D points of the right image. Then we try to use the `fusion_structure` function (see [here](https://github.com/hinczhang/3D-Scanning-and-Motion-Capture/blob/be94566c63d8db5dcffb68a6462646c31f565806/releaseEdition/SfM/Stereo.cpp#L220)) to integrate 3D points and, more important, the index stucture, which could be used to inquiry the relationship between 3D points and 2D keypoints;  
  
> 5. [Bundle adjustment](https://github.com/hinczhang/3D-Scanning-and-Motion-Capture/blob/be94566c63d8db5dcffb68a6462646c31f565806/releaseEdition/MVStereo/mv_stereo.cpp#L95). We try to load 3D points, keypoints, the intrinsic matrix along with extrinsic matrix to calculate the loss.
  

## Dataset
DTU dataset: https://roboimagedata.compute.dtu.dk/  
Middlebury: https://vision.middlebury.edu/stereo/data/