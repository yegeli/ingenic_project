
# 工作事项

## 登录

- citrix-html: 192.168.1.59/Citrix/XenApp/auth/login.aspx
- Citrix：gxli   password：Gxli00001    st
- mail.ingenic.com ---->guangxu.li ---->Liguangxu_15
- rdm.ingenic.com ---->guangxu.li ----->Jz1234567
- ssh -X xjyu@192.168.1.217    ---> xjyu_217
- scp -r xjyu@192.168.1.217:lgx/ /home/user/yege
- sftp://user_name@server_ip

## 常用服务器

- user@192.168.1.194:~/togx/        password: user

## 常用命令

- 编译器查看：/opt/mips-gccxxxxxxx/bin/

- PATH环境变量：
  - export PATH= xxxxxx路径:$PATH      向前添加           
  - export PATH= $PATH:xxxxxx路径      向后添加
  
- 文件改名：
  - rename "s/替换/替换内容/g" + 目标
  - rename "s/left/right/g" *.jpg
  
- 查找文件中关键内容：
  - grep "xxxxx" -nr *    （递归查找）
  
- 设置共享目录：

  1. cat /etc/exports   (查看共享的目录)
  2. 添加共享目录
  3. sudo /etc/init.d/nfs-kernel-server stop/start
  4. service nfs-kernel-server restart
  5. service --status-all


- mount 地址：

  - mount -t nfs -o nolock 192.168.1.217:/home/xjyu/share/ /mnt/img/

- 查看CUDA版本：
  - nvcc -V
  - cat /usr/local/cuda/version.txt

- 查看pip路径： 
  - which pip
- pip豆瓣源：
  - pip install xxx -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com

## 注意事项

- 移动周界中，在执行carrier-server时，只能使用4.7.2编译链接库运行。
- TOF 对应的conda环境为disp
- AutoShape对应的conda环境为paddle_latest
- 人脸检测流程conda环境360
- move移动宫格sample测试：
  - 1. fs_chn_attr.crop.enable = 0
  - 2. Makefile中使用对应板子的glibc或uclibc库：CONFIG_UCLIBC_BUILD=y/n
  - 3. sample_common.c宏定义对应的sensor：#define SENSOR_SC2235

- sense灵敏度逻辑：
  - thresh：做二值化的值，默认为20，大于20为255/1,小于20为0 
  - sense = 2时:
      NUM_OF_HISTORY[0] = 20;
      NUM_OF_HISTORY[1] = 10;
      NOISE_THRESH[0] = 10;
      NOISE_THRESH[1] = 5;
  - sense = 3时：sizeOfSldWin = 2,当前bufs存的是2帧图数据
    thresh = 26,做二值化的值  
  - sense = 4时：NUM_OF_HISTORY[0] = 6，噪声阈值开启时，NUM_OF_HISTORY[0]=3，未开启NUM_OF_HISTORY[0]=6
    sizeOfSldWin = 4,当前bufs存的是4帧图数据
  - 流程：
      Mat buf_idx1 = bufs[sizeOfSldWin-1]; 
      Mat buf_idx2 = bufs[0]; 
      absdiff( buf_idx1, buf_idx2, silh ); // 将存的最后一帧图与第一帧图做帧差，存到Mat::silh中，sizeofSldWin越大虚影越多，因其是一个运动的物体。
      threshold( silh, silh, thresh, 1, MXU_THRESH_BINARY ); // 将帧差后的图做二值化，0和1两种状态，映射到opencv上为0和255，更好的直观查看。
      灵敏度为2时：先做腐蚀后做膨胀-开运算：具有消除细小物体，在纤细处分离物体和平滑较大物体边界的作用。
        erode(silh, silh, Mat(), Point(-1,-1), 1);
        dilate(silh, silh, Mat(), Point(-1,-1), 2);
        <!-- mergemove(buf_idx1, buf_idx2, silh, thresh, 1, MXU_THRESH_BINARY, MXU_OPEN_MIN, Mat(),Point(-1,-1)); -->
      灵敏度大于2时：先做膨胀后做腐蚀-闭运算：具有田中物体内细小空洞，连接邻近物体和平滑边界的作用。
        dilate(silh, silh, Mat(), Point(-1,-1), 2);
        erode(silh, silh, Mat(), Point(-1,-1), 1);
        <!-- mergemove(buf_idx1, buf_idx2, silh, thresh, 1, MXU_THRESH_BINARY, MXU_CLOSE_MAX, Mat(),Point(-1,-1)); -->
      灵敏度小于2时：先做腐蚀后做膨胀-开运算：具有消除细小物体，在纤细处分离物体和平滑较大物体边界的作用。
        erode(silh, silh, Mat(), Point(-1,-1), 1);
        dilate(silh, silh, Mat(), Point(-1,-1), 2);
        <!-- mergemove(buf_idx1, buf_idx2, silh, thresh, 1, MXU_THRESH_BINARY, MXU_OPEN_MIN, Mat(),Point(-1,-1)); -->
      将开闭运算结果绘制成轮廓：
        contours(silh,contours_mxu,MXU_RETR_EXTERNAL, MXU_CHAIN_APPROX_SIMPLE, Point(0,0));
      计算轮廓的垂直边界最小矩形，矩形是与图像上下边界平行的:
        Rect rect = boundRect(contours_mxu[i]); 

- PC端move移动验证地址：
  ssh -X jjcai@192.168.1.195   jjcai_195
  /home/jjcai/WORK/SSD/PROJECTS_2021/MSTracker/MSTracker/ivs_pc_move_lgx