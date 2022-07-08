
# 活体项目

## 可见光活体
- 位置：/home/xjyu/lgx/Live/new_live
- 图片大小：(96,96,1) Txx/T40-8bit模型
- 环境：conda activate new_live

## 单目红外活体
- 位置：/home/xjyu/lgx/Live/anti_spoofing
- 图片大小：(96,96,1) Txx-8bit模型
- 环境：conda activate live

## 双目红外活体
### 1）帧差法：
- 位置：/home/xjyu/lgx/Live/face_anti_spoofing/anti_spoofing
- 图片大小：(128,128,1) Txx-8bit模型，中心差方卷积
- 环境：conda activate cdcn
- 方法：使用镜头的亮帧亮帧数据-暗帧数据，避免环境光的影响，使用帧差后的图进行人脸检测，确定人脸框的位置，对人脸框以最大边进行上下左右扩充，此过程需要对亮帧数据进行过曝检测，剔除过曝数据，对扩充后的框进行裁剪，resize至（128，128），进入帧差活体网络进，输出活体或非活体。

### 2）视差法：
- 位置：/home/xjyu/lgx/Live/disp_live1
- 图片大小：(128,128,1) Txx-8bit模型
- 环境：conda activate disp
- 方法：使用左图进行人脸检测，确定人脸框的位置，对人脸框以最大边进行上下左右扩充（5：4），此过程需要对左图进行过曝检测，剔除过曝的图片，以该扩充框裁剪左右图，对裁剪图进行均值归一化处理，使图片变亮、均衡，对归一化的图片进行特征点匹配，来替换双目立体矫正对齐的步骤，求出左右图中行、列方向的差距，对右图在裁剪框的基础上添加高度、宽度差距，并重新裁剪，该裁剪后的图，确定为左右图行方向一致，列方向有一定视差，使用裁剪对齐后的图，resize至（160，128），进行双目立体匹配，求得视差图（128，128），将此视差图进行视差网络判断，输出活体或非活体。



- 优化方法1：
    - 现存在的问题：近距离的非活体会检测为活体，分析存在的原因：虽然纸张近距离的深度信息不明显，但是轮廓还是比较清晰的，这跟深度信息不明显，但人脸轮廓比较清晰的活体存在冲突，可以向外扩充人脸框，使用活体的身体信息，以及非活体的手的深度信息来作为负样本。
    - 较近的活体非活体人脸数据，深度视差信息会比较明显，采集较近距离的活体非活体数据，只挑选出深度信息明显的活体，对于非活体可以保持正常扩充。
    - 较远的活体非活体人脸数据，深度视差信息会比较差，可以扩充人脸检测框，其中活体依靠人体身体信息，脑袋跟肩膀的深度会突出，非活体人脸向外扩充后可能存在手的深度信息，也可以作为负样本。
    - 所以训练集中不能存在深度信息不明显且没有身体信息的活体数据，这样做是为了区分近处的非活体数据与没有身体信息且深度信息不明显的活体的相似性。
    - 综上所述，可以提供一个测试版本，直接对所有活体非活体数据，将扩充框直接包含活体及非活体的身体信息，这样可以规避上述描述的相似性。

- 流程优化：
    - 准备LB(左亮帧)、RB(右亮帧)、LD(左暗帧)、RD(右暗帧) 、DL(LB-LD左帧差)、DR(RB-RD右帧差)数据。
    - 对LB、RB、DL、DR进行亮度检测，检测值分别为bl，br，bdl，bdr。
    - 确定bdl/bl>K,bdr/br>K,确定比值是否满足，若满足，使用DL进行帧差活体检测，同时，使用DL、DR做视差活体检测。若未满足，使用LB，RB做视差活体检测。


## TOF活体
- 位置：/home/xjyu/lgx/Live/TOF
- 图片大小：(96,96,1) Txx-8bit模型
- 环境：conda activate disp




活体检测   版本 0.0.1
所需模型3个:
           <1>人脸检测模型,faceDet;
		   <2>帧差活体检测，anti_spoofing_diff;
		   <3>视差活体检测，anti_spoofing_disp；	

<1>人脸检测模型，训练相关信息
  1>训练代码:
    训练地址：192.168.1.211:/home/jjcai/PROJECTS_2021/T40/FHDET/T40_FHDet
    训练环境：
            python           : 3.8.5
            pytorch          : 1.6.0
            cuda             : 9.0
            MagikTrainingKit : None
            MagikTransKit    : INFO(magik): magik version:1.1.0(00010100_fdf41d4)  built:20210402-1548
    NOTE:
         1.量化插件：包换在conda_envs 环境里。
         2.模型存放路径：weights/202101271937_sgd_FHD_4bit_fineturning_bigface_bg_v2/weights/last_59.pt
         3.转换脚本：convert_tools
         4.相关超参数和配置文件：cfg/FHDet_4bit.cfg,adam,lr 0.00001
         5.数据增强使用mosic + 添加background+约束box裁剪。
         6.查看训练路径下的README了解更信息的信息。
   2>训练样本数据：
     数据地址：/home/jjcai/PROJECTS_2021/T40/FHDET/T40_FHDet_DATA
     上传服务器地址：jjcai_bj@192.168.1.218:/algdata/alg_bj/jjcai_bj/T40_FHDet_DATA.git
     commit 98167a6b0efc86aa16054697fccaff1649d393f0
     NOTE:
         1.人脸人头数据集，包含训练集测试集。
         2.详细使用情况见git仓库下的README.

<2>帧差活体检测,训练相关信息
  1>训练代码:
    训练地址： 192.168.1.217:/home/xjyu/lgx/Live/displive1/diff_model
    训练环境：
            python           : 3.6.12
            tensorflow       : 1.15.0
            cuda             : 10.0.130
            MagikTrainingKit : None
            MagikTransKit    : INFO(magik): magik-transform-tools version:1.0.0(00010000_14382c1)  cuda version:9.0.176  built:20211213-1826_GPU
    上传服务器地址：gxli_bj@192.168.1.218:/algdata/alg_bj/gxli_bj/face_diff.git |branch:master
    commit: 
    NOTE:
      1.量化插件：ingenic_magik_trainingkit文件夹
      2.转换工具及脚本：board_test_train4文件夹
      3.相关参数、数据预处理及模型配置文件：prepare_train文件夹
      4.run案例模型：sample文件夹

  2>训练样本数据:
    数据地址：192.168.1.217:/home/xjyu/lgx/Live/displive1/diff_model/data
    上传服务器地址：gxli_bj@192.168.1.218:/algdata/alg_bj/gxli_bj/face_diff_data.git |branch:master
    commit: 
    NOTE:
      1.包含室内不同距离的活体、非活体、非活体卷起来帧差图
      2.包含室外不同距离的阴天活体、非活体帧差图，其中室外帧差数据的亮度均值大于10
      以上样本数据均为流程测试存图而来


<3>视差活体检测,训练相关信息
  1>训练代码:
    训练地址： 192.168.1.217:/home/xjyu/lgx/Live/displive1/disp_model
    训练环境：
            python           : 3.6.12
            tensorflow       : 1.15.0
            cuda             : 10.0.130
            MagikTrainingKit : None
            MagikTransKit    : INFO(magik): magik-transform-tools version:1.0.0(00010000_14382c1)  cuda version:9.0.176  built:20211213-1826_GPU
    上传服务器地址：gxli_bj@192.168.1.218:/algdata/alg_bj/gxli_bj/face_disp.git |branch:master
    commit: 
    NOTE:
      1.量化插件：ingenic_magik_trainingkit文件夹
      2.转换工具及脚本：board_test_train12_1文件夹
      3.相关参数、数据预处理及模型配置文件：prepare_train文件夹
      4.run案例模型：sample文件夹

  2>训练样本数据:
    数据地址：192.168.1.217:/home/xjyu/lgx/Live/displive1/disp_model/data
    上传服务器地址：gxli_bj@192.168.1.218:/algdata/alg_bj/gxli_bj/face_disp_data.git |branch:master
    commit: 
    NOTE:
      1.包含两种板子求解视差时使用标定参数的活体及非活体视差图  
      2.包含室内不同距离的活体、非活体、非活体卷起来视差图
      3.室外不同距离的阴天、太阳光下活体、非活体视差图
      以上样本数据均为流程测试存图而来


<4>帧差视差测试完整流程,相关信息
  1>代码:
    代码地址： 192.168.1.217:/home/xjyu/lgx/Live/live_det/diff_disp_360x640
    测试环境：
            mxu           : libmxu_imgproc.so,libmxu_core.so,libmxu_calib3d.so,libmxu_features2d.so
            jzdl          : libjzdl.so
            重要流程代码    : dual_infrared_liveness.cpp,face_det_inference.cpp,anti_spoofing_diff.cpp,anti_spoofing_disp.cpp
    上传服务器地址：gxli_bj@192.168.1.218:/algdata/alg_bj/gxli_bj/diff_disp.git |branch:master
    commit: 
    NOTE:
      1.所需要的mxu、jzdl库文件: external文件夹
      2.模型文件：magik_model相关的.h文件
      3.读数据demo：demo文件夹
      4.读取存取后的数据脚本：format_conversion文件夹
      5.mxu源码文件：mxu_ximgproc
      5.带有标定版本的视差活体：liveness-disp-sgbm

  2>训练样本数据:
    数据地址：192.168.1.217:/home/xjyu/share/live_det/diff_disp_360x640/data
    上传服务器地址：gxli_bj@192.168.1.218:/algdata/alg_bj/gxli_bj/diff_disp_data.git |branch:master
    commit: 
    NOTE:
    1.包含（360，640）,(640,480)分辨率的活体及非活体原始图片


