<template>
    <div style="width:100%;height:100%;">
        <div class="main_box">
            <left-nav></left-nav>
            <div class="map_bg" id="container"></div>
            <!--地图边框线-->
            <div class="left_xian"></div>
            <div class="top_xian"></div>
            <!--右边上半部分-->
            <div class="face_rightbox1">
                <div class="faceleft_photo">
                    <div class="face_photoimg">
                        <div class="faceshow_img" v-show="dataUrl"><img :src="dataUrl" v-show="dataUrl"></div>
                        <img class="decoration_img" src="../assets/mmanage/add_file.png"  v-show="!dataUrl" />
                        <input class="face_file" type="file" @change="handleFileChange" ref="inputer" />
                    </div>
                    <div class="facephoto_text">检索对象</div>
                </div>
                <div class="face_fillbox">
                    <div class="face_timetext">时间范围</div>
                    <div class="face_timeinput">
                        <el-date-picker 
                          v-model="datevalue"
                          type="datetimerange"
                          :picker-options="pickeroptions"
                          range-separator="至"
                          start-placeholder="开始日期"
                          end-placeholder="结束日期">
                        </el-date-picker>
                    </div>
                    <div class="similar_box">
                        <div class="similar_text">相似度</div>
                        <div class="slider_box">
                            <el-slider v-model="search_data.confidence"></el-slider>
                        </div>
                        <div class="percentage"><input type="text" v-model="search_data.confidence"/></div>
                        <div class="percentage_text fp_percentage">%</div>
                        <div class="search face_search" @click="click_to_search(search_data)">搜索</div>
                    </div>
                    <div class="results_box">
                        <div class="results_text">发现{{init_data.allnum}}个结果</div>
                        <div class="face_rowselect">
                            <div class="rowselect_text">限制</div>
                            <div class="rowselect_select">
                                <select v-model="search_data.identifyNumber">
                                    <option>10</option>
                                    <option>20</option>
                                    <option>30</option>
                                </select>
                            </div>
                        </div>
                        <div class="export_btn face_btn" @click="click_to_clear">清空</div>
                        <div class="export_btn face_btn" @click="export_data2excel">导出</div>
                    </div>
                </div>
            </div>
            <!--右边下半部分-->
            <div class="face_rightbox2">
                <div class="results_listbox">
                    <div class="results_list" :style="{'border-left': item.mystyle}" v-for="item in tabledata">
                        <div class="digital_bg" @click="change_map_center(item.location)">{{init_data.allnum - item.uuid}}</div>
                        <div class="results_right">
                            <div class="results_conter1">{{item.cameraName}}</div>
                            <div class="re_conterbox1">
                                <div class="results_conter2">{{item.nyr}}</div>
                                <div class="results_conter2">{{item.sfm}}</div>
                                <div class="results_conter2">相似度：{{ item.confidence }}</div>
                            </div>
                            <div class="re_conterbox2">
                                <img :src="item.snapshotUrl" @click="show_pic(item.wholePhoto)" title="点击显示原图" />
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!--遮罩层-->
        <div class="mack_box" v-show="is_show_pic" @click="is_show_pic = false"></div>
        <div class="t_graphBox" v-show="is_show_pic" @click="is_show_pic = false">
            <div class="t_graph" >
                <div class="graph_table">
                    <div class="graph_cell">
                        <img style="max-width:800px; max-height:800px;margin:0 auto;" :src="total_pic" />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import LeftNav from "./left_nav2"
    import AMap from 'AMap'
    import Vue from 'vue'

	export default {
		components : {
            LeftNav,
        },
        data(){
            return{
                icon_eye: require("../assets/historyface/icon7.png"),
                icon_setting: require("../assets/historyface/icon2.png"),
                icon_marker: require("../assets/facepath/map_iconimg.png"),
                // 地图数据
                map: null,
                markers : [],
                circleMarker: null,
                polyline : null,
                center_xy: [114.059648,22.543665],
                locations: [],
                infomycontent: [],
                mycontent_marker: [],
                markers_list: [],
                allcamera_list: null,

                // 初始化数据
                init_data:{
                    allnum: 0,
                },

                // 检索数据
                dataUrl: "",
                pic_value: "",
                default_confidence: 70,
                search_data:{
                    photo: "",
                    confidence: 70,
                    startTime: "",
                    endTime: "",
                    identifyNumber: 10
                },
                // input_confidence: 0,
                // same_confidence: 0,
                datevalue: [(new Date() - 3600 * 1000 * 24 * 15),new Date()-1],
                pickeroptions:{
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

                // default_data:{
                //     id: 1,
                //     cameraName: "camera 12##########",
                //     nyr: "2018-08-01",
                //     sfm: "11:11:11",
                //     confidence: 89,
                //     mystyle: "2px solid white"
                // },
                // 初始化数据
                tabledata:[
                   
                ],

                // 原图
                is_show_pic: false,
                total_pic: "",
            }
        },
        methods:{
            // 显示全图
            show_pic:function(imgUrl){
                if( imgUrl ){
                    this.total_pic = imgUrl
                    this.is_show_pic = true
                }else{
                    this.total_pic = ""
                    this.warning_info("未找到原图")
                }
            },

            // 地图初始化
            init:function(location){
                this.map = new AMap.Map('container', {
                  center: location,
                  resizeEnable: true,
                  zoom: 17,
                  mapStyle: 'amap://styles/darkblue',
                })
            },
            change_map_center:function(location){
                if( this.circleMarker ){
                    this.circleMarker.setMap(null)
                }
                // 半径为米，随地图变化
                this.circleMarker = new AMap.Circle({
                    map: this.map,
                    center: location,
                    radius: 10,  //半径
                    strokeColor: "#fefefe",  //线颜色
                    strokeWeight: 1,  //线粗细度
                    fillColor: '#1CC7FF',  //填充颜色
                    fillOpacity: 0.8 ,//填充透明度
                })

                // this.circleMarker = new AMap.CircleMarker({
                //     map: this.map,
                //     center: location,
                //     radius: 15,  //半径
                //     strokeColor: "#fefefe",  //线颜色
                //     strokeWeight: 1,  //线粗细度
                //     fillColor: '#1CC7FF',  //填充颜色
                //     fillOpacity: 0.8 ,//填充透明度
                // })
                // this.circleMarker.setMap(this.map)
                this.map.setCenter(location);
            },
            // 地图添加标志
            add_markers:function(){
                this.change_map_center(this.tabledata[0].location)
                for( let i = 0; i < this.infomycontent.length; i++)
                {
                    var infoWindow
                    // console.log(this.locations[i])
                    this.markers.push(
                        new AMap.Marker({ 
                            //添加自定义点标记
                            map: this.map,
                            position: this.tabledata[this.markers_list[i]].location, //基点位置
                            offset: new AMap.Pixel(-22, -50), //相对于基点的偏移位置
                            draggable: false,  //是否可拖动
                            // icon : "/static/logo2.png",
                            content: this.mycontent_marker[i],
                        })
                    )
                    this.markers[i].uuid = i
                    this.markers[i].mylnglat = this.tabledata[this.markers_list[i]].location
                    this.markers[i].on('mouseover',(e) =>{
                        infoWindow = new AMap.InfoWindow({
                            isCustom: true,  //使用自定义窗体
                            content : this.infomycontent[e.target.uuid],
                            draggable: false,
                            // offset: new AMap.Pixel(-140, 225),
                            offset: new AMap.Pixel(260, -500),
                        })
                        e.target.setIcon("/static/logo3.png")
                        infoWindow.open(this.map,e.target.mylnglat)
                    })
                    this.markers[i].on('mouseout',(e) => {
                        e.target.setIcon("/static/logo2.png")
                        infoWindow.close()
                    })
                }
            },
            add_markers_all:function(add_data){
                let allcamera_marker_icon = '<div class="map_maxbox">\
                                                <div class="map_iconbg">\
                                                </div>\
                                                <div class="map_iconimg">\
                                                    <img src='+ this.icon_marker +' />\
                                                </div>\
                                            </div>'
                for( let i = 0; i < add_data.length; i++)
                {   
                    // console.log(add_data[i].cameraStatus)
                    let camerastatus = '',eye_div = ''
                    if( add_data[i].cameraStatus ){
                        camerastatus = '<div class="state state1">正常</div>'
                        eye_div = '<div class="face_icon1 face_icon1_img face_icon_fpath" onclick="skip_to_realtimem(\''
                                   + add_data[i].sdkId +'\',\'' + add_data[i].name + '\' )" title="跳转实时监控"></div>'
                    }else{
                        camerastatus = '<div class="state state2">闲置</div>'
                        eye_div = '<div class="face_icon1 face_icon1_img face_icon_fpath" style="cursor: not-allowed;" title="闲置状态不可跳转"></div>'
                    }

                    let infomycontent = '<div class="face_infobox">\
                                            <div class="face_title">\
                                                <div class="snap">\
                                                    <div class="snap_text1">抓拍:</div>\
                                                    <div class="snap_text2" title="'+ add_data[i].snapCount +'">'+ add_data[i].snapCount +'</div>\
                                                </div>\
                                                '+ camerastatus +'\
                                                '+ eye_div +'\
                                                <div class="face_icon2 face_icon2_img face_icon_fpath" onclick="skip_to_mmanage4(\''
                                                + add_data[i].groupName +'\',\'' + add_data[i].name + '\')" title="跳转到设备配置"></div>\
                                            </div>\
                                            <div class="face_camera">'+add_data[i].name+'</div>\
                                            <div class="face_conter"></div>\
                                        </div>'
                    var infoWindow,markers
                    markers = new AMap.Marker({ 
                            //添加自定义点标记
                            map: this.map,
                            position: add_data[i].location, //基点位置
                            offset: new AMap.Pixel(-22, -50), //相对于基点的偏移位置
                            draggable: false,  //是否可拖动
                            // icon : "/static/logo2.png",
                            content: allcamera_marker_icon,
                        })
                    markers.uuid = add_data[i].uuid
                    markers.mylnglat = add_data[i].location
                    markers.on('mouseover',(e) =>{
                        infoWindow = new AMap.InfoWindow({
                            isCustom: true,  //使用自定义窗体
                            content : infomycontent,
                            draggable: false,
                            offset: new AMap.Pixel(260, -500),
                        })
                        // e.target.setIcon("/static/logo3.png")
                        infoWindow.open(this.map,e.target.mylnglat)
                    })
                    markers.on('mouseout',(e) => {
                        // e.target.setIcon("/static/logo2.png")
                        infoWindow.close()
                    })
                }
            },
            // 显示折线
            add_line:function(){
                this.polyline = new AMap.Polyline({
                    map: this.map,
                    path: this.locations,
                    strokeColor: "#00a0e9",  //线颜色
                    strokeOpacity: 1,     //线条透明度，取值范围[0,1]，0表示完全透明，1表示不透明。默认为0.9
                    strokeWeight: 3,      //线宽
                    strokeStyle: "solid",  //线样式
                });
                this.polyline.setMap(this.map) // 显示折线
            },
            // 添加mark图标
            add_markers_icon:function(num){
                return '<div class="map_maxbox">\
                            <div class="map_iconbg">\
                                <div class="map_icontext">'+ (this.init_data.allnum - num) +'</div>\
                            </div>\
                        </div>'
            },

            // 上传图片
            imgPreview:function(file){
                let self = this;
                // 看支持不支持FileReader
                if (!file || !window.FileReader) return;
        
                if (!/image\/\w+/.test(file.type)) {
                    this.warning_info("请选择图片")
                    return false;
                }

                // 创建一个reader
                var reader = new FileReader()
                // 将图片将转成 base64 格式
                reader.readAsDataURL(file)
                // 读取成功后的回调
                reader.onloadend = function (e) {
                    self.dataUrl = e.target.result
                    let  image = new Image()
                    let Maxpic = 4000
                    image.onload = () => {
                        let width = image.width
                        let height = image.height
                        if( width > Maxpic || height > Maxpic ){
                            let PicBaseText=self.compress(image,width*0.5,height*0.5,1);
                            self.search_data.photo = self.dataURItoBlob(PicBaseText);
                            // console.log(self.search_data.photo.size)
                        }
                    }
                    image.src = e.target.result
                }
                
            },
            handleFileChange:function(e){
                this.click_to_clear(false)
                let inputDOM = this.$refs.inputer

                // console.log(this.$refs.inputer.files[0])

                let tempdata = inputDOM.files[0]
                if( tempdata.size < 10*1024*1024 ){
                    this.search_data.photo = inputDOM.files[0]
                    this.imgPreview(this.search_data.photo)
                }else{
                    this.$refs.inputer.value = ""
                    this.warning_info("图片大小不超过10M")
                } 
            },
            // 图片压缩 canvas
            compress:function(img, width, height, ratio) {
                var canvas, ctx, img64;

                canvas = document.createElement('canvas');
                canvas.width = width;
                canvas.height = height;

                ctx = canvas.getContext("2d");
                ctx.drawImage(img, 0, 0, width, height);

                img64 = canvas.toDataURL("image/jpeg",ratio);

                return img64;
            },
            // base64 转 二进制 图片
            dataURItoBlob:function(dataURI) {
                let byteString = atob(dataURI.split(',')[1]);
                let mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];
                let ab = new ArrayBuffer(byteString.length);
                let ia = new Uint8Array(ab);
                for (let i = 0; i < byteString.length; i++) {
                    ia[i] = byteString.charCodeAt(i);
                }
                console.log([ab]);
                return new Blob([ab], {type: mimeString});
            },


            // 初始化请求数据
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
                this.add_markers_all(this.allcamera_list)
            },
            post_to_get_facepath:function(search_data,model=""){
                var params = new FormData()
                let cameraGroupIds = this.$store.state.facepath_search_data.cameraGroupIds
                let cameraIds = this.$store.state.facepath_search_data.cameraIds
                if( cameraGroupIds.length ){
                    cameraGroupIds.join(',')
                    params.append( "cameraGroupIds", cameraGroupIds) // 搜索设备组id
                }else{
                    params.append( "cameraGroupIds", "" )
                }
                if( cameraIds.length ){
                    cameraIds.join(',')
                    params.append( "cameraIds", cameraIds) // 搜索设备id
                }else{
                    params.append( "cameraIds", "" )
                }
                if( search_data.identifyNumber ){
                    params.append( "identifyNumber", search_data.identifyNumber )
                }
                
                if( this.datevalue ){
                    // console.log(this.datevalue)
                    params.append( "startTime", this.datevalue[0]-1 )
                    params.append( "endTime", this.datevalue[1]-1 )
                }else{
                    params.append( "startTime", "" )
                    params.append( "endTime", "" )
                }
                if( -1 < search_data.confidence && search_data.confidence < 101 ){
                    params.append( "confidence", search_data.confidence )
                }else{
                    this.error_info("选择可信度")
                    return ;
                }
                if( search_data.photo ){
                    params.append( "photo", search_data.photo )
                }else if( this.dataUrl ){
                    if( search_data.photoUrl ){
                        params.append("photoUrl",search_data.photoUrl)
                    }else{
                        params.append("photoUrl","")
                    }
                }else{
                    // console.log(search_data)
                    this.error_info("请添加图片")
                    return ;
                }
                // 请求搜索轨迹
                // var params = new URLSearchParams()
                // params.append("")
                this.$ajax.post("/main/identifySnap",params,{headers: {'Content-Type': 'multipart/form-data'}}).then((res) => {
                // this.$ajax.post("",params).then((res) => {
                    if( res.data.status === 0){

                        if( !res.data.data.total ){
                            this.$message({
                                type: 'warning',
                                message: '无对应数据',
                                showClose: true,
                                center: true
                            })
                            return 
                        }
                        this.init_data.allnum = res.data.data.total
                        this.tabledata = res.data.data.list
                        // console.log(this.tabledata)
                        for( let i = 0; i < this.tabledata.length; i++){
                            this.tabledata[i].uuid = i
                            if( i+1 === this.tabledata.length ){
                                this.tabledata[i].mystyle = "none"
                            }else{
                                this.tabledata[i].mystyle = "2px solid white"
                            }
                            [this.tabledata[i].nyr,this.tabledata[i].sfm] = this.tabledata[i].catchTime.split(" ")
                            this.tabledata[i].location = [this.tabledata[i].location.split(",")[0],this.tabledata[i].location.split(",")[1]]
                            this.locations.push(this.tabledata[i].location)
                        }
                        this.map_info() // 组装窗口
                        this.add_markers() // 添加标记
                        this.add_line() // 添加轨迹
                    }else{
                        this.mes_handling(res.data.status,res.data.msg)
                    }
                }).catch((error) => {
                    console.log(error)
                    this.error_info('网络连接出错')
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

            // 按钮事件
            // 搜索按钮
            click_to_search:function(search_data){
                this.click_to_clear(false)
                this.post_to_get_facepath(search_data)
            },
            // 清空事件
            click_to_clear:function(flag=true){
                this.markers = []
                this.locations = []
                this.infomycontent = []
                this.mycontent_marker = []
                this.markers_list = []
                this.map.clearMap()
                this.tabledata = null
                this.init_data.allnum = 0
                if( flag ){
                    this.datevalue = [(new Date() - 3600 * 1000 * 24 * 15),new Date()-1],
                    // this.tabledata = null
                    this.dataUrl = ""
                    this.$refs.inputer.value = ""
                    this.search_data = {
                        photo: "",
                        confidence: this.default_confidence,
                        startTime: "",
                        endTime: "",
                        identifyNumber: 10,
                    }
                    this.add_markers_all(this.allcamera_list)
                    if( this.polyline ){
                        this.polyline.setMap(null)
                    }
                }
            },
            // 导出事件
            // 导出数据为excel
            export_data2excel:function(){
                require.ensure([], () => {
                    const { export_json_to_excel } = require('../vendor/Export2Excel');
                    const tHeader = ['设备名','年-月-日','时:分:秒','相似度','人脸'];
                    const filterVal = ['cameraName','nyr','sfm','confidence','snapshotUrl'];
                    let export_list = [];
                    for ( let i = 0; i < this.tabledata.length; i++){
                        export_list.push(JSON.parse(JSON.stringify(this.tabledata[i])))
                    }
                    const list = export_list;
                    const data = this.formatJson(filterVal, list);
                    console.log(data)
                    export_json_to_excel(tHeader, data, '导出列表');
                })
            },
            formatJson:function(filterVal, jsonData) {
                return jsonData.map(v => filterVal.map(j => v[j]))
            },

            // 组装窗口
            map_info:function(){
                var contects = []
                var location_remark = []
                for ( let i = 0; i < this.tabledata.length; i++ ){
                    let tempdata = JSON.parse(JSON.stringify(this.tabledata[i]))
                    let contect = '<div class="conter_box">\
                                        <div class="conter_img"><img src='+ tempdata.snapshotUrl +'></div>\
                                        <div class="conter_textbox">\
                                            <p>轨迹序号：'+ (this.init_data.allnum - tempdata.uuid) +'</p>\
                                            <p>'+ tempdata.nyr +'</p>\
                                            <p>'+ tempdata.sfm +'</p>\
                                        </div>\
                                    </div>'
                    if( location_remark.length ){
                        let flag = false
                        for( let j = 0; j < location_remark.length; j++ ){
                            if( location_remark[j][0] === this.tabledata[i].location[0] && location_remark[j][1] === this.tabledata[i].location[1]){
                                flag = true
                                contects[j] = contects[j] + contect
                            }
                        }
                        if( !flag ){
                            location_remark.push( this.tabledata[i].location )
                            contects.push( contect )
                            this.mycontent_marker.push( this.add_markers_icon(this.tabledata[i].uuid) )
                            this.markers_list.push( this.tabledata[i].uuid )
                        }
                    }else{
                        this.mycontent_marker.push( this.add_markers_icon(this.tabledata[i].uuid) )
                        this.markers_list.push( this.tabledata[i].uuid )
                        location_remark.push( this.tabledata[i].location )
                        contects.push( contect )
                    }
                }
                // console.log(contects.length)
                for( let i = 0; i < contects.length; i++ ){
                    let camerastatus = '', eye_div = ''
                    if( this.tabledata[this.markers_list[i]].cameraStatus ){
                        camerastatus = '<div class="state state1">正常</div>'
                        eye_div = '<div class="face_icon1 face_icon1_img face_icon_fpath" " onclick="skip_to_realtimem(\''
                                + this.tabledata[this.markers_list[i]].cameraSdkId +'\',\'' + this.tabledata[this.markers_list[i]].cameraName + '\')" title="跳转实时监控"></div>'
                    }else{
                        camerastatus = '<div class="state state2">闲置</div>'
                        eye_div = '<div class="face_icon1 face_icon1_img face_icon_fpath" style="cursor: not-allowed;" title="闲置状态不可跳转"></div>'
                    }
                    this.infomycontent.push(
                        '<div class="face_infobox">\
                            <div class="face_title">\
                                <div class="snap">\
                                    <div class="snap_text1">抓拍:</div>\
                                    <div class="snap_text2" title="'+ this.tabledata[this.markers_list[i]].snapCount +'">'+ this.tabledata[this.markers_list[i]].snapCount +'</div>\
                                </div>\
                                '+ camerastatus +'\
                                '+ eye_div +'\
                                <div class="face_icon2 face_icon2_img face_icon_fpath" onclick="skip_to_mmanage4(\''
                                + this.tabledata[this.markers_list[i]].cameraGroupName +'\',\'' + this.tabledata[this.markers_list[i]].cameraName + '\')" title="跳转到设备配置"></div>\
                            </div>\
                            <div class="face_camera">'+ this.tabledata[this.markers_list[i]].cameraName +'</div>\
                            <div class="face_conter">'
                            + contects[i] +
                            '</div>\
                        </div>'
                    )
                    // this.infomycontent.push(
                    //     '<div class="face_infobox">\
                    //         <div class="face_title">\
                    //             <div class="state">正常</div>\
                    //         </div>\
                    //         <div class="face_camera">'+ this.tabledata[this.markers_list[i]].cameraName +'</div>\
                    //         <div class="face_conter">'
                    //         + contects[i] +
                    //         '</div>\
                    //     </div>'
                    // )
                }
                
            },
            click_to_test:function(res){
                // this.$router.push('/historyface1')
                console.log(res)
            },

            // 请求默认可信度
            init_confidence:function(){
                var params = new URLSearchParams()
                this.$ajax.post("/getIdentifySnapThreshold",params).then((res) => {
                    if( res.data.status === 0){
                        this.default_confidence = res.data.data
                        this.search_data.confidence = this.default_confidence
                    }else{
                        this.mes_handling(res.data.status,res.data.msg)
                    }
                }).catch((error) => {
                    console.log(error)
                    this.error_info('网络连接出错')
                    return ;
                })
            },

            skip_to_realtimem:function(sdkId,name){
                // 实时监控
                this.$store.state.realtime_data.sdkId = sdkId
                this.$store.state.realtime_data.name = name
                this.$store.state.is_search_data = true
                this.$router.push('/realtimem')
            },
            skip_to_mmanage4(groupName,name){
                this.$store.state.search_data.groupId = groupName
                this.$store.state.search_data.name = name
                this.$store.state.is_search_data = true
                this.$router.push('/mmanage4')
            },
        },
        mounted:function(){
            // 请求默认可信度
            this.init_confidence()

            // 将window原生事件绑定到vue的事件中
            window['skip_to_realtimem'] = (sdkId,name) => {
                this.skip_to_realtimem(sdkId,name)
            }
            window['skip_to_mmanage4'] = (groupName,sdkId) => {
                this.skip_to_mmanage4(groupName,sdkId)
            }
        },
        beforeRouteLeave(to, from, next) {
            if( from.path === "/facepath" && to.path === "/facepath_offline"){
                to.meta.keepAlive = false; 
            }
            next()
        },
        watch:{
            'search_data.confidence':function (newVal,oldVal) {
                if( newVal === "" ){
                    this.search_data.confidence = 0
                }else if( newVal >= 100 ){
                    this.search_data.confidence = 100
                }
                this.search_data.confidence = parseInt(this.search_data.confidence)
            },

            '$store.state.is_search_data_facepath':function(newVal,old){
                if ( newVal  && this.$store.state.facepath_model === "online" ){
                    this.search_data.photo = ""
                    this.search_data.photoUrl = this.$store.state.facepath_data.photo
                    this.dataUrl = this.search_data.photoUrl
                    this.click_to_clear(false)
                    this.post_to_get_facepath(this.search_data,"skip")

                    this.$store.state.is_search_data_facepath = false
                    this.$store.state.facepath_data.photo = ""
                }
            },

            '$store.state.facepath_search_data.allcamera_list':function(newVal,old){
                if( this.$store.state.facepath_model === "online" ){
                    this.allcamera_list = this.$store.state.facepath_search_data.allcamera_list
                    if( this.allcamera_list.length ){
                        this.init( this.allcamera_list[0].location )
                    }else{
                        this.init( this.center_xy )
                    }
                    if ( this.$store.state.is_search_data_facepath ){
                        this.search_data.photo = ""
                        this.search_data.photoUrl = this.$store.state.facepath_data.photo
                        this.dataUrl = this.search_data.photoUrl
                        // this.map.clearMap()
                        this.click_to_clear(false)
                        this.post_to_get_facepath(this.search_data,"skip")

                        this.$store.state.is_search_data_facepath = false
                        this.$store.state.facepath_data.photo = ""
                    }else{
                        this.get_init_data()
                    }
                }
            }
        }
	}
</script>

<style>
    @import "../css/historyface.css";

    /* 地图logo */
    .amap-logo {
        right: 0 !important;
        left: auto !important;
        display: none;
    }
    .amap-copyright{
        display: none !important;
    }

    
    .map_iconbg{
        position: relative;
        width: 44px;
        height: 56px;
        color: white;
        background: url(../assets/facepath/map_icon.png) no-repeat;
        background-size: 100% 100%;
    }
    .map_icontext{
        /*display: none;*/
        width: 44px;
        text-align: center;
        height: 56px;
        line-height: 45px;
        position: absolute;
        font-size: 20px;
    }
    .map_iconimg{
        /*display: none;*/
        position: absolute;
        top:2px;
        left: 2px;
        width: 44px;
        height: 44px;
    }
    .amap-marker-content{
        white-space: inherit !important;
    }
</style>
<style scoped>
    /* map.marker */
    .map_maxbox{
        width: 44px;
        height: 56px;
    }
</style>
