Hardware Setup
==============

Tracking box
------------

Many of the challenges in achieving robust tracking over long time
scales that would be very difficult to solve by adjusting parameters in
the tracking can be entirely avoided by constructing a dedicated space
for tracking. A tracking box has the combined benefit of isolating both
the camera and the tracked individuals from external optical and
mechanical perturbations. Although a tracking box is not strictly
required, a box of the general configuration in the sample schematic
below may serve as a reference for good tracking conditions before
attempting to optimize other hardware or software parameters (fig 2.1).
The only essential features of a tracking box are opaque walls, a camera
mount, and a diffuse illumination source. The difference image
calculation used by MARGO is very sensitive to any changes within the
field of view of the camera. This is good because it means that the
software is sensitive to very small movements of small objects, but it
also means that the only movement visible to the camera should be the
tracked objects. MARGO can automatically detect and adjust for spikes of
noise in the image, but walls will drastically reduce the time MARGO
spends attempting to correct noisy imaging and will increase the time
spent tracking.

![*figure 2.1* - Sample tracking platform constructed from aluminum
rails and acrylic plastic. Backlit behavioral arenas and opaque walls
enhance ROI contrast and reduce imaging
noise.](images/Hardware Setup/Tracking box/behavioral_box_isometric_painted_labels.pdf){width="50%"}

In addition to walls, use clear sanded plastic, diffuser film, or paper
either between the illumination source and behavioral arenas or on the
floor of the arenas themselves to diffuse the light source. Having a
sanded or matte finish on all surfaces inside of the tracking box can
also help reduce reflections and glares inside the box.

::: {#arenasection}
Individual behavioral arenas
----------------------------
:::

MARGO ROI setting can operate in two different modes: *automatic* and
*grid* modes. Choosing appropriate individual behavioral arenas will be
dependent on the ROI detection mode used. Automatic detection mode works
by finding bright regions of the image that are all roughly the same
size. This means that ideal behavioral arenas used for automatic ROI
detection are backlit, transparent areas separated by opaque areas in
between. Sample arena construction for automatic detection is shown
below in fig. 5.

![*figure 2.2.1* - Sample images of automatic ROI
detection.](images/Hardware Setup/Behavioral Arenas/autoROI_detection.pdf){width="95%"}

Grid detection mode works only on the assumption that ROIs will be
arranged in regularly spaced rows and columns. The user is prompted to
draw and adjust the position of one or more grids of any arbitrary
dimensions to specify ROIs. This means that behavioral arenas can be
anything with stereotyped dimensions. Wellplates of any dimensions make
ideal behavioral arenas for grid mode detection. Grid mode also works
well with linear arrays of tunnels. Sample arena constructions for grid
detection mode are shown below in fig. 6.

![*figure 2.2.2* - Sample tracking arena constructed from layered, laser
cut acrylic. Semi-transparent floors and opaque walls create
high-contrast, easily detectable
ROIs.](images/Hardware Setup/Behavioral Arenas/circular_arenas.pdf){width="50%"}

::: {#illumsection}
Illumination
------------
:::

Quality illumination is a prerequisite for high-quality tracking. An
ideal light source will be evenly diffuse and sufficiently bright.
Because MARGO uses a single tracking threshold for the entire field of
view of the camera, even illumination is the top priority for an
illumination source. MARGO can correct for uneven brightness, but having
an even illumination source will ensure that the quality of tracking is
equally good in all parts of the cameras field of view. Any illumination
source bright enough to saturate the camera is sufficiently bright for
tracking.

We perform all of our tracking with infrared light. Using an infrared
light source and longpass filter on the camera has the dual benefit of
reducing sensitivity of the tracking to perturbation by most external
light sources (not sunlight) and allowing for independent control of
visible light sources to deliver stimuli not visible to the camera. An
affordable example of two color channel LED light panels configured with
white and infrared LEDs can be found
[here](https://www.knema.com/led-modules.html). Alternatively, LCD
backlight panels can provide a ready-made solution to bright, even
illumination.

::: {#camconfig}
Camera configuration
--------------------
:::

MARGO has built-in tools to automatically detect and configure any
camera visible to MATLAB. The appropriate configuration for your camera
will largely depend on your particular experiment. Camera customization
features are accessible under the \"Hardware\" menu bar. But before
configuring the camera in MARGO, ensure that the camera and camera lens
are properly setup for tracking. The following are good guidelines for
configuring your camera:

-   Aperture opened enough that ROIs are nearly saturation

-   Lens maximally zoomed to reduce lens fisheye

-   Lens focused on ROIs

-   Camera rigidly fixed to a camera mount

::: {#camdetect}
### camera detection
:::

Camera support in MARGO is built on MATLAB's Image Acquisition Toolbox.
Before a camera can be detected by MARGO, the associated the associated
MATLAB camera adaptors must first be installed. The appropriate adaptor
to install will depend on the camera manufucturer. See MATLAB's tutorial
for complete instructions on [installing MATLAB camera support
packages](https://www.mathworks.com/help/imaq/installing-the-support-packages-for-image-acquisition-toolbox-adaptors.html).

Once the camera adaptors are installed, a list of available cameras and
camera modes should auto-populate upon launching MARGO. The list
available cameras can be refreshed under the hardware menubar.

::: {#cammodes}
### camera modes
:::

MARGO will automatically detect available camera modes for operating at
different pixel resolutions and color bit depths. The tracking is
designed to work with 8-bit color depth because most tracking operations
are performed on binary images. It is also strongly recommended that
monochromatic cameras are used. By default, MARGO only tracks the green
color channel of RGB images.

::: {#camsettings}
### camera settings
:::

Camera properties such as exposure time, shutter speed, gain, and frame
rate can be adjusted through *camera settings* under the *Hardware* menu
bar. Because the configurable settings are specific to each device, the
properties that can be adjusted here will depend on your camera. Many
cameras have modes to automatically adjust the camera exposure time,
shutter speed, frame rate, gain or focus in real time. These modes are
also often enabled by default, which may be good for many applications
but is highly problematic for tracking. Constantly fluctuating images
will introduce a lot of noise into the tracking due to the sensitivity
of difference image to even minor changes from frame to frames.

To adjust camera settings, first initiate *preview camera* and leave it
running to get feedback on the settings as they change. Select
***Hardware* $>$ *Camera* $>$ *camera settings*** to open the camera
settings window. Keep in mind that the available settings and names of
each property will be dependent on the camera. From the settings menu:

-   Ensure that any automatically adjust fields are set to \"off\" or
    \"manual\"

-   Adjust exposure and shutter speeds until the preview is just below
    saturation

-   Set max frame rate (target acquisition rate can be set lower in
    tracking parameters)

-   Close window to save settings