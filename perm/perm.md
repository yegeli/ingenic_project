头文件说明：
	/*
	* 周界信息结构体
	*/
	typedef struct{
	IVSPoint *p;    /**< 周界各个顶点信息，不能有线交叉，最多6个点 */
	int pcnt;      /**< 周界顶点数量 */
	int fun;       /**<功能：0：周界检测；1：热区检测;2：判断移动方向 */
	uint64_t alarm_last_time;	/**< 持续报警时间 */
	}single_perm_t;


	/*
	移动框的状态
	*/
	typedef struct{
		int status; /**<移动框的状态 -1：从内到外    0：方向不确定   1：从外到内  */
		int perm_index; /**与具体哪个周界相交 */

	}rect_status_t;

		
	/*
	* 周界防范算法的输入结构体
	*/
	typedef struct{
	single_perm_t perms[IVS_PERM_MAX_ROI];  /**< 周界信息 */
	int permcnt;                            /**< 周界数量 */
	int sense;							  /**< 灵敏度 */
	bool isSkipFrame;    /**< 跳帧 >  */

	bool light;/**< 开灯 >  */
	float level;/**0-1,屏幕检测程度  */
	int timeon;/**开灯用时  */
	int timeoff;/**开灯间隔  */
	
	IVSFrameInfo frameInfo;				  /**< 帧信息 */
	}perm_param_input_t;

	/*
	* 周界防范算法的输出结构体
	*/
	typedef struct{
	int ret;								/**< 是否出现越界 */
	int is_alarmed[IVS_PERM_MAX_ROI];     /**< 那个周界出现越界 */
	IVSRect rects[IVS_PERM_MAX_RECT];     /**< 越界物体的矩形信息 */
	int rectcnt;							/**< 越界物体的数量 */
	rect_status_t rect_status[IVS_PERM_MAX_RECT]; /**<移动框的状态 fun=2生效 */
	int state[IVS_PERM_MAX_ROI]; 			/**< fun=0/1生效，state=0表示在周界内部，state=1表示在周界外部，state=2表示与周界相交 */
	}perm_param_output_t;



功能说明：
	fun=0-周界检测：
		①移动物体顶点距离小于最短边10%报警   （在周界内部）
		②移动物体与周界相交报警             （与周界相交）
		③移动物体与周界距离小于20报警		 （在周界外部）

	fun=1-热区检测：
		移动物体与周界有交集就报警，求两个区域的交集。

	fun=2-移动方向检测：
		前提是移动物体与周界是相交的，通过其历史框与周界的相交面积变化，来判断其方向。
		移动框的状态结果 -1：从周界内到外    0：方向不确定   1：从周界外到内  */



参数配置：
    int ret = 0;
	perm_param_input_t param;
	memset(&param, 0, sizeof(perm_param_input_t));
	param.sense = 2; # 0~4，灵敏度功能设计复杂，主要作用是越大越灵敏，越大去噪效果越好，如果要调试远距离，可以将灵敏度降低
	param.frameInfo.width = sensor_sub_width;
	param.frameInfo.height = sensor_sub_height;
	param.isSkipFrame=false;
	param.permcnt=1; # 一个周界
	param.perms[0].fun=1; # 使用fun为1的功能
	param.perms[0].pcnt=4; # 周界顶点4个

    # fun=0时，使用的是周界检测，有移动物体在周界内外靠近边界、与边界相交时报警
    # fun=1时，无论是在周界内还是外，移动物体区域与周界区域有交集就报警
    # fun=2，是在移动物体与周界边界相交时的情况下，返回3个状态结果：移动物体是从周界内到外，还是外到内，还是无法判断不确定。
    # 具体返回值请查看头文件中输出参数

	param.light = false; # 开关灯功能，打开开关灯，会过滤掉一些占屏较大的移动帧
	param.level = 0.6;   # 占屏比例
	param.timeon = 110;  # 开灯时长，单位帧
	param.timeoff =310;  # 下次开灯间隔，具体描述：打开开关灯，开灯110帧会过滤占屏较大移动帧，这些移动不进入算法判断，110帧后，所有移动正常计算，间隔310帧后，自动开启开灯模式。关闭开关灯，所有移动正常计算。
	 	
    # 周界是由顶点坐标坐标顺时针围城的区域
	param.perms[0].p = (int *)malloc(12* sizeof(int));
	param.perms[0].p[0].x=36;
	param.perms[0].p[0].y=16;
	param.perms[0].p[1].x=304;
	param.perms[0].p[1].y=16;
	param.perms[0].p[2].x=304;
	param.perms[0].p[2].y=216;
	param.perms[0].p[3].x=36;
	param.perms[0].p[3].y=216;
	
	*interface = PermInterfaceInit(&param);




结果使用：
	int ret;								/**< 是否出现越界 */
	int is_alarmed[IVS_PERM_MAX_ROI];     /**< 那个周界出现越界，报警状态 */
	IVSRect rects[IVS_PERM_MAX_RECT];     /**< 越界物体的矩形信息 */
	int rectcnt;							/**< 越界物体的数量 */
	rect_status_t rect_status[IVS_PERM_MAX_RECT]; /**<移动框的状态 fun=2生效 */
	int state[IVS_PERM_MAX_ROI]; 			/**< fun=0/2生效，state=0表示在周界内部，state=1表示在周界外部，state=2表示与周界相交 */

	if(result->ret>0){
		for(j =0;j < result->rectcnt;j++){
			IVSRect* show_rect=&result->rects[j];
		}	
	}

    