[image1]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/chessboard.png "Chessboard"
[image3]: ./output_images/chessboard_warped.png "Chessboard Warped"
[image4]: ./output_images/original.png "Original"
[image5]: ./output_images/test_warped.png "test_warped"
[image6]: ./output_images/hist.png "histogram"
[image7]: ./output_images/masked_curve.png "masked_curve"
[image8]: ./output_images/masked_straight.png "masked_straight"
[image9]: ./output_images/curve_fitting_fill.png "Curve fitting"

[video1]: ./videos/output.mp4 "Video"


## Camera Calibration

Implementation can be found in _camera_calibration_ jupyter notebook. 

Chessboard undistortion visualization is located in _camera_calibartion_ cell 3 and _advanced_lane_line_ notebook cell 3. 

Camera calibration, geometric image transformation matrix M, cell 2.

![Chessboard][image2]

_cv2.undistort_ in cell 6, is used to undistort image based chessboard calibration. 

![Undistortion][image1]

Differences can be seen around the edge of images. 

![Undistorted and warped][image3]

Illustrating using generated warped transform matrix to peform warp perspective transform. Same tranformation matrix will be used to generate bird eye view.

## Bird eye view

Please see _warped_ function in cell 7 of advanced lane line for _source (base\_points)_ and _destination (dst\_bas\_points)_ used in warp transformation.

Below strategy is used for programmatically extract parallel points in sample images

```python
  base_points = [[w // 2 - 76, h * .625], [w // 2 + 76, h * .625], [-100, h], [w + 100, h]]
  dst_base_points = [[100, 0], [w - 100, 0], [100, h], [w - 100, h]]
```

where _h_ and _w_ is height and width of sample images


Applying warp transform with above parallel points set up on below test images

![Test images][image4]

We obtain

![Warped_test][image5]


Other ideas worth exploring includes ensemble method, adding an independent layer tuned for challenging frames such as images under the shadow or with background rapidlly changing, then superimpose
this layer onto universal layer.

## Edges Masked image  

 * Gradient Thresholds using _abs sobel mask_ and _mag_sobel_mask

 * HLS(Color Space) Mask 

 are used to obtain high contrast lane line features. An illustration of histogram feature is shown below 

![histogram][image6]

 Obtained masked images

![masked_curve][image7]

![masked_straight][image8]

## Second order curve fitting

Cell 20 in _advanced\_lane\_line  notebook

![Curve fitting][image9]

Cell 27 using curvature measuring formula in previous cell

Road radius estimation _eval\_curve_ in Cell 28. In generated video, estimated curvature is around 600m to 2000m. 

## Pipeline and interframe filtering

Completed pipeline is in cell 30 with interframe smoothing function _process\_curr\_frame_ for taking navie average over last _MAX\_Q_\_LEN_ number of frames.

This approach significantly improved stability of curve prediction especially for challenging frames such as images under tree shadow. Predicatable downside for this low pass filter 
approach is system incresing system reaction time. Considering vehicle movement is continuously, this downside is acceptable for this usecase.

## Video

Marked video

![Output video][video1]
