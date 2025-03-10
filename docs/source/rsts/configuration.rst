=============
Configuration
=============

We provide the explanation of the configurations in this file.
A KITTI experiment is used as an example.
We also provide possible choices in the current framework.
We have some unused configurations (will be commented) 
for our Experiment Version but not used in this Release Version.

Notes:

- **None**: when the choice is None, it means leave the option blank. DON'T put 'None', which yaml would recognize as string 'None'


.. code:: yaml

    # ------------------------------------
    # Basic setup
    # ------------------------------------
    dataset: kitti_odom                     # dataset [kitti_odom, kitti_raw, tum-1/2/3, adelaide1/2]
    seed: 4869                              # random seed
    image:
        height: 192                         # image height
        width: 640                          # image width
        ext: png                            # image file extension for data loading
    seq: "10"                               # sequence to run
    frame_step: 1                           # frame step
    
    # ------------------------------------
    # Directories
    # ------------------------------------
    directory:
        result_dir: {RESULT_DIR}            # directory to save result
        img_seq_dir: {IMAGE_DATA_DIR}       # image data directory
        gt_pose_dir: {GT_POSE_DIR}          # (optional) ground-truth pose data directory
        depth_dir: {DEPTH_DATA_DIR}         # (optional) external depth data, e.g. ground-truth depths

    # ------------------------------------
    # Depth
    # ------------------------------------
    depth:                                  # Depth configuration
        depth_src:                          # depth source [None, gt]
                                                # None - depth model predition
                                                # gt - use ground truth depth
        deep_depth:
            network: monodepth2             # depth network
            pretrained_model: {MODEL_DIR}   # directory stores depth.pth and encoder.pth 
        max_depth: 50                       # maximum depth 
        min_depth: 0                        # minimum depth 
    
    # ------------------------------------
    # Deep flow
    # ------------------------------------
    deep_flow:                              # Deep optical flow configuration
        network: liteflow                   # optical flow network, [liteflow]
        flow_net_weight: {FLOW_MODEL}       # optical flow model path
        forward_backward: True              # predict both forward/backward flows and compute forward-backward flow consistency
    
    # ------------------------------------
    # Deep Pose (Experiment Ver. only)
    # ------------------------------------
    deep_pose:                              # Deep pose network configuration
        enable: False                       # enable/disable pose network
        pretrained_model: {MODEL_DIR}       # model directory contains pose_encoder.pth and pose.pth
    
    # ------------------------------------
    # Online Finetuning
    # ------------------------------------
    online_finetune:                        # online fine-tuning configuration
        enable: False                       # enable/disable flow finetuning
        lr: 0.00001                         # learning rate
        num_frames:                         # number of frames to be fine-tuned, [None, int]
        flow:                               # flow fine-tuning configuration
            enable: False                   # enable/disable flow finetuning
            scales: [1, 2, 3, 4, 5]         # scales to be used for training
            loss:                           # flow loss configuration
                flow_consistency: 0.005     # forward-backward flow consistency loss weight
                flow_smoothness: 0.1        # flow smoothness loss weight
        depth:                              # depth fine-tuning configuration
            enable: False                   # enable/disable depth finetuning
            scales: [0, 1, 2, 3]            # scales to be used for training
            pose_src: DF-VO                 # pose source for depth-pose finetuning [DF-Vo, deep_pose]
            loss:                           # depth loss configuration
                apperance_loss: 1           # apperance loss weight
                disparity_smoothness: 0.001 # disparity smoothness loss weight
                depth_consistency: 0.001    # depth consistency loss weight
        pose:                               # pose finetuning configuration (with depth)                  
            enable: False                   # enable/disable pose finetuning

    # ------------------------------------
    # Preprocessing
    # ------------------------------------
    crop:                                   # cropping configuration
        depth_crop: [[0.3, 1], [0, 1]]      # depth map cropping, format: [[y0, y1],[x0, x1]]
        flow_crop: [[0, 1], [0, 1]]         # optical flow map cropping, format: [[y0, y1],[x0, x1]]
    
    # ------------------------------------
    # Correspondence (keypoint) selection
    # ------------------------------------
    kp_selection:                           # correspondence selection configuration
        local_bestN:                        # local best-N configuration
            enable: True                    # enable/disable local best-N selection
            num_bestN: 2000                 # number of keypoints
            num_row: 10                     # number of divided rows
            num_col: 10                     # number of divided columns
            score_method: flow              # selection score, [flow, flow_ratio]
                                                # flow: L2 distance of forward-backward flow 
                                                # flow_ratio: relative flow difference ratio
            thre: 0.1                       # flow consistency masking threshold
        bestN:
            enable: False                   # enable/disable best-N selection
            num_bestN: 2000                 # number of keypoints
        sampled_kp:                         # random/uniform keypoint sampling
            enable: False                   # enable/disable random/uniform keypoint sampling
            num_kp: 2000                    # number of keypoints
        rigid_flow_kp:                      # keypoint selection from optical-rigid flow consistency (for scale recovery)
            enable: False                   # enable/disable rigid-flow based keypoint selection
            num_bestN: 2000                 # number of keypoints
            num_row: 10                     # number of divided rows
            num_col: 10                     # number of divided columns
            score_method: flow              # selection score, [flow]
            rigid_flow_thre: 3              # masking threshold for rigid-optical flow consistency 
            optical_flow_thre: 0.1          # masking threshold for forward-backward flow consistency 
        depth_consistency:                  # (Experiement Ver. only) depth consistency configuration
            enable: False                   # enable/disable depth consistency
            thre: 0.05                      # masking threshold

    # ------------------------------------
    # Tracking
    # ------------------------------------
    tracking_method: hybrid                 # tracking method [hybrid, PnP, deep_pose]
                                                # hybrid - E-tracker + PnP-tracker; 
                                                # PnP - PnP-tracker
                                                # deep_pose - pose_cnn-tracker
    
    e_tracker:                              # E-tracker configuration
        ransac:                             # Ransac configuration
            reproj_thre: 0.2                # inlier threshold value
            repeat: 5                       # number of repeated Ransac
        validity:                           # model selection condition
            method: GRIC                    # method of validating E-tracker, [flow, GRIC]
            thre:                           # threshold value for model selection, only used in [flow]
        kp_src: kp_best                     # type of correspondences to be used [kp_list, kp_best]
                                                # kp_list - uniformaly sampled keypoints
                                                # kp_best - keypoints sampled from best-N / local best method

    scale_recovery:                         # scale recovery configuration
        method: simple                      # scale recovery method [simple, iterative]
        ransac:                             # Ransac configuration
            method: depth_ratio             # fitting target [depth_ratio, abs_diff]
                                                # depth_ratio: find a scale s.t. most triangulated_depth/cnn_depth close to 1
                                                # abs_diff: find a scale s.t. abs(triangulated_depth - cnn_depth) close to 0
            min_samples: 3                  # minimum number of min_samples
            max_trials: 100                 # maximum number of trials
            stop_prob: 0.99                 # The probability that the algorithm produces a useful result
            thre: 0.1                       # inlier threshold value
        kp_src: kp_best                     # type of correspondences to be used [kp_list, kp_best, kp_depth]
                                                # kp_list - uniformaly sampled keypoints
                                                # kp_best - keypoints sampled from best-N / local best method
                                                # kp_depth - keypoints sampled after optical-rigid flow consistency masking
    
    pnp_tracker:                            # PnP-tracker configuration
        ransac:                             # Ransac configuration
            iter: 100                       # number of iteration
            reproj_thre: 1                  # inlier threshold value
            repeat: 5                       # number of repeated Ransac
        kp_src: kp_best                     # type of correspondences to be used [kp_list, kp_best, kp_depth]
                                                # kp_list - uniformaly sampled keypoints
                                                # kp_best - keypoints sampled from best-N / local best method
                                                # kp_depth - keypoints sampled after optical-rigid flow consistency masking
    
    # ------------------------------------
    # Visualization
    # ------------------------------------
    visualization:                          # visualization configuration
        enable: True                        # enable/disable frame drawer
        save_img: True                      # enable/disable save frames
        window_h: 900                       # frame window height
        window_w: 1500                      # frame window width
        kp_src: kp_best                     # type of correspondences to be drawn
        flow:                               # optical flow visualization configuration
            vis_forward_flow: True          # enable/disable forward flow visualization
            vis_backward_flow: True         # enable/disable backward flow visualization
            vis_flow_diff: True             # enable/disable forward-backward flow consistency visualization
            vis_rigid_diff: False           # enable/disable optical-rigid flow consistency visualization
        kp_match:                           # keypoint matching visualization
            kp_num: 100                     # number of selected keypoints to be visualized
            vis_temp:                       # keypoint matching in temporal 
                enable: True                # enable/disable visualization
            vis_side:                       # keypoint matching side-by-side
                enable: True                # enable/disable visualization
                inlier_plot: False          # enable/disable inlier plot
        trajectory:                         # trajectory visualization configuration
            vis_traj: True                  # enable/disable predicted trajectory visualization
            vis_gt_traj: False              # enable/disable ground truth trajectory visualization
            mono_scale: 1                   # scaling factor to align with gt (if gt is available)
        depth:                              # depth visualization configuration
            use_tracking_depth: False       # enable/disable visualizing depth map used for tracking (preprocessed, e.g. range capping)
            depth_disp: disp                # visualize depth or disparity map [depth, disp]
