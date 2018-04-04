# CarND-Advanced-LaneLines

### Camera Calibration

Using calibration images under "camera_cal" folder (uploaded with this project) are used for Camera Calibration. 
Used opencv functions to find the corners of these chessboard images are used to find 3D and 2D object and Image points and are passed to 
cv2.calibrateCamera to get "distortion" (dst) , rotation, translation vector matices (coefficients/intrinsics and extrincis of camera) and saved to "Calibration.p" pickle file

** Please ref to "FindingLaneLines.ipynb" which has chessboard images with corners drawn using opencv

### Undistorting images

Based on above calclulated calibration values. Using cv2.undistort tried to remove distorition error on sample calibration1.jpg file.


** Please ref to "FindingLaneLines.ipynb" which has distored and undistorted image

### Collecting sample image data from Video file (provided with Ucity project folder)

Extracted video frames from "Challenege_video.mp4" and saved to "Challenge/frame*.jpg" (uploaded with this project), converted each frame from BGR to RGB. Used these images for rest of the project

** Please ref to "FindingLaneLines.ipynb" which has sample challenge video frame displayed using cv2.imshow

### undistort on video frame

Passed above collected images through "undistoro" api call, which takes calibrated matrices and sample image file to do the undistortion

** Please ref to "FindingLaneLines.ipynb" which show "Original Image" Vs "Undistorted Image"

### Perspective Transform.

Next step to get the bird eye view of the Lane section of the road. For this we need to Unwarp the image frames.

First important step is to find the Source and Destination coordinates. Source: Is the region in the original frame which the user needs to 
get a perspective transform (this region is the Lane region in which the vehcile is in). Destination: is the image coordinates in which the perspective transformed image to be appearing/drawn.

Used "getPerspectiveTransform" to get the transform matrix and also computed inverse of it by swaping src and dst

** Please ref to "FindingLaneLines.ipynb" which show "Undistorted Image" Vs "Unwarpped Image"

### Visualizing Multi colorspace Channels

Used unwarpped image example and plotted it in multiple color space like R, G, B and H, S and V.

** Please ref to "FindingLaneLines.ipynb" which show "RGB R-Channel etc" and "HSV H-Channel etc"

### Sobel Absolute Threshold (SAT)

Used Sobesl Absolute Threshold to compute the gradient of the image (To find the Lane Lines). Converted input image to gray scale and applied Using cv2.Sobesl on the gray image and normalized (scale it to 8-bit - 0-255).

Tried different threshold value range for min and max threshold. Initialized a binary duplicate image with all zeros.
Compared scaled sobel values with min and max threshold values. Setting scaled values to 1 at positions where scaled soberl value falls  within the min and max threshold (Which extracts the lane lines)

** Please ref to "FindingLaneLines.ipynb" which show "Unwarpped image" and "Sobel Absolute" output.

### Sobel Magnitude Threshold (SMT)

Applying Sobel w.r.t Oorientation of the image plan "x" and "y", computing magnitude and creating binary image as followed in "SAT"
Tunning "Kernel Size", min and max threshold and passing it to mag_thresh function call to compute SMT 

** Please ref to "FindingLaneLines.ipynb" which show "Unwarpped image" and "Sobel Magnitude" output.

### Sobel Direction Threshold  - and combining Magnitude and Threshold outputs

Applying Sobel w.r.t Oorientation of the image plan "x" and "y", computing direction using arctan2 and creating binary image as followed in "SAT" Tunning "Kernel Size", min and max threshold and passing it to mag_thresh function call to compute SMT 

** Please ref to "FindingLaneLines.ipynb" which show "Unwarpped image" and "Sobel Magnitude + Direction" output.

### Pipeline

Desinged a pipeline to undistort and unwarp the input image/video frame. Used few color space methods - which provied to be better than other color spaces based on experimentation to compute binary output (based on fine tuning the min and max threshold values)

### Running piple on sample frame image that are extracted from Challenge_video.mp4

Passing each frame/image under "./Challenge/frame1*.jpg" to the pipeline and computing perspective transform matrix 
and displaying few sample output 

** Please ref to section "Pipeline" in "FindingLaneLines.ipynb" .

### Sliding Window Poly Fit

Method to fit polynomial to binary image with lines extracted, using sliding window. 
1. Compute histogram
2. Find Left and right peak of the Histogram
3. Selecting specific ROI in the histogram for left and right halfs
4. Slection sliding window number
5. Indentify non-zero pixels 
6. Compute left and right lane indices based on non-zero pixel positions in x and y
7. Use polyfit to draw polygon based on the coordniates computed

Passing sameple image file through pipeline and sliding window poly fit

** Please ref to section "Sliding window poly fit" in "FindingLaneLines.ipynb" .

Polyfir using from Previous frame
Used same steps, but extracting previously computed indices to predict next frame indices values

### Radius of curvature and Distance from lane center

The radius of curvature is based upon http://www.intmath.com/applications-differentiation/8-radius-curvature.php and calculated in the code.

curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:
```
Lane_center_position = (r_fit_x_int + l_fit_x_int) /2

center_dist = (car_position - lane_center_position) * x_meters_per_pix
```
####  Draw Lanes

Using cv2.fillpoly and coordinates compute from "Polyfit_using_prev_fit" is used to fill the poly (lane line area)

Finally have a Class called "Line", which gets the characteristics of each lane line.

#### Running the pipleline on each image frame of Challenge_video and "project_video"

Passing each video frame thorugh process_image call
1. Passes frame through pipeline
2. If lines detected, Perspective transformed image is passed through sliding window polyfit
3. Selecting ROI to invalidate other lines (which are not lanes)
4. Calculating curvatuew and distance from center
5. Fill the lane region using fillpoly
6. Processed video is svaed as "Challenge_video_out.mp4" and "project_video_out.mp4"
