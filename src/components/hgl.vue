<template>
    <div class="echarts">
        <div style="width:100;height:100%" ref="hgl"></div>
        <div class="mapChoose">
            <span v-for="(item,index) in parentInfo" :key="item.code">
                <span class="title" @click="chooseArea(item,index)">{{item.cityName=='全国'?'中国':item.cityName}}</span>
                <span class="icon" v-show="index+1!=parentInfo.length">></span>
            </span>
        </div>
    </div>
</template>

<script>
    import echarts from 'echarts'
    import resize from "./mixins/resize";
    import myStore from '@/components/Store';
    
    import china from 'echarts/map/json/china.json'
    // import axios from 'axios'

    export default {
        name: 'hgl',
        mixins: [resize],
        data() {
            return {
                myCharts: null,
                geoJson: {
                    features: []
                },
                parentInfo: [{
                    cityName: "全国",
                    code: 100000
                }],
                timeTitle: ['2014', '2015', '2016', '2017', '2018']
            }
        },
        mounted() {
            this.getGeoJson(100000)
            // this.getGoodsList()
        },
        methods: {
            // getGoodsList () {
            //         this.$axios.get('../assets/data_14.json').then((res) => {
            //         console.log(res)
            //         })
            // },
            getGeoJson(adcode) {
                let that = this;
                AMapUI.loadUI(["geo/DistrictExplorer"], DistrictExplorer => {
                    var districtExplorer = new DistrictExplorer();
                    districtExplorer.loadAreaNode(adcode, function (error, areaNode) {
                        if (error) {
                            console.error(error);
                            return;
                        }
                        let Json = areaNode.getSubFeatures();
                        
                        if (Json.length > 0) {
                            that.geoJson.features = Json;
                        } else if (Json.length === 0) {
                            that.geoJson.features = that.geoJson.features.filter(
                                item => item.properties.adcode == adcode
                            );
                            if (that.geoJson.features.length === 0) return;
                        }

                        // console.log('geoJson')
                        // console.log(that.geoJson.features)
                        // console.log(that.geoJson)

                        // for(var i=0,len=that.geoJson.features.length; i<len; i++){
                        //     var sglProvinceProperties = that.geoJson.features[i].properties;
                        //     var sglProvinceName = sglProvinceProperties.name;
                        //     console.log(sglProvinceName)
                        //     console.log(sglProvinceProperties.center)
                        //     switch(sglProvinceName){
                        //             case '内蒙古自治区':
                        //                     that.geoJson.features[i].properties.center[0]=112.17;
                        //                     that.geoJson.features[i].properties.center[1]=42.81;
                        //                     break;
                        //             case '四川'://def:101.778916,36.623178
                        //                     that.geoJson.features[i].properties.center[0]=116.405285;
                        //                     that.geoJson.features[i].properties.center[1]=39.904989;
                        //                     break;
                        //             case '北京市'://def:118.767413,32.041544
                        //                     that.geoJson.features[i].properties.center[0]=112.17;
                        //                     that.geoJson.features[i].properties.center[1]=42.81;
                        //                     that.geoJson.features[i].properties.centroid[0]=112.17;
                        //                     that.geoJson.features[i].properties.centroid[1]=42.81;
                        //                     break;
                        //         }
                        // }

                        that.getMapData();

                    });
                });
            },
            //获取数据
            getMapData() {
                console.log(myStore.data_14)
                let mapData = {},
                    pointData = {},
                    sum = {},
                    data_len = {}
                let all_data = myStore.data_all
                console.log(all_data)
                this.timeTitle.forEach(item => {
                    mapData[item] = []
                    pointData[item] = []
                    sum[item] = 0
                    data_len[item] = 0

                    let cur_data = all_data[item]
                    console.log(cur_data)
                    cur_data.forEach(j => {
                        let name = j["地区"]
                        let value = j["产品抽查合格率（％）"]
                        mapData[item].push({
                            name: name,
                            value: value,
                            // cityCode: j.properties.adcode
                        })
                        pointData[item].push({
                            name: name,
                            value: value,
                            // cityCode: j.properties.adcode
                        })
                        sum[item] += value
                        data_len[item] += 1
                    })


                    // this.geoJson.features.forEach(j => {
                    //     let value = Math.random() * 3000
                    //     mapData[item].push({
                    //         name: j.properties.name,
                    //         value: value,
                    //         cityCode: j.properties.adcode
                    //     })
                    //     pointData[item].push({
                    //         name: j.properties.name,
                    //         value: [j.properties.center[0], j.properties.center[1], value],
                    //         cityCode: j.properties.adcode
                    //     })
                    //     sum[item] += value
                    // })
                    mapData[item] = mapData[item].sort(function (a, b) {
                        return b.value - a.value
                    });
                })

                this.initEcharts(mapData, pointData, sum, data_len)
            },
            initEcharts(mapData, pointData, sum, data_len) {
                // console.log(data_len)
                this.myChart = echarts.init(this.$refs.hgl)

                // for(var i=0,len=this.geoJson.features.length; i<len; i++){
                //     var sglProvinceProperties = this.geoJson.features[i].properties;
                //     var sglProvinceName = sglProvinceProperties.name;
                //     switch(sglProvinceName){
                //             case '内蒙古自治区':
                //                     sglProvinceProperties.cp[0]=112.17;
                //                     sglProvinceProperties.cp[1]=42.81;
                //                     break;
                //             case '四川'://def:101.778916,36.623178
                //                         sglProvinceProperties.cp[0]=121.46;
                //                         sglProvinceProperties.cp[1]=31.28;
                //                     break;
                //             case '江苏'://def:118.767413,32.041544
                //                     sglProvinceProperties.cp[0]=119.767413;
                //                     sglProvinceProperties.cp[1]=33.041544;
                //                     break;
                //         }
                // }

                // console.log('geoJson2')
                // console.log(this.geoJson.features)
                // console.log(this.geoJson)

                // var chinaEchartsObj = echarts.getMap('china');
                // var geoJSONChina = chinaEchartsObj.geoJson;
                var geoJSONChina = china;

                // console.log('geoJsonEcharts')
                // console.log(geoJSONChina)


                if (this.parentInfo.length === 1) {
                    echarts.registerMap('china', geoJSONChina); //注册
                } else {
                    echarts.registerMap('map', this.geoJson); //注册
                }

                
                
                var option = {
                    timeline: {
                        data: this.timeTitle,
                        axisType: 'category',
                        autoPlay: true,
                        playInterval: 3000,
                        left: '10%',
                        right: '10%',
                        bottom: '2%',
                        width: '80%',
                        label: {
                            normal: {
                                textStyle: {
                                    color: 'rgb(179, 239, 255)'
                                }
                            },
                            emphasis: {
                                textStyle: {
                                    color: '#fff'
                                }
                            }
                        },
                        symbolSize: 10,
                        lineStyle: {
                            color: '#8df4f4'
                        },
                        checkpointStyle: {
                            borderColor: '#8df4f4',
                            color: '#53D9FF',
                            borderWidth: 2,
                        },
                        controlStyle: {
                            showNextBtn: true,
                            showPrevBtn: true,
                            normal: {
                                color: '#53D9FF',
                                borderColor: '#53D9FF'
                            },
                            emphasis: {
                                color: 'rgb(58,115,192)',
                                borderColor: 'rgb(58,115,192)'
                            }
                        },

                    },
                    baseOption: {
                        animation: true,
                        animationDuration: 900,
                        animationEasing: 'cubicInOut',
                        animationDurationUpdate: 900,
                        animationEasingUpdate: 'cubicInOut',
                        tooltip: {
                            trigger: 'axis',
                            axisPointer: {
                                type: 'shadow'
                            },
                        },
                        grid: {
                            right: '2%',
                            top: '12%',
                            bottom: '8%',
                            width: '20%'
                        },
                        toolbox: {
                            feature: {
                                restore: {
                                    show: false
                                },
                                dataView: {
                                    optionToContent: function (opt) {
                                        let series = opt.series[0].data //折线图数据
                                        let tdHeads =
                                            '<th  style="padding: 0 20px">地区</th><th style="padding: 0 20px">合格率</th>' //表头
                                        let tdBodys = '' //数据
                                        let table =
                                            `<table border="1" style="margin-left:20px;border-collapse:collapse;font-size:14px;text-align:left;"><tbody><tr>${tdHeads} </tr>`
                                        for (let i = 0; i < series.length; i++) {
                                            table += `<tr>
                              <td style="padding: 0 50px">${series[i].name}</td>
                              <td style="padding: 0 50px">${series[i].value.toFixed(2)}%</td>
                              </tr>`
                                        }
                                        table += '</tbody></table>'
                                        return table
                                    }
                                },
                                saveAsImage: {
                                    name: this.parentInfo[this.parentInfo.length - 1].cityName + '合格率统计图'
                                },
                                dataZoom: {
                                    show: false
                                },
                                magicType: {
                                    show: false
                                }
                            },
                            iconStyle: {
                                normal: {
                                    borderColor: '#1990DA'
                                }
                            },
                            top: 15,
                            right: 35
                        },
                        geo: {
                            map: this.parentInfo.length === 1 ? 'china' : 'map',
                            zoom: 1.1,
                            roam: true,
                            center: this.parentInfo.length === 1 ? ['118.83531246', '32.0267395887'] : false,
                            tooltip: {
                                trigger: 'item',
                                formatter: (p) => {
                                    let val = p.value[2];
                                    if (window.isNaN(val)) {
                                        val = 0;
                                    }
                                    let txtCon =
                                        "<div style='text-align:left'>" + p.name + ":<br />合格率：" + val.toFixed(
                                            2) + '%</div>';
                                    return txtCon;
                                }
                            },
                            label: {
                                normal: {
                                    show: true,
                                    color: "rgb(249, 249, 249)", //省份标签字体颜色
                                    formatter: p => {
                                        switch (p.name) {
                                            case '内蒙古自治区':
                                                p.name = "内蒙古"
                                                break;
                                            case '西藏自治区':
                                                p.name = "西藏"
                                                break;
                                            case '新疆维吾尔自治区':
                                                p.name = "新疆"
                                                break;
                                            case '宁夏回族自治区':
                                                p.name = "宁夏"
                                                break;
                                            case '广西壮族自治区':
                                                p.name = "广西"
                                                break;
                                            case '香港特别行政区':
                                                p.name = "香港"
                                                break;
                                            case '澳门特别行政区':
                                                p.name = "澳门"
                                                break;
                                            default:
                                                break;
                                        }
                                        return p.name;
                                    }
                                },
                                emphasis: {
                                    show: true,
                                    color: '#f75a00',
                                }
                            },
                            itemStyle: {
                                normal: {
                                    areaColor: '#24CFF4',
                                    borderColor: '#53D9FF',
                                    borderWidth: 1.3,
                                    shadowBlur: 15,
                                    shadowColor: 'rgb(58,115,192)',
                                    shadowOffsetX: 7,
                                    shadowOffsetY: 6,
                                },
                                emphasis: {
                                    areaColor: '#8dd7fc',
                                    borderWidth: 1.6,
                                    shadowBlur: 25,
                                }

                            },
                        },

                    },
                    options: []
                }
                this.timeTitle.forEach(item => {
                    var xData = [],
                        yData = []
                    var min = mapData[item][mapData[item].length - 1].value
                    if (min == 0){
                        min = mapData[item][mapData[item].length - 2].value 
                    }
                    var max = mapData[item][0].value
                    if (mapData[item].length === 1) {
                        min = 0
                    }
                    mapData[item].forEach(c => {
                        xData.unshift(c.name)
                        yData.unshift(c.value)
                    })
                    option.options.push({
                        title: [{
                                left: 'center',
                                top: 10,
                                text: item + this.parentInfo[this.parentInfo.length - 1].cityName +
                                    '年' +
                                    '合格率统计图',
                                textStyle: {
                                    color: 'rgb(179, 239, 255)',
                                    fontSize: 16
                                },
                            },
                            {
                                text: "平均合格率：" + (sum[item]/data_len[item]).toFixed(2) + '%',
                                left: 'center',
                                top: '6.5%',
                                textStyle: {
                                    color: '#FFAC50',
                                    fontSize: 26
                                }
                            }
                        ],
                        visualMap: {
                            min: min,
                            max: max,
                            left: '3%',
                            bottom: '5%',
                            calculable: true,
                            seriesIndex: [0],
                            inRange: {
                                color: ['#24CFF4', '#2E98CA', '#1E62AC']
                            },
                            textStyle: {
                                color: '#24CFF4'
                            }
                        },
                        xAxis: {
                            type: 'value',
                            scale: true,
                            position: 'top',
                            boundaryGap: false,
                            splitLine: {
                                show: false
                            },
                            axisLine: {
                                show: true,
                                lineStyle: {
                                    color: '#455B77'
                                }
                            },
                            axisTick: {
                                show: false
                            },
                            axisLabel: {
                                margin: 2,
                                textStyle: {
                                    fontSize: 12,
                                    color: '#c0e6f9'
                                }
                            },
                        },
                        yAxis: {
                            type: 'category',
                            nameGap: 16,
                            axisLine: {
                                show: true,
                                lineStyle: {
                                    color: '#455B77'
                                }
                            },
                            axisTick: {
                                show: false,
                            },
                            axisLabel: {
                                interval: 0,
                                textStyle: {
                                    color: '#c0e6f9'
                                }
                            },
                            data: xData
                        },
                        series: [{
                                name: item + '合格率',
                                type: 'map',
                                geoIndex: 0,
                                map: this.parentInfo.length === 1 ? 'china' : 'map',
                                roam: true,
                                zoom: 1.3,
                                tooltip: {
                                    trigger: "item",
                                    formatter: p => {
                                        let val = p.value;
                                        if (p.name == '南海诸岛') return
                                        if (window.isNaN(val)) {
                                            val = 0;
                                        }
                                        let txtCon =
                                            "<div style='text-align:left'>" + p.name +
                                            ":<br />合格率：" + val.toFixed(2) + '%</div>';
                                        return txtCon;
                                    }
                                },
                                label: {
                                    normal: {
                                        show: false,
                                    },
                                    emphasis: {
                                        show: false,
                                    }
                                },
                                data: mapData[item],

                            }, 
                            // {
                            //     name: '散点',
                            //     type: 'effectScatter',
                            //     coordinateSystem: 'geo',
                            //     rippleEffect: {
                            //         brushType: 'fill'
                            //     },
                            //     itemStyle: {
                            //         normal: {
                            //             color: '#F4E925',
                            //             shadowBlur: 10,
                            //             shadowColor: '#333'
                            //         }
                            //     },
                            //     data: pointData[item],
                            //     // symbolSize: 8,
                            //     symbolSize: function (val) {
                            //         let value = val[2]
                            //         if (value == max) {
                            //             return 27
                            //         }
                            //         return 10
                            //     },
                            //     showEffectOn: 'render', //加载完毕显示特效
                            // },
                            {
                                type: 'bar',
                                barGap: '-100%',
                                barCategoryGap: '55%',
                                itemStyle: {
                                    normal: {
                                        barBorderRadius: 30,
                                        color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [{
                                            offset: 0,
                                            color: 'rgb(57,89,255,1)'
                                        }, {
                                            offset: 1,
                                            color: 'rgb(46,200,207,1)'
                                        }]),
                                    },
                                    emphasis: {
                                        show: false
                                    }
                                },
                                data: yData
                            }

                        ]
                    })
                })
                this.myChart.setOption(option, true)
                this.myChart.off('click');
                // this.myChart.on('click', this.echartsMapClick);
            },

            //点击下钻
            // echartsMapClick(params) {
            //     if (!params.data) {
            //         return
            //     }
            //     if (
            //         this.parentInfo[this.parentInfo.length - 1].code ==
            //         params.data.cityCode
            //     ) {
            //         return;
            //     }
            //     let data = params.data;
            //     this.parentInfo.push({
            //         cityName: data.name,
            //         code: data.cityCode
            //     });
            //     this.getGeoJson(data.cityCode);

            // },
            // //选择切换市县
            // chooseArea(val, index) {
            //     if (this.parentInfo.length === index + 1) {
            //         return
            //     }
            //     this.parentInfo.splice(index + 1)
            //     this.getGeoJson(this.parentInfo[this.parentInfo.length - 1].code);

            // }
        }
    }
</script>
<style lang="scss" scoped>
    .echarts {
        width: 100%;
        height: 100%;
        position: relative;
        background: url("../assets/bg1.jpg") no-repeat;
        background-size: 100% 100%;
    }

    .mapChoose {
        position: absolute;
        left: 20px;
        top: 55px;
        color: #eee;


        .title {
            padding: 5px;
            border-top: 1px solid rgba(147, 235, 248, .8);
            border-bottom: 1px solid rgba(147, 235, 248, .8);
            cursor: pointer;
        }

        .icon {
            font-family: "simsun";
            font-size: 25px;
            margin: 0 11px;
        }
    }
</style>