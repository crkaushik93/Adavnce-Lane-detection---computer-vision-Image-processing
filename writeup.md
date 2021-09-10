### Image reference

[notebook]: ./python_notebook.ipynb
[image1]: ./example_images/image1.PNG "image-1"
[image2]: ./example_images/image2.PNG "image-2"
[image3]: ./example_images/image3.PNG "image-3"
[image4]: ./example_images/image4.PNG "image-4"
[image5]: ./example_images/image5.PNG "image-5"
[image6]: ./example_images/image6.PNG "image-6"
[image7]: ./example_images/image7.PNG "image-7"
[image8]: ./example_images/image8.PNG "image-8"
[image9]: ./example_images/image9.PNG "image-9"
[image10]: ./example_images/image10.PNG "image-10"
[image11]: ./example_images/image11.PNG "image-11"
[image12]: ./example_images/image12.PNG "image-12"
[video1]: ./output_videos/project_video_process.mp4 "Video"

### Camera Calibration

#### Calibration image.

The images provided are subject to some sort of distortion, due to many factors discussed earlier. So images captired bu one camera tend to different from the images captured by others. Thus the project requires fitting of these images especially to determine the lane lines.So using camera caliberation we are creating alens distortion. 

Camera calibration is  processed using the images of checkerboard patterns. The dimensions are known especially the 4 corners of the square board that are maent to be equidistant from each other. By taking all the images of different angles provided we tend to calibrate  and define the number of corners. Using this we can calculate the focal length point offset parameters that make the camera intrinsc matrix and distortion coefiicients called the intrinsic and extrinsic parameters. The code used to find this are shown in the python notebook.
![alt_text][notebook]


We initialized an matrix, that contains the 3D (X, Y, Z) coordinates corners for each image caliberated. Even though the pts are #d we can assume the coordinates of chessboard fixed on only x,y plane and while z coordinate is 0. Thus object pts are considered as an array having the corner coordinates whose copy is available in obt_pts. Also we stored the detected corners of values in x, y coordinates in list img_pts.

Now that we pass these initialised coordinates into the 'cv2.calibrateCamera()' function. I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![alt_text][image1]

### Prespective Image:

Using the function cv2.undistort() we  make a perspective transformation. we achieved this by first defining two quadrangles. Then we calculated the  source of projection quadangle (src)  to the destination quadrangle (dst) through passing the quadrangles to cv2.getPerspectiveTransform(). the resultant is matrix.

The code is found in 5th cell  notebook, and the result image can be seen  can be seen here:
![alt_text][image2]

### Image to Birds-eye View

The first of the finding lanes pipeline  is to transform whole image into a  birds-eye view. This is done  by src and dst quadrangles, matching very close to  the 3D coordinates of plane of the image and having a projection of the rectangle of the image parallel to the   plane of the camera. The code is shown in  get_quad_angles() method of the lanedetector_get class.

Next, we undistort the image, using  camera matrix and distortion coefficients, and apply a perspective transform using the dst and src quadrangles.
![alt_text][image3]
![alt_text][image4]


### Gradients and color filters:

To find the lane lines in the transformed images, we transform the image to the HSV color space, and  perform histogram equalization On transformation we define  yellow and white color ranges to mask out image not in those ranges. To tease out more challenging lane images, I we used two Sobel filters . Because of these filters, lines  more vertical and  highly contrast with the dark gray, tend to be found by the Sobel filter. We applied this Sobel operation to both the S and V channels of the HSV image.The resultant image is shown below:
![alt_text][image5]

### Finding lanes and fitting curves:

The binary images resulted from above filtering steps are 2-dimensional arrays composed of 1s and 0s. Out of the available results, we get higher value of columns having 1s. Thus we take index of the maximum value from left half and from right half thus assuming that we have good results. 
![alt_text][image6]
![alt_text][image7]

### Sliding Window
Using the initial X values we create a search window of specific dimensions. We try to  save the coordinates of every non-zero pixel within the bounds of this window and compute the mean X value of all these pixels. We  set that X value as the center of the next search window, which uses the top of the current window as its base and height half one and half times of the base. By proceeding this way we try to reach the top of the image, thus collecting pixels that make up X,Y coordinates matching the lane lines in the image.

The code for this procedure can all be found in the window_search_sliding() method of the lanedetector_get() class.
![alt_text][image8]
![alt_text][image9]

### Lookahead Search
The work progress has been on finding lane lines across video images. But there are minor variation in the curves from one frame to the next. This meant to search for a margin in the left and right of each curve rather than starting from scratch of finding serach window. This locates pixel of next curve with in that margin.T

The code for this lookahead search procedure can be seen in search_ahead() method of the lanedetector_get() class and is visualized in the image below.

![alt_text][image10]
![alt_text][image11]

### Implementation on Original video

Now we have found our curves, executed all the required parameters, defined the required functions and the  the remaining steps of the image processing pipeline are simple.So all is set to check whether the algorithm works perfectly is by initaiting on the video provided.The result obtained are shown below:

[video1]: ./project_video_process.mp4 "Video"

### Troubleshooting:

We created a diagnostic screen to stack the different stages of pipeline such that we can understand and troubleshoot the lanes thatr didn't go as expected. the code to have such a view is found in the following code:

![alt_text][image12]

## Future Improvements

By using two fixed quadrangles for the perspective transform is found to be not that could idea because the ground changes every time when the motion of vehicle changes. Due to these changes such as car moving overa a bumper or moving down along the hill, the bird eye image fluctuates and bec of this lane tend to loose its parallelism. This raises a qusetion  whether the obtained lane results are valid or not. Thus from my understanding I believed to have a better dynamic ground estimation methods and see that we project a better destination quadrangle on to the ground. The time required for such an approach is quite long and in this project was not determined and held it up as future improvements.


