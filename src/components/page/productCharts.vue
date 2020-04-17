<template>
    <section class="chart-container">
        <div>
        <span>选择主机：</span>
        <el-select
            v-model="selectAddress"
            placeholder="请选择"
            @change="changeSelector()"
            filterable
        >
            <el-option v-for="item in address" :key="item" :label="item" :value="item"></el-option>
        </el-select>
        </div>
        <el-row>
            <el-col :span="12">
                <div id="chartColumn" style="width:100%; height:600px;"></div>
            </el-col>
            <el-col :span="12">
                <div id="chartPie" style="width:100%; height:600px;"></div>
            </el-col>
        </el-row>
    </section>
</template>

<script>
import echarts from 'echarts';

import Vue from 'vue';

export default {
    data() {
        return {
            chartColumn: null,
            chartPie: null,
            address: [],
            selectAddress: ''
        };
    },
    methods: {
        drawColumnChart(res) {
            this.chartColumn = echarts.init(document.getElementById('chartColumn'));
            var data = res.data;
            this.chartColumn.setOption({
                title: { text: 'Column Chart' },
                tooltip: {},
                xAxis: {
                    data: data.x_axis
                },
                yAxis: {},
                series: [
                    {
                        name: 'memory',
                        type: 'bar',
                        data: data.series
                    }
                ]
            });
        },
        drawPieChart(res) {
            this.chartPie = echarts.init(document.getElementById('chartPie'));
            var data = res.data;
            this.address = data.legend;
            this.chartPie.setOption({
                title: {
                    text: '申请数量统计',
                    x: 'center'
                },
                tooltip: {
                    trigger: 'item',
                    formatter: '{a} <br/>{b} : {c} ({d}%)'
                },
                legend: {
                    orient: 'vertical',
                    left: 'left',
                    data: data.legend
                },
                series: [
                    {
                        name: '主机端口',
                        type: 'pie',
                        radius: '55%',
                        center: ['50%', '60%'],
                        data: data.series,
                        itemStyle: {
                            emphasis: {
                                shadowBlur: 10,
                                shadowOffsetX: 0,
                                shadowColor: 'rgba(0, 0, 0, 0.5)'
                            }
                        }
                    }
                ]
            });
        },
        getHardwareInfo() {
            Vue.axios
                .get('http://192.168.1.106:65221/api/v1/host/register')
                .then(res => {
                    this.drawPieChart(res.data);
                    this.getSingleRegister(this.address[0]);
                })
                .catch(err => {
                    console.log(err);
                });
        },
        drawCharts() {
            this.getHardwareInfo();
        },
        changeSelector() {
            this.getSingleRegister(this.selectAddress);
        },
        getSingleRegister(addr){
            Vue.axios({
                method:"get",
                url:"http://192.168.1.106:65221/api/v1/host/register/memory",
                params:{
                    address:addr
                }
            }).then(res=>{
                this.drawColumnChart(res.data);
            })
        }
    },

    mounted: function() {
        this.drawCharts();
    }
};
</script>


<style scoped>
.chart-container {
    width: 100%;
    float: left;
}
/*.chart div {
        height: 400px;
        float: left;
    }*/

.el-col {
    padding: 30px 20px;
}
</style>