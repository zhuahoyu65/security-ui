<template>
    <!--html,不用head和body-->
    <div class="dataview_main_box">
        <div class="top_information">
            <div class="index_topbox">
                <div class="index_time" >{{show_date}}</div>
                <div class="index_snap index_snap1">抓拍    <span>{{ snapCount }}</span>   次</div>
                <div class="index_snap">报警    <span>{{ warningCount }}</span>   次</div>
                <div class="index_personnel" v-if="person_total > 400000">底库人员总量    <span>{{ person_total + 250000 }}</span>   张</div>
                <div class="index_personnel" v-else>底库人员总量    <span>{{ person_total }}</span>   张</div>
            </div>
        </div>
        <div class="index_bottombox">
            <div class="min_nav">
                <div class="index_navbox">
                    <div class="index_nav1"   @click="change_active(0)"  @mouseenter="enter(0)" @mouseleave="leave">
                        <div class="index_text1" :class="{ 'index_active': isactive1}">
                            <div class="index_table">
                                <div class="index_cell">
                                    流量统计
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="index_nav1"  @click="change_active(1)"  @mouseenter="enter(1)" @mouseleave="leave">
                        <div class="index_text1" :class="{ 'index_active': isactive2}">
                            <div class="index_table">
                                <div class="index_cell">
                                    人群分析
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="index_nav1"  @click="change_active(2)"  @mouseenter="enter(2)" @mouseleave="leave">
                        <div class="index_text1" :class="{ 'index_active': isactive3}">
                            <div class="index_table">
                                <div class="index_cell">
                                    设备性能
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="active_div" :style="{ left: 192*listnum + 'px' }"></div>
                </div>
                <div class="index_rightbtn">
                    <div class="data_timebox" v-show="is_show_date">
                        <el-date-picker
                          v-model="dateValue"
                          type="daterange"
                          align="right"
                          unlink-panels
                          range-separator="至"
                          start-placeholder="开始日期"
                          end-placeholder="结束日期"
                          :picker-options="pickerOptions">
                        </el-date-picker>
                    </div>
                    <div class="index_cam" v-show="$route.path != '/dataview3' " :title="info_cam_data"> {{info_cam_data}} </div>
                    <!-- <div class="index_btn" v-show="false">数据生成</div> -->
                    <div class="index_btn" @click="is_show_info = true" v-show="is_show_choose">设备选择</div>
                </div>
            </div>
            <transition :name="transitionName">
                <!-- <keep-alive> -->
                    <router-view :searchData="search_data" :dateValue="dateValue"></router-view>
                <!-- </keep-alive> -->
            </transition>
        </div>
        <!--弹出框-->
        <div class="mack_box" v-show="is_show_info" @click="clear_search_data"></div>
        <div class="alert_Box" id="alert_Box" v-show="is_show_info">
            <div class="ale_leftbox">
                <div class="ale_topinput">
                    <div class="ale_input">
                        <select v-model="choose_groupName">
                            <option v-for="item in groupNames">{{item.name}}</option>
                        </select>
                    </div>
                    <div class="ale_input">
                        <input type="text" placeholder="输入名称、ID" v-model="cameraName"/>
                        <!-- <div class="search_box"></div> -->
                    </div>
                </div>
                <div class="ale_leftlist" v-if="choose_groupName!='设备组选择'">
                    <div class="ale_list" v-for="item in info_show_data" v-show="item.isshow" @click="add_search_data(item,'camera')" >
                        <div class="ale_text" :title="item.name">{{item.name}}</div>
                        <div class="ale_icon"></div>
                    </div>
                </div>
                <div class="ale_leftlist" v-if="choose_groupName==='设备组选择'">
                    <div class="ale_list"  
                         v-for="item in groupNames" 
                         v-show="item.isshow"
                         @click="add_search_data(item,'group')"
                    >
                        <div class="ale_text">{{item.name}}</div>
                        <div class="ale_icon"></div>
                    </div>
                </div>
            </div>
            <div class="ale_rightbox">
                <div class="right_table">
                    <table>
                        <tr>
                            <td class="ale_td1_1">编号</td>
                            <td class="ale_td2_1">名称</td>
                            <td class="ale_td3_1">移除</td>
                        </tr>
                        <tr v-for="item,index in info_search_data" :title="item.groupName+'\n'+item.name" v-if="choose_groupName!='设备组选择'">
                            <td class="ale_td1">{{index+1}}</td>
                            <td class="ale_td2">{{item.name}}</td>
                            <td class="ale_td3">
                                <div class="ale_delete" @click="delete_search_data(index,'camera')"></div>
                            </td>
                        </tr>
                        <tr v-for="item,index in info_search_data_group" v-if="choose_groupName==='设备组选择'">
                            <td class="ale_td1">{{index+1}}</td>
                            <td class="ale_td2">{{item.name}}</td>
                            <td class="ale_td3">
                                <div class="ale_delete" @click="delete_search_data(index,'group')"></div>
                            </td>
                        </tr>
                    </table>
                </div>
            </div>
            <div class="se_cancel" @click="is_show_info = false" title="关闭"></div>
            <div class="se_confirm" @click="confirm_search" title="确认" v-if="confirm_button_flag"></div>
            <div class="se_confirm" title="请求中..." style="cursor: not-allowed;" v-else></div>
        </div>
    </div>
</template>

<script> 
    export default {
        data() {
                return {
                    // 页面切换
                    transitionName: '',

                    // 顶栏数据显示
                    snapCount: 0,
                    warningCount: 0,
                    person_total: 0,

                    items: [],

                    // 弹窗加切换栏
                    listnum : 1,
                    isactive1:false,
                    isactive2:true,
                    isactive3:false,
                    active_num: 1,
                    index_span1:'9513651',
                    index_span2:'5',
                    index_span3:'1000000',
                    is_show_info: false,
                    is_show_date: false,
                    is_show_choose: true,

                    // 弹窗
                    cameraName: "",
                    choose_groupName: null,
                    info_cam_data: "",
                    info_show_data: [],
                    info_search_data: [],
                    info_search_data_group: [],
                    confirm_button_flag: true,

                    // 预警栏
                    show_date : null,

                    // 请求
                    groupNames: [],
                    video_names: [],
                    cameraSdkIds: "",
                    cameraGroupIds: "",
                    dateValue: [(new Date() - 3600 * 1000 * 24 * 7),new Date()-1],
                    pickerOptions: {
                      shortcuts: [{
                        text: '最近三天',
                        onClick(picker) {
                          const end = new Date();
                          const start = new Date();
                          start.setTime(start.getTime() - 3600 * 1000 * 24 * 3);
                          picker.$emit('pick', [start, end]);
                        }
                      },{
                        text: '最近一周',
                        onClick(picker) {
                          const end = new Date();
                          const start = new Date();
                          start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
                          picker.$emit('pick', [start, end]);
                        }
                      }, {
                        text: '最近一个月',
                        onClick(picker) {
                          const end = new Date();
                          const start = new Date();
                          start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
                          picker.$emit('pick', [start, end]);
                        }
                      }, {
                        text: '最近三个月',
                        onClick(picker) {
                          const end = new Date();
                          const start = new Date();
                          start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
                          picker.$emit('pick', [start, end]);
                        }
                      }]
                    },

                    // 子组件传递参数
                    search_data: {},
                    
                    name: [],
                    day_time: [],
                    showData_hour: [],
                    showData_day: [],
                    // 更改标志
                    update_flag: false,

                    // 定时器
                    timer_num: null,
                }
            },
            mounted() {
                this.show_date = this.real_time()
                this.get_snapCounting()
                this.get_mmanage_people_num()
                setInterval(() => {
                    this.show_date = this.real_time()
                }, 1000);
                this.timer_num = setInterval(() => {
                    this.get_snapCounting()
                }, 5000);

                this.get_init_data()
                // this.search_data = {}

                this.change_mynav_active()
            },
            methods: {
                // test:function(){
                //     // console.log("hah")
                //     this.day_time = ['08-11','08-12','08-13','08-14','08-15','08-16','08-17']
                // },

                real_time:function(){
                    let date = new Date();
                    let month_data = "",day_data = ""
                    let Hdata = "",Mdata = "",Sdata = ""
                    if( date.getMonth() + 1 < 10 ){
                        month_data = "0" + (date.getMonth() + 1)
                    }else{
                        month_data = (date.getMonth() + 1)
                    }
                    if( date.getDate() < 10){
                        day_data = "0" + date.getDate()
                    }else{
                        day_data = date.getDate()
                    }
                    if( date.getHours() < 10){
                        Hdata = "0" + date.getHours()
                    }else{
                        Hdata = date.getHours()
                    }
                    if( date.getMinutes() < 10 ){
                        Mdata = "0" + date.getMinutes()
                    }else{
                        Mdata = date.getMinutes()
                    }
                    if( date.getSeconds() < 10 ){
                        Sdata = "0" + date.getSeconds()
                    }else{
                        Sdata = date.getSeconds()
                    }
                    return date.getFullYear() + '年' + month_data + '月' + day_data + '日    ' + Hdata + '点' + Mdata + '分' + Sdata + '秒';
                },

                // 弹窗加切换栏
                change_active:function(num){
                    for(let i = 1; i < 4; i++){
                        this["isactive"+i] = false//清除所有此导航栏名为“isactive”的Class
                        if ( i === num + 1){
                            this["isactive"+i] = true//为当前div添加Class"isactive"
                            this.listnum = num //将参数num赋值给listnum
                            this.active_num=num //保存点击选中的num值
                        }
                    }

                    if( num === 0 ){
                        this.$router.push('/dataview1')
                    }else if( num === 1 ){
                        this.$router.push('/dataview2')
                    }else if( num === 2 ){
                        this.$router.push('/dataview3')
                    }
                },
                enter:function(num){//鼠标移入时调用该函数
                    this.listnum = num //鼠标移入时给listnum重新赋值
                },
                leave:function(){//鼠标移出时调用该函数
                    this.listnum = this.active_num //鼠标移出时恢复之前点击选中的num值
                },

                // 弹窗
                // 修改设备组，显示数据变更
                change_show_data:function(num,model="video"){
                    if( model === "group" ){
                        for(let i = 1; i < this.groupNames.length; i++){
                            if( this.groupNames[i].name.indexOf( this.cameraName ) === -1 ){
                                this.groupNames[i].isshow = false
                            }else{
                                this.groupNames[i].isshow = true
                            }
                        }
                    }else{
                        // console.log(num-1,model)
                        this.info_show_data = []
                        for( let i = 0; i < this.video_names[num-1].length; i++ ){
                            if( this.video_names[num-1][i].name.indexOf( this.cameraName ) === -1 ){
                                this.video_names[num-1][i].isshow = false
                            }else{
                                this.video_names[num-1][i].isshow = true
                            }
                            this.info_show_data.splice(-1,0,this.video_names[num-1][i])
                        }
                    }
                },
                add_search_data:function(data,model){
                    if( model === "camera" ){
                        if( this.info_search_data.length < 5 ){
                            // 判断是否已选择设备
                            for( let i = 0; i < this.info_search_data.length; i++ ){
                                if( data.sdkId === this.info_search_data[i].sdkId ){
                                    this.warning_info("选择设备已在列表中")
                                    return
                                }
                            }
                            
                            this.info_search_data.push( 
                                {
                                    name: data.name,
                                    sdkId: data.sdkId,
                                    groupName: data.groupName,
                                } 
                            )
                        }else{
                            this.warning_info("最多选择五个设备")
                        }
                    }else if( model === "group" ){
                        if( this.info_search_data_group.length < 5 ){
                            // 判断是否已选择设备
                            for( let i = 0; i < this.info_search_data_group.length; i++ ){
                                if( data.uuid === this.info_search_data_group[i].sdkId ){
                                    this.warning_info("选择设备组已在列表中")
                                    return
                                }
                            }
                            
                            this.info_search_data_group.push( 
                                {
                                    name: data.name,
                                    sdkId: data.uuid,
                                } 
                            )
                        }else{
                            this.warning_info("最多选择五个设备组")
                        }
                    }
                },
                // 将设备从搜索中删除
                delete_search_data:function(index,model){
                    if( model === "camera" ){
                        this.info_search_data.splice(index,1)
                    }else{
                        this.info_search_data_group.splice(index,1)
                    }
                    
                },
                // 确认请求信息
                confirm_search:function(){
                    if( this.choose_groupName != "设备组选择" ){
                        let search_data = {
                            cameraSdkIds: [],
                        }
                        let temp_data = []
                        for( let i = 0; i < this.info_search_data.length; i++ ){
                            search_data.cameraSdkIds.push(this.info_search_data[i].sdkId)
                            temp_data.push(this.info_search_data[i].name)
                        }
                        this.info_cam_data = temp_data.join("/")
                        if( search_data.cameraSdkIds.length ){
                            this.search_data = search_data
                        }else{
                            this.warning_info("未选择设备")
                        }
                    }else{
                        let search_data = {
                            cameraGroupIds: [],
                        }
                        let temp_data = []
                        for( let i = 0; i < this.info_search_data_group.length; i++ ){
                            search_data.cameraGroupIds.push(this.info_search_data_group[i].sdkId)
                           temp_data.push(this.info_search_data_group[i].name)
                        }
                        this.info_cam_data = temp_data.join("/")
                        if( search_data.cameraGroupIds.length ){
                            this.search_data = search_data
                        }else{
                            this.warning_info("未选择设备组")
                        }
                    }
                },

                // 请求数据
                // mes_handling:function(status, msg){
                //     if( status === 1 ){
                //         this.error_info(msg)
                //         return ;
                //     }else if( status === 2 ){
                //         this.error_info(msg)
                //         return ;
                //     }else if( status === 10 ){
                //         this.error_info('请先登录')
                //         return ;
                //     }else{
                //         if( status === 401 && msg === "未登录" ){
                //             this.error_info(msg)
                //             this.$router.push("/login")
                //         }else{
                //             this.error_info(status + "  " + msg)
                //         }
                //     }
                // },
                get_init_data:function(){
                    // 请求设备组列表
                    var params = new URLSearchParams()
                    this.$ajax.post("/groupCamera/getAllCameras",params).then((res) => {
                        if( res.data.status === 0){
                            for( let item in res.data.data ){
                                // console.log(item)
                                let [name,uuid] = item.split(",")
                                this.groupNames.push( {"name":name,"uuid":uuid,"isshow":true} )
                                this.video_names.push( res.data.data[item] )
                            }
                            for( let i = 0; i < this.video_names.length; i++ ){
                                for( let j = 0; j < this.video_names[i].length; j++ ){
                                    this.video_names[i][j].uuid = j
                                }
                            }
                            this.choose_groupName = this.groupNames[0].name
                            this.groupNames.splice(0,0,{name:"设备组选择"})
                        }else{
                            this.mes_handling(res.data.status,res.data.msg)
                        }
                    }).catch((error) => {
                        console.log(error)
                        this.error_info('网络连接出错')
                        return ;
                    })
                },
                get_snapCounting:function(){
                    var params = new URLSearchParams()
                    this.$ajax.post("/data/snapCounting",params).then((res) => {
                        if( res.data.status === 0){
                            if( res.data.data.snapCount ){
                                this.snapCount = res.data.data.snapCount
                            }
                            if( res.data.data.warningCount ){
                                this.warningCount = res.data.data.warningCount
                            }                    
                        }else{
                            this.mes_handling(res.data.status,res.data.msg)
                        }
                    }).catch((error) => {
                        console.log(error)
                        // this.error_info('网络连接出错')
                        return ;
                    })
                },
                get_mmanage_people_num:function(){
                    // 请求人员数据
                    this.$ajax.post("/person/list").then((res) => {
                        if( res.data.status === 0){
                            this.person_total = res.data.data.total
                        }else{
                            this.mes_handling(res.data.status,res.data.msg)
                        }
                    }).catch((error) => {
                        console.log(error)
                        // this.error_info('网络连接出错')
                        return ;
                    })
                },
                // 消息窗口
                error_info:function(mes){
                    this.$message({
                        type: 'error',
                        message: mes,
                        showClose: true,
                        center: true
                    })
                },
                warning_info:function(mes){
                    this.$message({
                        type: 'warning',
                        message: mes,
                        showClose: true,
                        center: true
                    })
                },
                success_info:function(mes){
                    this.$message({
                        type: 'success',
                        message: mes,
                        showClose: true,
                        center: true
                    })
                },
                
                // 弹窗关闭清除
                clear_search_data:function(){
                    // this.info_search_data = []
                    this.is_show_info = false
                },

                // 页面跳转
                change_mynav_active:function(){
                    if( this.$route.path.indexOf("dataview1") === 1 ){
                        this["isactive"+(this.active_num+1)] = false
                        this.isactive1 = true
                        this.listnum = 0
                        this.active_num = 0
                        this.is_show_date = false
                        this.is_show_choose = true
                    }else if( this.$route.path.indexOf("dataview2") === 1 ){
                        this["isactive"+(this.active_num+1)] = false
                        this.isactive2 = true
                        this.listnum = 1
                        this.active_num = 1
                        this.is_show_date = true
                        this.is_show_choose = true
                    }else if( this.$route.path.indexOf("dataview3") === 1 ){
                        this["isactive"+(this.active_num+1)] = false
                        this.isactive3 = true
                        this.listnum = 2
                        this.active_num = 2
                        this.is_show_date = false
                        this.is_show_choose = false
                    }
                }
            },
            watch:{
                $route(to,from){
                    this.transitionName = 'slide-left';
                    this.change_mynav_active()
                },
                'choose_groupName':function(newVal,old){
                    for( let i = 0; i < this.groupNames.length; i++ ){
                        if( this.groupNames[i].name === newVal ){
                            if( this.groupNames[i].name === "设备组选择" ){
                                this.change_show_data(i,"group")
                            }else{
                                this.change_show_data(i,"video")
                            }
                        }
                    }
                },
                'cameraName':function(newVal,old){
                    if( this.choose_groupName === "设备组选择" ){
                        for( let i = 1; i < this.groupNames.length; i++ ){
                            if( this.groupNames[i].name.indexOf( newVal ) === -1 ){
                                this.groupNames[i].isshow = false
                            }else{
                                this.groupNames[i].isshow = true
                            }
                        }
                    }else{
                        for( let i = 0; i < this.info_show_data.length; i++ ){
                            if( this.info_show_data[i].name.indexOf( newVal ) === -1 ){
                                this.info_show_data[i].isshow = false
                            }else{
                                this.info_show_data[i].isshow = true
                            }
                        }
                    }
                },
                '$store.state.dataview_data.update_flag1':function(newVal,old){
                    this.clear_search_data()
                },
                '$store.state.dataview_data.update_flag2':function(newVal,old){
                    this.clear_search_data()
                },
                '$store.state.dataview_data.update_flag3':function(newVal,old){
                    this.clear_search_data()
                },
                '$store.state.dataview1_flag':function (newval,old) {
                    if( newval ){
                        this.confirm_button_flag = false
                    }else{
                        this.confirm_button_flag = true
                    }
                },
                '$store.state.dataview2_flag':function (newval,old) {
                    if( newval ){
                        this.confirm_button_flag = false
                    }else{
                        this.confirm_button_flag = true
                    }
                },
            },
            beforeRouteLeave(to, from, next){
                clearInterval(this.timer_num)
                next()
            }
    }
</script>

<style>
    .main_ch{
        width: 100%;
        height: calc( 100% - 60px );
        background-color: rgba(0,0,0,0.4);
    }
    .main_text{
        width: 100%;
        height: 100%;
        font-size: 25px;
        color: #03BD71;
        text-align:center;
    }
    .main_loading {
        width: 100%;
        height: 100%;
    }
    .main_table{
        display: table;
        width: 100%;
        height: 100%;
    }
    .main_cell{
        display: table-cell;
        vertical-align: middle;
        width: 100%;
        height: 100%;
    }
    .spinner {
        margin: auto;
        width: 50px;
        height: 60px;
        text-align: center;
        font-size: 10px;
    }

    .spinner>div {
        background-color: #03BD71;
        height: 100%;
        width: 6px;
        display: inline-block;
        -webkit-animation: stretchdelay 1.2s infinite ease-in-out;
        animation: stretchdelay 1.2s infinite ease-in-out;
    }

    .spinner .rect2 {
        -webkit-animation-delay: -1.1s;
        animation-delay: -1.1s;
    }

    .spinner .rect3 {
        -webkit-animation-delay: -1.0s;
        animation-delay: -1.0s;
    }

    .spinner .rect4 {
        -webkit-animation-delay: -0.9s;
        animation-delay: -0.9s;
    }

    .spinner .rect5 {
        -webkit-animation-delay: -0.8s;
        animation-delay: -0.8s;
    }

    @-webkit-keyframes stretchdelay {
        0%,
        40%,
        100% {
            -webkit-transform: scaleY(0.4)
        }
        20% {
            -webkit-transform: scaleY(1.0)
        }
    }

    @keyframes stretchdelay {
        0%,
        40%,
        100% {
            transform: scaleY(0.4);
            -webkit-transform: scaleY(0.4);
        }
        20% {
            transform: scaleY(1.0);
            -webkit-transform: scaleY(1.0);
        }
    }
</style>
<style scoped>
    @import "../css/historyface.css";
    
    .dataview_main_box{
        width: 100%;
        height: 100%;
        position: absolute;
    }

    /*界面切换样式*/
    .slide-right-enter-active,
    .slide-right-leave-active,
    .slide-left-enter-active{
        will-change: transform;
        transition: all 1000ms ease;
        /*position: absolute;*/
        /*float: left;*/
    }
    .slide-left-leave-active {
        will-change: transform;
        transition: all 1000ms ease;
        /*position: absolute;*/
        float: left;
    }
    .slide-right-enter {
        opacity: 0;
        transform: translate3d(-100%, 0, 0);
    }
    .slide-right-leave-active {
        opacity: 0;
        transform: translate3d(100%, 0, 0);
    }
    .slide-left-enter {
        opacity: 0;
        /*-webkit-transform: translate3d(100%,0, 0);*/
        transform: translate3d(100%, 0, 0);
    }
    .slide-left-leave-active {
        opacity: 0;
        /*-webkit-transform: translate3d(-100%,0, 0);*/
        transform: translate3d(-100%, 0, 0);
    }
</style>
