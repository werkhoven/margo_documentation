Starting a New Experiment
=========================

Before running any experiment, MARGO must go through a simple setup that
follows the workflow shown in fig. 1. MARGO's user interface enforces
this workflow by progressively enabling downstream controls as each
sequential step is completed. To access the next step in the workflow,
the user must complete the preceding step. Keep in mind that [loading a
custom profile](#saveprofile) will allow many of these steps to be
skipped and substantially reduce the time needed to setup a new
experiment.

Confirm camera and camera mode
------------------------------

The only control panels enabled by default are camera and illumination
(not covered here). The user must minimally select a camera, image
resolution, and data output format from the drop down menus. Once
selected, *confirm settings* to initialize the camera. At this point, it
is also a good to preview the camera and adjust any other camera
settings if necessary. Because the tracking and ROI thresholds selected
in the downstream steps can be sensitive to changes in exposure, shutter
speed, and gain, it is strongly recommended that these settings are
[manually configured](#camsettings) before continuing.

::: {#roidetect}
ROI detection
-------------
:::

A region of interest (ROI), is a static location in the camera's field
of view that tells MARGO where to expect tracked objects. All movement
or changes in the image outside of the ROIs will be untracked. The
software is forgiving of setting ROIs in regions without a tracked
objects since ROIs with little or no movement by filtering traces in
those ROIs, but is unforgiving of failing to set an ROI over a tracking
target. It is therefore important that ROIs are set properly before the
tracking begins. ROI detection in MARGO comes in two flavors: *auto* and
*grid* detection modes. As the name suggests, *auto* mode is essentially
instantaneous and easy, but requires your ROIs and imaging setup to meet
particular conditions. This mode is enabled by default. On the other
hand, *grid* mode tolerates greater variability in ROI and imaging
conditions but requires the user to draw and position one or more ROI
grids over the image. ROI detection modes can be switched under
***Tracking* $>$ *tracking parameters* $>$ *ROI detection***.

**auto mode**

Automatic ROI detection works by finding a threshold value that
separates bright regions of the image from a dark background. Before
applying the threshold, MARGO first attempts to correct for in
vignetting or global unevenness in the illumination. Images commonly
tend to be brighter in the center and dimmer at the edges, correcting
this unevenness makes it easier to find a single threshold value that
will cleanly separate all ROIs in the field of view from the background.

To run automatic ROI detection, select ***detect ROIs*** and adjust the
ROI threshold slider until the ROIs are cleanly separated from the
background. Displayed ROI numbers and bounding boxes show the
automatically assigned identity of each ROI in the image and its
boundaries. A vertical orientation indicated by font color will also be
assigned to each ROI. This is useful for ROIs with a clear vertical
assymetry such as the ones shown in **fig 3.2.1**. Orientation can
largely be ignored for ROIs without assymetry.

The following steps may help in cases where auto mode detection either
fails to pick up or inaccurately identifies one or more ROIs:

-   increase the threshold if it incorrectly assigns identities to
    non-ROIs

-   decrease the threshold if fails to detect real ROIs

-   ensure nothing dark or opaque bisects any ROIs

-   use [manual vignette correction](#vignettecorrection) if uneven
    illumination causes ROI dropping at the edges of the camera field of
    view

-   [manually edit](#ROIediting) individual ROIs if needed

![*figure 3.2.1* - Sample schematic (left) and ROI detection (right) of
an arena optimized for automatic detection. Non-grid structure of arenas
makes it unsuitable for grid detection mode. Automatic ROI detection
records vertical asymmetry of ROIs. In a Y-shaped arena, recording the
orientation makes it easy to infer endpoints of the maze
arms.](images/Starting a New Experiment/ymaze_96mazes_isometric.pdf){width="85%"}

**grid mode**

Grid mode ROI detection avoids the need for using imaging tricks to
assign ROIs but requires a little user input to make assignments. This
mode also makes the assumption that boundaries between ROIs can be drawn
in a regular grid-like structure.

After starting grid-based detection by selecting *detect ROIs*, the user
interface controls will be temporarily disabled as it waits for the user
to drag and drop a new grid into the imaging window. A grid settings
user panel will appear with options to add and remove grids as well as
change the dimensions of each grid. Once a grid is placed in the imaging
window, customize the grid by repositioning corners or editing the
number of rows and columns. The ROI bounds displayed in the imaging
window show the enclosed space in which a centroid will be assigned to a
particular ROI. The bounds should be positioned such that they fall in
the space arenas. New grids can be added or removed at any time by
selecting the + and - controls. Multiple grids allows several trays to
be imaged simultaneously by the same camera. This works particularly
well when imaging multiple well plates at the same time. Once finished
editing grids, select *Accept* to save the ROI positions.

![*figure 3.2.2* - Grid detection mode captures regularly arrayed and
low-contrast
ROIs.](images/Hardware Setup/Behavioral Arenas/ROIs_detected_1.png){width="50%"}

Background referencing
----------------------

Once ROI locations are set, MARGO creates a background reference image
to keep track of the frame to frame differences between the current
frame and the background image. MARGO uses periodic sampling of the
background throughout tracking to constantly update the background
appearance, but a reference must first be initialized before tracking
can begin. Because the arenas are not actually empty when this reference
is created, MARGO takes snapshots of each ROI separately any time a
tracked individual has moved far enough away from any position where a
previous snapshot was taken. A median image is computed for each ROI
separately and then combined into a single master reference for entire
field of view. Initializing a relatively accurate reference *very
important* because it allows for accurate noise profiling, which is
essential for robust tracking.

Select ***initialize reference*** to begin referencing. The imaging
window will display a thresholded difference image between the reference
and the current frame. Because the reference is first initialized to a
sample image at the start of referencing, the image should be largely
blank at first but begin to populate with individuals as they begin to
move. New samples of each ROI will be progressively collected as
individuals move around in each ROI. Circular indicators to the upper
left of each ROI will progress from purple to green as more references
are taken for each ROI, with green indicating referencing complete for a
given ROI. Adjust the tracking threshold until pixel noise is largely
absent from the thresholded image. Select *Accept* on the tracking
threshold slider to set the reference image. *Note:* Rougly half or more
of the ROIs at green should be sufficient to proceed to sampling noise
statitics.

Noise profiling
---------------

Difference imaging is extremely sensitive to even minor changes between
the background reference and the current frame. For that reason, minor
pertubations to the imaging such ambient vibrations over long time
scales or acute pertubations such as bumping tracking box can
drastically increase the noise in tracking. MARGO samples the
distribution of above threshold pixels during a sample, clean tracking
period of one hundred frames so that it can constantly monitor the
quality of and counter any deterioration in the difference image. For
this approach to work, the distribution collected during this period
must be a relatively accurate representation of what it should look
during the rest of the experiment.

Select ***confirm tracking*** to begin sampling. The tracking threshold
can be adjusted up or down to bias sampling to be either more or less
sensitive to noise during tracking. Noise sampling tips:

-   Try to get at least half of all ROIs completely referenced during
    the previous step

-   Slightly lower the tracking threshold if too few individuals are
    showing above the difference image

-   Obtain more accurate noise profiling or further lower the tracking
    threshold during sampling if noise threshold reference reset is
    constantly triggered during tracking

Experiment settings
-------------------

With the tracking setup complete, all that remains is to select the
experiment to be run and, if necessary adjust a few settings before
beginning tracking.

### select experiment

The experiment selection determines which experiment and data processing
protocols to execute once tracking begins. If you only want to record
centroid coordinates and time stamps, select *Basic Tracking* from the
dropdown menu and move to the next step. MARGO was designed not only to
record basic tracking data, but to run a suite of experimental
protocols. Those protocols differ in the raw data they record as well as
their hardware control schema and behavioral metrics recorded. Users can
define their own [custom experiments]{#customexperiment} or select from
a list of pre-defined protocols. More detailed explanations of the
pre-defined experiments in MARGO see the [ original MARGO
publication](https://www.google.com).

### parameters

If necessary, adjust the duration of tracking in hours, the number of
reference samples per minute, and the number of reference samples to
collect per ROI. Depending on the selected experiment, additional
customizable may be available under *experiment parameters*. Keep in
mind not all protocols, including *Basic Tracking* will have any
additional parameters to adjust.

### labels

Select *labels* to attach meta data to any particular range of ROIs. By
default, MARGO has fields for recording genetic strain, sex, treatment
condition, ID numbers, Day of testing, tracking box, tracking
arena/plate, and any additional comments. To append labels, enter a
range or ROIs the label applies to, fill in the labels, and select
*Accept*. Distinct categories can be entered in subsequent rows. *Note:*
MARGO will auto-generate a file label from the first row of entries.

### save path

Browse to a parent directory for the MARGO output. MARGO will
auto-generate a new directory within the chosen save path with the time
stamp and label information for the experiment where it will save all
raw and processed data.

::: {#saveprofile}
Save your settings - *recommended*
----------------------------------
:::

Before starting the experiment, it is *strongly* recommended to save a
profile for the current configuration of *MARGO*. By loading a saved
configuration profile in the future, you will avoid having to do any
parameter configuration. Only information such as the ROI positions and
reference image that is unique to each instance of tracking will not be
saved. Saving a profile not only substantially simplifies the setup
process, but also ensures consistency that will make it easier to
compare recordings across sessions.

To save a new profile, select *File* $>$ *save a new preset*. A profile
can be loaded at any time, but because presets contain information about
camera configuration, it will be necessary to re-initialize the camera
anytime a new preset is loaded.