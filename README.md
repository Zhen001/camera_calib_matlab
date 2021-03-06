# camera_calib_matlab
This is a camera calibration toolbox. It's partly based on [Bouguet's toolbox](http://www.vision.caltech.edu/bouguetj/calib_doc/) and Zhang's camera calibration paper, but with more functionality:

* Setup is based on an input configuration file which allows for easy tweaking and modification of algorithms and allows for greater reproducibility. If you save the images, configuration file, and script, the calibration will be repeatable.
* Includes fiducial marker recognition which makes the calibration fully automatic.
* The distortion function is input as a symbolic function (via configuration file) and is therefore very easily modifiable. Two distortion functions are provided already ("heikkila97" and "wang08"); this toolbox uses symbolic differentiation to compute the updated jacobians/hessians/gradients automatically.
* Supports multi-camera calibration.
* Implements both "distortion refinement" and "frontal refinement" techniques.
* Supports multiple calibration board targets (checkers, circles, etc...) and correctly accounts for "center of ellipse" vs "projected center of circle" for circular targets depending on the type of calibration (i.e. "frontal refinement" or "distortion refinement").
* Supports custom calibration board geometries by overriding an abstract calibration board geometry class.
* Supports (optional) covariance optimization (i.e. generalized least squares) based on uncertainties computed during target localization.
* Supports calibration board going partially "out of frame" which improves robustness and allows for bigger calibration boards to be used.
* Code is organized, documented, and utilizes object oriented principles for code reuse.

# Installation instructions:
1) Clone the repo:
```
git clone https://github.com/justinblaber/camera_calib_matlab.git
```

# Example:
1. First, download images/config/script from [here](http://justinblaber.org/downloads/github/camera_calib/dotvision_checker.zip).
2. Unzip, navigate to the folder in matlab, and check the configuration script, `dotvision_checker.conf`:
```
% Checkerboard target
target              = checker
target_optimization = edges

% Checkerboard geometry
height_cb           = 50.8
width_cb            = 50.8
num_targets_height  = 16
num_targets_width   = 16
target_spacing      = 2.032
height_fp           = 42.672
width_fp            = 42.672
obj_cb_geom         = class.cb_geom.csgrid_cfp

% Optimization
calib_optimization  = distortion_refinement

% Plotting
camera_size         = 20
```
The targets are checkers and the target optimization is a cool "edges" refinement algorithm from Mallon07. The calibration board geometry class used is "csgrid_cfp" which is a centered square grid with centered "four point" fiducial markers.

3. Next, open the `dotvision_checker.m` script in matlab, which should include:
```
clear; clc;

% Read calibration config
calib_config = intf.load_calib_config('dotvision_checker.conf');

% Set images
path_cbs(:,1) = {
'SERIAL_19061245_DATETIME_2019-05-30-01:17:49-279539_CAM_1_FRAMEID_0_COUNTER_1.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:19:10-092852_CAM_1_FRAMEID_0_COUNTER_2.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:20:05-049316_CAM_1_FRAMEID_0_COUNTER_3.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:20:53-252540_CAM_1_FRAMEID_0_COUNTER_4.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:21:26-059003_CAM_1_FRAMEID_0_COUNTER_5.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:22:22-406834_CAM_1_FRAMEID_0_COUNTER_6.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:22:55-725990_CAM_1_FRAMEID_0_COUNTER_7.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:23:34-075146_CAM_1_FRAMEID_0_COUNTER_8.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:24:13-309275_CAM_1_FRAMEID_0_COUNTER_9.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:24:50-385473_CAM_1_FRAMEID_0_COUNTER_10.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:25:35-707981_CAM_1_FRAMEID_0_COUNTER_11.png', ...
'SERIAL_19061245_DATETIME_2019-05-30-01:26:07-053008_CAM_1_FRAMEID_0_COUNTER_12.png'
};
             
path_cbs(:,2) = {
'SERIAL_16276941_DATETIME_2019-05-30-01:17:49-279573_CAM_2_FRAMEID_0_COUNTER_1.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:19:10-092894_CAM_2_FRAMEID_0_COUNTER_2.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:20:05-049342_CAM_2_FRAMEID_0_COUNTER_3.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:20:53-252565_CAM_2_FRAMEID_0_COUNTER_4.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:21:26-059023_CAM_2_FRAMEID_0_COUNTER_5.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:22:22-406855_CAM_2_FRAMEID_0_COUNTER_6.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:22:55-726022_CAM_2_FRAMEID_0_COUNTER_7.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:23:34-075170_CAM_2_FRAMEID_0_COUNTER_8.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:24:13-309309_CAM_2_FRAMEID_0_COUNTER_9.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:24:50-385500_CAM_2_FRAMEID_0_COUNTER_10.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:25:35-708005_CAM_2_FRAMEID_0_COUNTER_11.png', ...
'SERIAL_16276941_DATETIME_2019-05-30-01:26:07-053021_CAM_2_FRAMEID_0_COUNTER_12.png', ...
};

path_cbs(:,3) = {
'SERIAL_16276942_DATETIME_2019-05-30-01:17:49-279580_CAM_3_FRAMEID_0_COUNTER_1.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:19:10-092897_CAM_3_FRAMEID_0_COUNTER_2.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:20:05-049351_CAM_3_FRAMEID_0_COUNTER_3.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:20:53-252567_CAM_3_FRAMEID_0_COUNTER_4.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:21:26-059040_CAM_3_FRAMEID_0_COUNTER_5.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:22:22-406871_CAM_3_FRAMEID_0_COUNTER_6.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:22:55-726019_CAM_3_FRAMEID_0_COUNTER_7.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:23:34-075180_CAM_3_FRAMEID_0_COUNTER_8.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:24:13-309307_CAM_3_FRAMEID_0_COUNTER_9.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:24:50-385526_CAM_3_FRAMEID_0_COUNTER_10.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:25:35-708001_CAM_3_FRAMEID_0_COUNTER_11.png', ...
'SERIAL_16276942_DATETIME_2019-05-30-01:26:07-053028_CAM_3_FRAMEID_0_COUNTER_12.png', ...
};

% Validate all calibration board images
img_cbs = intf.validate_multi_path_imgs(path_cbs);

% Display calib config
intf.plot_cb_geom(calib_config);

% Detect four points
[p_fpss, debug_fp] = intf.fp_detect(img_cbs, calib_config);

% four point gui
intf.gui_fp_detect(p_fpss, img_cbs, debug_fp, calib_config)

% Perform calibration
calib = intf.calib_fp(img_cbs, p_fpss, calib_config);
                           
% Calib gui
intf.gui_calib(calib);
```
Make sure to first add the library to path with something like: `addpath('~/camera_calib')`

After running this script, the first figure to appear should be:
![four point detector](https://i.imgur.com/mTd2DGF.png)

This gui is useful for debugging the detection of the four fiducial markers. You can toggle through the images by pressing the left and right arrow keys.

The next figure should be:
![calibration](https://i.imgur.com/yjqjuBV.png)

This gui is useful for debugging the calibration. The first thing to do is to double check the calibration board geometry to make sure it's correct. Next, check the residuals to make sure they are reasonably small. I would also rotate the extrinsics to confirm the relative pose of the cameras makes sense. Lastly, you can toggle "w" and zoom into the calibration point with the largest residual.

