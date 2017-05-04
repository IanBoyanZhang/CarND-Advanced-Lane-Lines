[image1]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/chessboard.png "Chessboard"
[image3]: ./output_images/chessboard_warped.png "Chessboard Warped"
[image4]: ./output_images/original.png "Original"
[image5]: ./output_images/test_warped.png "test_warped"
[image6]: ./output_images/hist.png "histogram"
[image7]: ./output_images/masked_curve.png "masked_curve"
[image8]: ./output_images/masked_straight.png "masked_straight"
[image9]: ./output_images/curve_fitting_fill.png "Curve fitting"
[image10] ./output_images/centroid.png "Centroid"
[image11] ./output_images/color-fit-lines.jpg "fit"

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

The centroid method may fail to find proper lane line when not enough pixels shown in certain windows. 

![centroid][image10]

Hence we use histogram method to highlight lane lines. As shown in classroom, second order polynomial is used to fit lane line position

![Fitting][image11]

## Second order curve fitting

Cell 20 in _advanced\_lane\_line  notebook

![Curve fitting][image9]

Cell 27 using curvature measuring formula in previous cell

Road radius estimation _eval\_curve_ in Cell 28. In generated video, estimated curvature is around 600m to 2000m. 

Two slightly different curve fitting methods are used to estimated curvatures.

Which can be found in cell 28 and cell 22. In final implementation, we choose using _eval\_curve_ function defined in cell 28. 

According to side channel information, estimated curvature generated from this function is closer to ground truth than counterparts generated from _eval\_curve\_by\_pixles_, since it leverage 
second order polynomial fitting described in cell 28 mark down. Both approaches can be used to evaluate vehicle center offset under same cooridnation. Since same meters per pixel in image horizontal dimension.

Please see _detect\_lane\_line__ (in cell 30) function _draw\_text_ for more details.

## Discussion

***Pipeline and interframe filtering***

Completed pipeline is in cell 30 with interframe smoothing function _process\_curr\_frame_ for taking navie average over last _MAX\_Q_\_LEN_ number of frames.

This approach significantly improved stability of curve prediction especially for challenging frames such as images under tree shadow. Predicatable downside for this low pass filter 
approach is system incresing system reaction time. Considering vehicle movement is continuously, downside is acceptable for this usecase.

In final pipeline implementation the parameter is set as 37, as it hits a good balance between reaction speed and interframe smoothness. This method may have issues reacting rapidly enough or robustly enough when
  
  * Vehicle motion vector changes abruptly in short amount of time 
  
  * Continously and rapidlly  lighting condition changes.  

  * Rapidly curve changing likely encountered in wandering roads 

Another idea worths experimenting is using ensemble approach by adding independent detection layers tuned for challenging light condition or low contrast frames. Then superimpose this layer onto existed detection layers. 

More complicated approaches can be constructed by assigning decision weight on look ahead/back filter to reduce effect imposed by more outdated frame information; dynamically tuning look back memory queue to drop frame information
acquired under poor detection conditions.

Perspective transform serves naturally as a crop filter. By limiting ROI of input image, it significantly reduces the background noise in image.

### Color space and thresholding

Initially, when working with test images, unde RGB space, the _red_ and _green_ channel seems working well in identifying yellow and white lane lines. An ensemble method (logic AND) is used
in combining the two channels.

However, it is challenging for RGB channel color masking to perform well under poor lighting conditions, such as under tree shadow. 

We find HLS (Hue, Lightness, Saturation) has good performace in picking out
lane line features under different light conditions, since lightness is extracted as independent feature channel. Combing color thresholding, saturation channel masking is good at picking out artificial marks such as lane lines.

### Future work

If time allowed, we would rafactor notebook code to more modulized, small python functions scripts to help perform further experiments.

## Video

Marked video

![Output video][video1]

[Alternative link](https://youtu.be/OQienj9xbQI)
