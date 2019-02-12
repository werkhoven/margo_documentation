# Custom Experiment Tutorial

Every tracking protocol (eg. *Basic Tracking*, *Circadian* or *Y-maze*)
follows the same simple workflow consisting of a run file and optional parameter gui and analysis functions. An example of a custom experiment is shown below by
building off of MARGO's basic tracking protocol for the found in `../margo/experiments/Basic Tracking/`.

For the purposes of this tutorial, our goal will be to modify the basic tracking to add some functionality that will trigger turn on a light for any animal that comes within some minimum distance of the center of its ROI. Assume we have a behavioral configuration with multiple separate arenas and a small LED centered under each arena with all LEDs controlled by a single Arduino microcontroller recognized by MARGO as a serial COM device. Also assume that each ROI contains only one animal and that stimulus is binary (i.e. the LED can only be on or off). An animal receives a stimulus on any frame where it is within the minimum distance and receives no stimulus when it is not. 

## Overview of MARGO function inputs and outputs

Before diving into the code, we should explain how the data we'll be using to define our functions is organized. Most functions in MARGO pass in and out some combination of three variables that collectively define the core data of MARGO: `expmt`, `trackDat`, `gui_handles`.

### Experiment Data (expmt)

The `expmt` object is a custom MATLAB class (`ExperimentData`) that stores data specific to a particular experiment and is an important output of any tracking session in MARGO. The data stored in `expmt` broadly falls into four categories:

1. Data - Frame to frame raw data such as centroid coordinates and time stamps
2. Meta Data - ROI positions, date, time, experimental labels, file path information etc.
3. Parameters - Tracking parameters such as acquisition rate, image thresholds etc.
4. Hardware - MATLAB objects for cameras, displays, COM objects and associated parameters

In general, `expmt` contains information that is configured either during setup of the tracking in the MARGO GUI or in post-processing after the experiment is completed. See [MARGO Outputs - Master Data Container](google.com) for details on the `ExperimentData` class.

### Real-time Tracking Data (trackDat)

The `trackDat` struct contains all tracking data that is updated on a frame to frame basis such as the raw image data, timestamp, and centroid coordinates of each tracked object. For example, `trackDat.t` stores the time since the start of the experiment (in seconds) *for the current frame* and is overwritten with an updated timestamp at each subsequent frame. 

Of particular importance is the `fields` field of `trackDat`. The names of any trackDat fields matching the strings stored in `trackDat.fields` are flagged for writing to disk each frame. For example, by default MARGO always minimally saves centroid coordinates of tracked objects and timestamps for each frame. Therefore, the names of these fields are flagged in `trackDat.fields` during initialization:

```matlab
    trackDat.fields = {'centroid';'time'};
```

MARGO will look for any flagged fields of `trackDat` save the data stored in them to disk each frame. This means that somewhere during tracking we have to define fields of the same name data we want to save. For example, we already mentioned that `trackDat.t` stores the time elapsed since the start of the experiment. Therefore, to save an updated timestamp for each frame of the experiment, we could do the following:

```matlab
    trackDat.time = trackDat.t;
```

 We can take advantage this functionality to define other custom raw data fields to save at each frame as we'll see below.

### Handles to GUI controls (gui_handles)

The `gui_handles` struct contains all the MATLAB UI handles in MARGO's GUI. For example, the handle to axes where MARGO's image data is displayed can be accessed under `gui_handles.axes_handle`. These handles are used to access and configure the controls in MARGO's GUI and, as such, will generally not be useful to most users. However, they can be used to define display routines that provide visual feedback on the functioning of custom code.

## Creating a new experimental template

We can generate a template for a new custom experiment from the MARGO GUI under **File > new custom experiment** and enter a name for the new experimental protocol. We'll choose the name **center_stim** for this tutorial. Once completed, MARGO will create a new directory and two new files for the new protocol:

```
    ../margo/experiments/center_stim/
    ../margo/experiments/center_stim/run_center_stim.m
    ../margo/experiments/center_stim/analyze_center_stim.m
```

By structuring the directory and files this way, MARGO will detect a new protocol with the name we specified and functions to run the acquisition and analysis for the experiment. Upon opening, MARGO looks for keyword tags in the m-file names within each experiment directory to assign specific functionality to each file.

### M-file keyword tags

Any keywords below found in the file name of an experimental protocol's m-files will flag them following functionality within the protocol:

- run - main function to run data acquisition
- analyze - post-processing/analysis function
- sub_gui - GUI function to set custom parameters experiment via the MARGO GUI Experiment Parameters control button

Only the run and analyze m-files are generated by default, as shown in the example above.

## Custom run file

The run file `run_center_stim.m` is initialized as a copy of MARGO's Basic Tracking protocol and contains all the minimal functionality to conduct tracking in MARGO. The run file can be divided into protocol initialization and the tracking loop.

### Initialization

For most experiments, initialization will consist of some combination of:

- defining new raw data fields to save each frame
- defining variables to necessary to perform our custom tasks in real time
- initializing custom hardware 

Before adding any new functionality, let's take a look at the code in our new run file.

```matlab
% Initialization: Get handles and set default preferences
gui_notify(['executing ' mfilename '.m'], gui_handles.disp_note);

% get image handle
imh = findobj(gui_handles.axes_handle,'-depth', 3, 'Type', 'image');  

% raw data fields to save each frame (append names of custom fields to
% flag fields for writing to disk)
trackDat.fields = {'centroid'; 'time'};   

% initialize labels, files, and cam/video
[trackDat,expmt] = autoInitialize(trackDat, expmt, gui_handles);

% --------------------------------- %
% insert custom initialization here %
% --------------------------------- %
```
The code above is the bare minimum required to run any protocol in MARGO. Most importantly, it defines what data in `trackDat` will be saved as raw data each frame. By default, this is just centroid coordinates and timestamps. Among other things, it also ensures that we have an open input source for image data (camera or video) and creates all the raw binary data files in `autoInitialize`. We can start to add some new functionality by defining some custom initialization in the space below. We'll start by flagging a new raw data field `stimulus` to record which animals received a stimulus on any given frame.

```matlab
trackDat.fields = {'centroid'; 'time'; 'stimulus'};   
```

Adding this field name tells MARGO to expect a new field `trackDat.stimulus` and to initialize a new raw data file `stimulus.bin` to store that data. At this point, our new field is undefined. We will have to define our new field with the appropriate size and data type to save on each frame. The data type should be binary and we should initialize the stimulus for each ROI to false (OFF). In total we should have one value for each ROI.

```matlab
trackDat.stimulus = false(expmt.meta.roi.n, 1);
```

We'll also define another field `trackDat.center_distance` to store the distance of each animal from the center of their ROI at each frame.

```matlab
trackDat.center_distance = NaN(expmt.meta.roi.n,1);
```
The basic initialization template has been modified to include initialization of our custom fields. By adding `'stimulus'` to the list of fields stored in `trackDat.fields`, MARGO will expect us to define a new field of trackDat with the name "stimulus". Once tracking begins, any data stored in `trackDat.stimulus` will be written to the hard drive each frame. Our second field, `trackDat.center_distance` will not be saved to disk each frame because we did not flag it.

The Arduino to control our LEDs does not need to be initialized assuming that the device has been selected as the auxillary COM device in MARGO's GUI. No additional hardware configuration is needed.

The complete custom initialization will look something like this:

```matlab
% Initialization: Get handles and set default preferences
gui_notify(['executing ' mfilename '.m'], gui_handles.disp_note);

% get image handle
imh = findobj(gui_handles.axes_handle,'-depth', 3, 'Type', 'image');  

% raw data fields to save each frame (append names of custom fields to
% flag fields for writing to disk)
trackDat.fields = {'centroid'; 'time'; 'stimulus'};   

% initialize labels, files, and cam/video
[trackDat,expmt] = autoInitialize(trackDat, expmt, gui_handles);

% insert custom initialization here %
trackDat.stimulus = false(expmt.meta.roi.n, 1);
trackDat.center_distance = NaN(expmt.meta.roi.n,1);
```

### Tracking Loop

The real functionality of the custom experiment will come in the tracking loop. Before adding custom routines to the tracking loop, let's look at the basic functionality already provided.

```matlab
% run experimental loop until duration is exceeded or last frame
% of the last video file is reached
while ~trackDat.lastFrame

    % update time stamps and frame rate
    [trackDat] = autoTime(trackDat, expmt, gui_handles);

    % get next frame of image data
    [trackDat,expmt] = autoFrame(trackDat,expmt,gui_handles);

    % process image data, extract objects, and sort to ROIs
    trackDat = autoTrack(trackDat,expmt,gui_handles);

    % ---------------------- %
    % insert custom routines %
    % ---------------------- %

    % output flagged fields to raw data binary files
    [trackDat,expmt] = autoWriteData(trackDat, expmt, gui_handles);

    % update background reference or reset if noise thresh is exceeded
    [trackDat, expmt] = autoReference(trackDat, expmt, gui_handles);

    % update the GUI display
    trackDat = autoDisplay(trackDat, expmt, imh, gui_handles);
end
```
The main tracking loop consists of six core sub-routines essential to
any experiment in MARGO:

1.  update time-keeping variables
2.  query the next frame of data from camera or video file
3.  track objects in the current frame and match identities to previous
    frames
4.  write each field of raw data in *trackDat.fields* to the hard drive
5.  check to see if the background reference needs to be update
6.  update the display if necessary

 Custom routines should be implemented between steps 3-4 (tracking and raw data output) for real time control. Inserting custom routines downstream of the tracking allows us to take advantage of the most recent estimate of each animals' position in targeting the stimulus. Being upstream of the raw data output allows us to define our custom raw data field and output it with the tracking data from the current frame.

We'll insert a single function `update_center_stim` to perform all of our custom tasks on each frame as a placeholder and define later. After inserting the new function, the tracking loop will look something like this:

```matlab
% run experimental loop until duration is exceeded or last frame
% of the last video file is reached
while ~trackDat.lastFrame

    % update time stamps and frame rate
    [trackDat] = autoTime(trackDat, expmt, gui_handles);

    % get next frame of image data
    [trackDat,expmt] = autoFrame(trackDat,expmt,gui_handles);

    % process image data, extract objects, and sort to ROIs
    trackDat = autoTrack(trackDat,expmt,gui_handles);

    % update the stimulus for each animal
    trackDat = update_center_stim(trackDat, expmt);

    % output flagged fields to raw data binary files
    [trackDat,expmt] = autoWriteData(trackDat, expmt, gui_handles);

    % update background reference or reset if noise thresh is exceeded
    [trackDat, expmt] = autoReference(trackDat, expmt, gui_handles);

    % update the GUI display
    trackDat = autoDisplay(trackDat, expmt, imh, gui_handles);
end
```

### Defining a custom routine

**Inputs**

Calculating the the distance of each animal to the center of its ROI will require two pieces of information:

- the current position of each animal (`trackDat.centroid`)
- the position of the center of each ROI (`expmt.meta.roi.centers`)

We will define the function to take both `trackDat` and `expmt` as inputs. Providing `expmt` as an input also allows us to write data to the Arduino via the serial object for the auxillary COM port initialized through the MARGO GUI ( `expmt.hardware.COM.aux`).

**Outputs**

To write the status of the LEDs to hard drive, we will need to define `trackDat.stimulus` inside the function and output an updated version of `trackDat`.

**The Function**

Our custom routine will need to perform three tasks each frame:

<p>1. Calculate the distance of each animal from the center of its arena</p>

```matlab
% define function to calculate euclidean distance between two Nx2 vectors and calculate
euclid_dist = @(a,b) sqrt(sum((a-b).^2,2));
trackDat.center_distance = euclid_dist(trackDat.centroid, expmt.meta.roi.centers);
```
<p>2. Determine which animals should receive the stimulus</p>

Assuming we want to deliver the stimulus to any animal within 10 pixels of the center of its ROI.

```matlab
% define which animals are within 10 pixels of ROI center
trackDat.stimulus = trackDat.center_distance < 10;
```

This will define an Nx1 logical vector that is `true` where the distance to the ROI center is less than 10. Because we flagged the field named `stimulus` as a raw data field during initialization, the data in this vector will be appended to the `stimulus.bin` raw data file each frame. In principle, we might like to make this distance threshold an [adjustable parameter](#custom-experiment-parameters-gui-optional) within the MARGO GUI.

<p>3. Update the stimulus by writing to the Arduino</p>

```matlab
% write data to the auxillary COM port
fwrite(expmt.hardware.COM.aux, char(trackDat.stimulus), 'uchar');
```

This will write each value for our LEDs (true/false) as a byte of ASCII data to the serial port. This example, of course, assumes that the Arduino on the other end of the port will know how to parse the data we write to it and write the correct value to each LED. Programming microcontrollers is beyond the scope of this tutorial. See [Arduino's tutorials](https://www.arduino.cc/en/Guide/HomePage) for getting started with controlling Arduinos and other similar microcontrollers.

Altogether, the function for our custom routine will look something like this:

```matlab
function trackDat = update_center_stim(trackDat, expmt)

% define function to calculate euclidean distance between two Nx2 vectors and calculate
euclid_dist = @(a,b) sqrt(sum((a-b).^2,2));
trackDat.center_distance = euclid_dist(trackDat.centroid, expmt.meta.roi.centers);

% define which animals are within 10 pixels of ROI center
trackDat.stimulus = trackDat.center_distance < 10;

% write data to the auxillary COM port
fwrite(expmt.hardware.COM.aux, char(trackDat.stimulus), 'uchar');
```

## Custom analysis pipeline (optional)

### Analysis template

Defining a custom analysis routine enables automatic pre-processing and analysis of MARGO data to be toggled from the output options menu in the MARGO GUI under **Options > output**. Toggling the `enable` option tells MARGO to automatically execute an experimental protocol's analyze m-file immediately following tracking. 

By defaults, the analysis template consists of the `autoDataProcess` routine which conducts additional pre-processing and analyses from the centroid data based on the options selected in the MARGO GUI's [output options](https://www.debivortlab.org/margo/outputs.html) menu:

- parses traces into discreet movement bouts
- models fisheye lens distortion
- calculates and saves additional features from centroid traces:
    - speed
    - heading angle
    - four quadrant inverse tangent
    - distance to ROI center
    - handedness

### Adding custom analysis

For the purposes of this tutorial we will calculate a very simple metric, `occupancy`, which will just be defined as the fraction of the total experiment each animal spent with the light turned on. By default, the custom template will conduct the pre-processing flagged in MARGO as described above and will save the `expmt` object updated with our analysis to file.

```matlab
function expmt = analyze_center_stim(expmt,varargin)

% This function provides a sample analysis function to run after the
% run_basictracking.m. It takes the experimental master data
% struct (expmt) as an input, processes the data to extract features
% and store them to file.

% Parse pre-processing options, process centroid data
[expmt,options] = autoDataProcess(expmt,varargin{:});

% add a new property to the stimulus raw data field to store our new metric
addprop(expmt.data.stimulus, 'occupancy')

% calculate the number of frames with the stimulus ON (stimulus=1)
frames_on = sum(expmt.data.stimulus.raw());

% divide number of frames ON by the total number of frames
expmt.data.stimulus.occupancy =  frames_on ./ expmt.meta.num_frames;

% Update and save expmt to file, close open raw data files,
autoFinishAnalysis(expmt,options)
```

The raw data we wrote to `stimulus.bin` during acquisition can be used to calculate our custom metric. The raw data is accessible in the `expmt` object via custom MARGO [RawDataMap](https://www.debivortlab.org/margo/outputs.html) objects. Flagging the `'stimulus'` field under `trackDat.fields` automatically generates a RawDataMap under `expmt.data.stimulus.raw`. RawDataMaps take indices as arguments just like MATLAB arrays. Passing no indices the RawDataMap `expmt.data.stimulus.raw()` returns the raw data in its native dimensions (number of frames x number of ROIs).


## Custom experiment parameters GUI (optional)

The only parameter of the custom protocol example above is the minimum distance to the ROI center required to turn on the stimulus. A hard-coded constant was used to set the parameter above, but linking custom GUIs to experimental protocols allows the minimum distance parameter and other custom parameters to be set from within the MARGO GUI.

Custom parameter GUIs must satisfy the following constraints to be correctly detected and implemented in MARGO:

1. GUI m-file name contains the keyword "sub_gui"
2. The ExperimentData object (`expmt`) is defined as the GUI function's only input and output
3. New parameters are assigned to `expmt.parameters.(name)`

Once the GUI function is configured, the GUI will become accessible via the MARGO GUI Experiment Parameters button. See [MATLAB's tutorial on GUIDE](https://www.mathworks.com/help/matlab/creating_guis/about-the-simple-guide-gui-example.html;jsessionid=4ac08f6f0a44aa36865882ca84ab) for details on creating custom GUIs in MATLAB.

