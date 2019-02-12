Visual Stimulus Delivery with Psychtoolbox
==========================================

MARGO supports closed-loop targeting of visual stimuli to individual
ROIs using an external display or projector. Visual stimuli can be
crafted and displayed with
[Psychtoolbox](http://psychtoolbox.org/overview/), a freely-available,
MATLAB based software package for psychology and neuroscience research.
Psychtoolbox is not included with MARGO and must be [downloaded and
installed](http://psychtoolbox.org/download/) separately. Psychtoolbox
is an incredibly powerful and expansive package developed by Mario
Kleiner. See one of the many [online
resources](http://peterscarfe.com/ptbtutorials.html) to learn how to
generate and display stimuli.

Projector box
-------------

Mounting an overhead projector to the behavioral box allows targeting of
visual stimuli to individual ROIs. The camera can be placed slightly
off-center so the behavioral platform is visible to both the camera and
projector. Equipping a long-pass filter to the camera allows tracking
done in infrared without interference from the projector (fig. 8.1).

![*figure 8.1* - Sample schematic of projector behavioral box. An
external long-pass filter can be rotated in and out of the imaging path
allowing the camera to switch between imaging only infrared light and
imaging both infrared and visible light. The overhead projector targets
individual ROIs underneath by projecting visual stimuli on a diffuser
film on the floor of the behavioral
arenas.](images/projector/proj_schematic.pdf){width="95%"}

In order to accurately target objects within the camera's field of view
with the projector, a mapping must be made between the pixel coordinates
of both the camera and projector. MARGO creates this mapping by tracking
a spot that is rastered through the projector's display. For this to
work, the projection must be visible to the camera during registration
but invisible to the camera during tracking. Mounting a long-pass filter
on a swivel makes it easy to switch between the two without disturbing
the camera.

Registering the projector to the camera
---------------------------------------

Any coordinate mapping between the camera and the projector will only
accurate for a particular configuration and positioning of both. If
either is moved during or after registration, the process must be
started over again to have an accurate mapping. Having a high contrast,
non-reflective projection surface will greatly improve the ease of
tracking the projector with the camera. A flat white sheet of paper or
matte white painted surface works well. Do the following to register the
projector in MARGO:

1.  Ensure that Psychtoolbox is installed by executing
    ***PsychtoolboxVersion*** in the MATLAB command window (MARGO
    support built on version 3)

2.  Remove the infrared filter from the imaging path and start the
    camera preview

3.  Place a high-contrast, white surface on the behavioral platform

4.  Ensure no sources of illumination but the projector will be visible
    to the camera

5.  Drag a window into the projector display (anything with
    high-contrast black and white text) and ensure the projector is in
    focus

6.  Adjust the camera aperture or exposure (under camera settings) and
    focus until the projector display is clearly visible in the camera
    preview

7.  Select ***Hardware*** $>$ ***projector*** $>$ ***register
    projector***

8.  Select and confirm the projector screen and registration parameters
    when prompted to begin registration

    -   *Grid Step Size* sets how finely the projector rasters its field
        of view

    -   *Spot Radius* sets the size of the rastered dot (the apparent
        size should be between 1-5% the area of the camera)

9.  Once registration is finished, the output mapping will be saved to
    file under: ***MARGO/hardware/projector\_fit/***

Using registration output in experiments
----------------------------------------

Once registration is complete, no additional configuration of the
projector or registration mapping is required to run the prepackaged
***Optomotor***, ***Slow Phototaxis***, and ***Temporal Phototaxis***
experiments. To use the mapping in custom experiments, run the following
command:

**ExperimentData* = initialize\_projector(*ExperimentData*, bg\_color)*

This initializes a Psychtoolbox window to the screen set under
registration parameters with the background color specified by the RGB
triplet, *bg\_color*. Additionally, projector initialization stores the
registration mapping and screen properties to the **ExperimentData**
struct. To convert from camera coordinates to projector coordinates, use
the 2D scattered interpolants stored under **ExperimentData*.projector*:

*projector\_x = *ExperimentData*.projector.Fx(cam\_x, cam\_y)*;\
*projector\_y = *ExperimentData*.projector.Fy(cam\_x, cam\_y)*;

![*Table 8.3* - Field names and descriptions of the screen properties
output to **ExperimentData*.scrProp* following projector initialization.
The properties stored here be used to accurately time and display
stimuli to the Pyschtoolbox window opened during
initialization.](images/projector/scrProps.pdf){width="95%"}

Detailed use of MARGO and Psychtoolbox to deliver stimuli under
closed-loop control is beyond the scope of this manual, but sample
implementations are available in ***run\_optomotor.m*** and
***run\_slowphototaxis.m***.
