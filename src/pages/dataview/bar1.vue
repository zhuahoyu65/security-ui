<template>
	<div class="bar1">
		<div class="line_head">
			<div class="pie_title">周人流量统计图</div>
			<div class="pie_title"></div>
		</div>
		<!--<div class="main"></div>-->
        <div class="main" v-if="showData.length || dayTime.length || fname.length"></div>
        <div class="main_ch" v-else-if="$store.state.dataview1_flag">
            <div class="main_loading">
                <div class="main_table">
                    <div class="main_cell">
                        <div class="spinner">
                            <div class="rect1"></div>
                            <div class="rect2"></div>
                            <div class="rect3"></div>
                            <div class="rect4"></div>
                            <div class="rect5"></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="main_ch" v-else>
            <div class="main_text">
                <div class="main_table">
                    <div class="main_cell">
                        -- 暂无数据 --
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
	import echarts from 'echarts'

	export default{
		props: {
			showData:Array,
			dayTime: Array,
			
			fname: Array,
			flag: Boolean,
		},
		data(){
			return {
				legendArr: [],
				// color: this.$store.state.color,
				myChart: {},
				name: '柱形图',

				color: this.$store.state.color1,
				text_color: this.$store.state.text_color,
			}
		},
		methods:{
			my_init() {
				this.legendArr = this.myChart.getOption().series
				this.legendArr.forEach((data) => {
					data.selected = true;
				})
				this.$root.charts.push(this.myChart)
				window.addEventListener('resize', function() {
					this.myChart.resize()
				}.bind(this))
			},
			line_echart_init_test:function( data ){
				let option = {
					grid:{
	                	y: 100,
	                    y2: 100,
	                },
					tooltip : {
				        trigger: 'axis'
				    },
				    // 图例
				    legend: {
				        data: this.fname,
				        textStyle:{
				        	color: 'auto',
				        	fontSize: 16,
				        },
				        orient: 'horizontal',// 'horizontal' | 'vertical' 水平布局|垂直布局

				        // 图例位置
				        x: 'center',
				        y: 20,
				        itemWidth: 25,
				        itemHeight: 10,
				        itemGap: 20,

				        // 内边距
				        padding: 10,
				    },
				    toolbox: {
				        show : true,
				        // orient : 'vertical', // 垂直显示
		                x: '1150px',
		                // x: 'right', 
		                // y: 'center',
				        feature : {
				            mark : {show: true}, // 
				            dataView : {show: false, readOnly: false},
				            magicType : {
				            	show: false, 
				            	type: ['line', 'bar', 'stack', 'tiled'],
				            },
				            restore : {show: false},
				            saveAsImage : {show: true}
				        }
				    },
				    calculable : false, // 拖拽重计算特性
				    backgroundColor: 'rgba(0,0,0,0.4)', // 修改背景颜色
				    color: this.color, // 线条颜色
				    xAxis : [
				        {
				        	name : '时间', 
				            type : 'category',
				            data : this.dayTime,
				            axisLine:{
				            	lineStyle: {
				            		color: 'auto',
				            		type: 'dashed',
				            	},
				            }, // 线条设置
				            axisLabel:{
				            	textStyle:{
		            				color: this.text_color,
		            			},
				            }, // 字体设置
				            splitLine:{
				            	show: false,
				            },
				            axisTick:{
				            	show: false,
				            }
				        }
				    ],
				    yAxis : [
				        {
				        	name : '人流量',
				            type : 'value',
				            axisLine:{
				            	show: false,
				            	lineStyle: {
				            		color: this.text_color,
				            	},
				            },
				            axisLabel:{
				            	textStyle:{
		            				color: this.text_color,
		            			},
				            },
				            splitLine:{
				            	show: true,
				            	lineStyle:{
				            		color: ['rgba(0, 255, 0, 0.2)'],
				            		type: 'dotted',
				            	},
				            },
				            axisTick:{
				            	show: false,
				            }
				        }
				    ],

				    series: this.series_data( data )
				}


                this.myChart = echarts.init(document.querySelector('.bar1 .main'))
                this.myChart.setOption(option, true)

			},
			// 数据组装
			series_data:function( showdata ){
				let data = []
				for( let i = 0; i < showdata.length; i++ ){
					data.push(
						{
				            name: this.fname[i],
				            type: 'bar',
				            stack: this.fname[i],
				            data: showdata[i],
				        }
					)
				}
				// console.log(data)
				return data
			}
		},
		mounted(){
            // this.line_echart_init_test(this.showData)
		},
		watch:{
		    'flag':function (newval,old) {
                this.$nextTick(function() {
                    this.line_echart_init_test(this.showData)
                    this.my_init()
                })
            },
			// 'flag':function(newval,old){
            //     // this.$nextTick(function() {
            //     //     this.line_echart_init_test(this.showData)
            //     //     this.my_init()
            //     // })
			// }
		},

	}
	
</script>


<style scoped>
	.bar1{
		width: 100%;
		height: 100%;
	}
	.main{
		width: 100%;
		height: calc( 100% - 60px );
	}
	.line_head{
		width: 100%;
		height: 60px;
		/*margin-bottom: 10px;*/
		background-color: rgba(0,0,0,0.5);
	}
	.pie_title{
         width: 300px;
         height: 60px;
         line-height: 60px;
         color: #cccccc;
         font-size: 20px;
         margin-left: 20px;
	}
	.pie_title{
        width: 300px;
        height: 60px;
        line-height: 60px;
        color: #cccccc;
        font-size: 20px;
        margin-left: 20px;
        float: left;
	}
</style>
