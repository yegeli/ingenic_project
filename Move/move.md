头文件说明
    /*
    * 周界信息结构体
    */
    typedef struct{
    IVSPoint *p;    /**< 周界各个顶点信息，不能有线交叉 */
    /* line_det *l; */
    int pcnt;      /**< 周界顶点数量 */
    int fun;       /**<功能：0：热区检测 ;1:宫格检测*/
    uint64_t alarm_last_time;	/**< 持续报警时间 */
    }single_perm_tt;

    /*
    移动区域的宫格信息
    */
    typedef struct{
        IVSPoint det_rects[MAX_DET_NUM];/**< 检测出移动区域的宫格坐标 */
        int area[MAX_DET_NUM];/**< 移动区域的area */
                            /**< 最大宫格num为MAX_DET_NUM 640*/
    }move_g;
        

    /*
    * 高级移动侦测算法的输入结构体
    */   
    typedef struct {
        single_perm_tt perms[IVS_PERM_MAX_ROI];  /**< 周界信息 */
        int permcnt;                            /**< 周界数量 */
        int sense; /**< 高级移动侦测的灵敏度，范围为0-4 */
        int min_h; /**< 高级移动侦测物体的最小高度 */
        int min_w; /**< 高级移动侦测物体的最小宽度 */
        IVSRect* rois; /**< 高级移动侦测待检测的区域信息 */
        int cntRoi; /**< 高级移动侦测待检测区域的数量 */
        int isSkipFrame; /**< 高级移动侦测跳帧处理开关 */
        bool light;/**< 开灯 >  */
        float level;/**0-1,屏幕检测程度  */
        int timeon;/**开灯用时  */
        int timeoff;/**开灯间隔  */
        int isLightRemove; /**< 高级移动侦测光照处理开关 */
        int det_w; /**<宫格最小单元宽度，必须能被分辨率整除*/
        int det_h; /**<宫格最小单元高度，必须能被分辨率整除 */
        IVSFrameInfo frameInfo; /**< 帧信息 */
    }move_param_input_t;


        
    /*
    * 高级移动侦测算法的输出结构体
    */
    typedef struct {
        
    int ret;		/**< 是否检测出移动区域 */
    int count;    /**< 检测出移动区域的数量 */
    IVSRect rects[NUM_OF_RECTS]; /**< 检测出移动区域的信息 */
    int blockSize; /**< 检测出移动区域块大小 */
    IVSPoint *pt; /**< 移动区域左上角点坐标 */

    move_g g_res; /**< fun=1有效，检测出移动区域的宫格信息 */
    int detcount; /**< fun=1有效，检测出移动区域的宫格数量 */

    int is_alarmed[IVS_PERM_MAX_ROI];     /**< 那个周界出现越界 */
    /* IVSRect rects[IVS_PERM_MAX_RECT];     /\**< 越界物体的矩形信息 *\/ */
    int rectcnt;							/**< 越界物体的数量 */

    }move_param_output_t;


功能说明：
    fun=0-热区检测：
        移动物体与周界有交集就报警，求两个区域的交集。
    fun=1-宫格检测：
        ①将屏幕的h,w 按固定长度的det_w,det_h分为几份，如:
            frame_width=640
            frame_height=360
            det_w = 128
            det_h = 72
            宫格数num= frame_width/det_w * frame_height/det_h = 5*5=25
        ②判断每个宫格与移动物体之间的关系，从而统计出移动物体触发的具体某个宫格，并返回其宫格信息。


参数配置：
	move_param_input_t param;
	memset(&param, 0, sizeof(move_param_input_t));
	param.sense = 4;
	param.frameInfo.width = sensor_sub_width;
	param.frameInfo.height = sensor_sub_height;
	param.level=0.7;
	param.timeon = 110; 
	param.timeoff = 0;  
	param.light = 0;    
	param.isSkipFrame= 0;
	/* param.det_h = 90; */
	/* param.det_w = 160; */
	param.det_h = 36;  //宫格宽高需要被分辨率整除 
	param.det_w = 64;

	param.permcnt = 1;
	param.perms[0].fun = 0;
	/* param.perms[0].fun = 1; */
	param.perms[0].pcnt=4;
    
    //周界需要以顶点顺时针围成的区域
	param.perms[0].p = (int *)malloc(12* sizeof(int));
	param.perms[0].p[0].x=0;
	param.perms[0].p[0].y=0;
	param.perms[0].p[1].x=160;
	param.perms[0].p[1].y=0;
	param.perms[0].p[2].x=160;
	param.perms[0].p[2].y=300;
	param.perms[0].p[3].x=0;
	param.perms[0].p[3].y=300;
        
	*interface = MoveInterfaceInit(&param);



结果使用：
    int ret;		/**< 是否检测出移动区域 */
    int count;    /**< 检测出移动区域的数量 */
    IVSRect rects[NUM_OF_RECTS]; /**< 检测出移动区域的信息 */
    int blockSize; /**< 检测出移动区域块大小 */
    IVSPoint *pt; /**< 移动区域左上角点坐标 */

    move_g g_res; /**< 检测出移动区域的宫格信息 */
    int detcount; /**< 检测出移动区域的宫格数量 */

    int is_alarmed[IVS_PERM_MAX_ROI];     /**< 那个周界出现越界 */
    /* IVSRect rects[IVS_PERM_MAX_RECT];     /\**< 越界物体的矩形信息 *\/ */
    int rectcnt;							/**< 越界物体的数量 */

    if(result->rectcnt > 0){
        for (i=0; i < result->rectcnt;i++){
            x0 = result->rects[i].ul.x;
            y0 = result->rects[i].ul.y;
            x1 = result->rects[i].br.x;
            y1 = result->rects[i].br.y;
            printf("%d %d %d %d",x0,y0,x1,y1);
        }
    }       