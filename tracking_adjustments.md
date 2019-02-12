Advanced Tracking Features
==========================

MARGO understands that the setup and goals of tracking can vary greatly
from instance to instance. For that reason, many tools have been
included to offer flexibility and customization in how ROIs and
centroids are tracked, sorted, and measured.

Tracking parameters
-------------------

Many of tracking parameters are used to track, identify, or sort ROIs
and tracked objects. These parameters can be adjusted under *Tracking*
$>$ *tracking parameters*.

-   **speed threshold** - maximum allowed distance traveled per second.
    Each update in an object's position is time-stamped. Every frame,
    MARGO starts with unassigned blobs in a difference image. To
    assemble blob positions into traces over time, tracked objects are
    assigned to an ROI. The speed threshold excludes centroids in any
    given frame that have moved too far from the last recorded centroid
    for the paired ROI. This parameter helps prevent the centroid from
    jumping to pixel. *Note:* toggling the speed threshold radio button
    after ROI detection and referencing will display a measured rolling
    average speed and standard deviation for each ROI to help with
    parameter setting.

-   **distance threshold** - maximum allowed distance to the center of
    an ROI. This parameter is used to identify which ROI to assign blob
    centroids. This parameter is only used when *distance* centroid sort
    mode is selected. *Note:* toggling the distance threshold radio
    button will display a circle showing the range of inclusion for each
    ROI.

-   **area thresholds** - these parameters set lower and upper bounds
    that blobs in the threshold image will be tracked. Objects in the
    image outside of this range are excluded from tracking in the
    current frame. *Note:* toggling the area threshold radio button
    after ROI detection and referencing will display a measured rolling
    average area and standard deviation for each ROI to help with
    parameter setting. This will also display concentric circles of area
    equal to the specified bounds.

-   **vignette gaussian weight** - sets the multiplicative weight of a
    gaussian mask applied to the entire image during ROI detection.
    Detecting all ROIs in an image of varying luminance can be
    challenging. Once MARGO knows which parts of the image are
    foreground and which are background, a vignette filter can easily be
    calculated. Before foreground and background are distinguished, some
    assumptions can help smooth the luminance profile of the image. Many
    camera lenses and luminance sources create images that are slightly
    brighter in the center and dimmer at the edges. By default, MARGO
    assumes a gaussian profile of global luminance in the image and
    applies a filter to the image to smooth the global luminance. A
    lower weight will reduce the smoothing applied to the image. *Note:*
    this parameter is only used during automatic ROI detection and does
    not apply to *grid mode* detection or object tracking.

-   **vignette gaussian sigma** - sets the standard deviation of the
    above vignette correction gaussian as a fraction of the image
    height. *Note:* this parameter is only used during automatic ROI
    detection and does not apply to *grid mode* detection or object
    tracking.

-   **target acquisition rate** - sets the maximum acquisition rate of
    tracking in hertz. Adjusting this parameter only ensures that
    tracking will not exceed this rate. The maximum achievable rate will
    depend on computer hardware, image resolution, number of objects
    tracked and centroid sorting mode. For more information on
    optimizing acquisition rate, see [improving tracking
    performance](#trackingperformance).

-   **ROI clustering tolerance** - sets the maximum number of standard
    deviations in vertical distance for adjacent ROIs to be assigned to
    the same ROI. To automatically assign numbers to ROIs when using
    *auto* ROI detection mode, MARGO first sorts ROIs by their vertical
    center of mass and measures the vertical distance between adjacent
    ROIs. Any pair of ROIs that are within the maximum number of
    standard deviations of one another are said to belong to the same
    row. Decreasing this value will make vertical clustering of ROIs
    more stringent. Increasing the value will cluster ROIs that are more
    vertically separated into the same row of ROIs. *Note:* this
    parameter is only used during automatic ROI detection and does not
    apply to *grid mode* detection or object tracking.

-   **dilation size** - sets the number of pixels to dilate and erode
    the threshold image. Dilation and erosion helps fill in gaps between
    above threshold pixels that, ideally, should belong to the same
    blob. This is particular helpful in under-illuminated images where
    tracked individuals are split into multiple nearby blobs. Increasing
    this value will result in slower tracking speed but will help stitch
    nearby pixel islands into the same blob. *Note:* set this value to
    zero to disable dilation and erosion.

-   **sort mode** - sets mode by which centroids are assigned to ROIs.
    *Distance* and *bounds* sorting modes requires centroids to be
    within the specified distance of the center or fall within the
    bounds of an ROI, respectively. Centroids not meeting the
    requirement are not eligible for assignment to that ROI. *Distance*
    mode is most useful with round shaped ROIs. *Bounds* mode is most
    useful with elongated ROIs that have an asymmetric width and height.
    *Note:* bounds sorting mode is automatically employed when grid ROI
    detection mode is used.

-   **ROI detection mode** - sets the mode by which ROIs in the camera's
    field of view are defined. See [ROI detection](#roidetect) for more
    details on these tracking modes.

Converting pixel units to mm
----------------------------

By default MARGO measures distances in pixel units. A tool is included
to allow calculating a pixel/millimeter conversion factor by drawing a
line over a known distance in the image. To begin, measure the length in
mm of any object that will go into the camera's field of view (eg.
length of a 96 well plate). Place the object in under the camera and
select *Tracking* $>$ *distance scale* to open the conversion tool.
Enter the length of the target object in mm in the space provided and
select *draw line*. Click and drag in the camera preview window to drag
and drop a line along the length of the target object to automatically
calculate and store a pixel/mm conversion factor. If necessary,
reposition the line end points and select *update* to calculate a new
conversion factor. Close the window to save and exit. For accurate
measurement, ensure the following:

-   behavioral platform is perpendicular to the camera

-   conversion factor is reacquired if the camera adjusted

-   camera is calibrated to correct for fisheye lens distortion

Camera calibration
------------------

Camera parameter objects output by MATLAB's [camera calibrator
app](https://www.mathworks.com/help/vision/ug/single-camera-calibrator-app.html)
can be used to correct fisheye lens distortion in images. To use camera
calibration in MARGO, follow the instructions in MATLAB's tutorial to
calculate and export camera parameters to correct images. Save the
exported object *MARGO/hardware/camera\_calibration/* in a .mat file.
Create the *camera\_calibration* directory if necessary. MARGO will
automatically look for camera parameter objects in the specified
directory upon initialization. Once the camera parameter object is
exported and saved, toggle calibration under *Hardware* $>$ *camera* $>$
*use calibration*.

Vignette correction
-------------------

Variation in baseline luminance across images can make it difficult to
cleanly separate out tracked objects and ROIs with a single threshold
value. MARGO uses vignette correction to increase the dynamic range over
which objects can be separated from the background. To achieve this, the
software employs two different strategies to smooth the baseline
illumination of all ROIs in the image.

-   **Gaussian correction** is utilized in *auto ROI detection* mode
    prior to initial detection of ROIs. Uneven illumination occurs most
    commonly as vignetting (ie. light fall-off) at the edges of image
    (fig 5.4). MARGO uses the assumption of a gaussian profile of
    vignetting centered on the image to smooth out the illumination. The
    width and weighting of the gaussian can be adjusted under *tracking
    parameters*. Figure 5.4 illustrates how correcting vignetting can
    allow all ROIs in the image to be cleanly separated from the
    background.

    ![*figure 5.4* - vignette correction can improve automatic ROI
    detection](images/vignette_correction/vignette_correction.pdf){width="50%"}

-   **ROI-based correction** can only be employed after ROI detection
    because it relies on using ROI definitions to pick out dim ROIs in
    the image. Using this information, MARGO generates a subtraction
    matrix from the reference image. This matrix reduces all bright
    areas of image to the brightness of the dim ROI. This has the
    advantage of not assuming any particular shape to the variation in
    luminance across the image and often results in more even
    illumination than gaussian correction. A sample dim ROI can be
    picked manually by selecting ***Tracking* $>$ *vignetting***. Click
    and drag in the camera preview window to select a region of the
    image at the target brightness.

Video recording
---------------

Raw video data can be streamed to disk simultaneous with tracking.
Uncompressed video files are useful for software packages that train
behavioral classifiers on the raw pixel data such as the [Janelia
Automatic Animal Behavior Annotator
(JAABA)](http://jaaba.sourceforge.net/). Videos can additionally be used
for manual annotation behaviors or fed back into MARGO or other tracking
programs for independent validation of traces.

Toggle video recording by selecting ***Tracking* $>$ *video* $>$
*record***. By default, MARGO saves uncompressed video data. Because raw
video files can be very large, compression can be toggled by selecting
*Tracking* $>$ *video* $>$ *compress*. It is strongly recommended that
uncompressed videos are saved for any application using the raw pixel
data as compression will result in the loss of information.

::: {#trackingperformance}
Tracking performance
--------------------
:::

Low frame rates are often sufficient to capture behavior and can help
reduce files sizes and data processing. But closed-loop stimulus
delivery may demand acquisition rates that far exceed these rates.
MARGO's real-time tracking excels at applications requiring tight
closed-loop feedback. By optimizing your setup, MARGO can easily achieve
acquisition rates greater than 60Hz. Each of the following can limit
tracking performance:

-   background pixel noise

-   *grid* ROI detection mode

-   camera frame rate

-   low target rate set under ***Tracking* $>$ *tracking parameters***

-   unoptimized minimum and maximum area thresholds

-   display update mode (disabled by selecting ***Display* $>$ *none***)

-   blob dilation/erosion (disabled by setting dilation size = 0)