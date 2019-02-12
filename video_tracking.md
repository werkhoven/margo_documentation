Tracking with Video Input
=========================

MARGO accepts video files as an input source for tracking. Select
*Source* $>$ *video file* to switch inputs. Many programs will output
raw video data in *avi2* format, which has a 2GB file size limit. When
the file size limit is reached, subsequent files are automatically
generated. To allow sequential tracking across multiple files, MARGO
will automatically track all videos under the target directory. The
software makes the assumption that all files under the same directory
belong to the same experiment and will only generate a single .mat
output file. The software can take *.avi*, *.mp4*, *.m4v*, *.mov*,
*.wmv* and *.mpg* file formats as input. Because video files are not
tracked in real time, the time-stamps output from video file tracking
are inferred from the frame rate of the file and may not be accurate.