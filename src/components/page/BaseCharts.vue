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
                <div id="chartLine" style="width:100%; height:600px;"></div>
            </el-col>
            <el-col :span="12">
                <div id="chartBar" style="width:100%; height:600px;"></div>
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
            chartBar: null,
            chartLine: null,
            chartPie: null,
            address: [],
            selectAddress: ''
        };
    },
    methods: {
        drawBarChart(res) {
            this.chartBar = echarts.init(document.getElementById('chartBar'));
            var data = res.data;
            var legend = data.legend;
            var y_ax = data.y_axis;
            this.address = data.y_axis;
            var cpu = data.series.cpu;
            var memory = data.series.memory;
            this.chartBar.setOption({
                title: {
                    text: 'CPU Memory统计',
                    subtext: '来自程序上报'
                },
                tooltip: {
                    trigger: 'axis',
                    axisPointer: {
                        type: 'shadow'
                    }
                },
                legend: {
                    data: legend
                },
                grid: {
                    left: '3%',
                    right: '4%',
                    bottom: '3%',
                    containLabel: true
                },
                xAxis: {
                    type: 'value',
                    boundaryGap: [0, 0.01]
                },
                yAxis: {
                    type: 'category',
                    data: y_ax
                },
                series: [
                    {
                        name: 'cpu',
                        type: 'bar',
                        data: cpu
                    },
                    {
                        name: 'memory',
                        type: 'bar',
                        data: memory
                    }
                ]
            });
        },
        drawLineChart(res) {
            this.chartLine = echarts.init(document.getElementById('chartLine'));
            var data = res.data;
            var legend = data.legend;
            var x_ax = data.x_axis;
            var cpu = data.series.cpu;
            var memory = data.series.memory;
            this.chartLine.setOption({
                title: {
                    text: '单机CPU、Mem使用率',
                    subtext: '百分比'
                },
                tooltip: {
                    trigger: 'axis'
                },
                legend: {
                    data: legend
                },
                grid: {
                    left: '3%',
                    right: '4%',
                    bottom: '3%',
                    containLabel: true
                },
                xAxis: {
                    type: 'category',
                    boundaryGap: false,
                    data: x_ax
                },
                yAxis: {
                    type: 'value'
                },
                series: [
                    {
                        name: 'cpu',
                        type: 'line',
                        stack: '使用率',
                        data: cpu
                    },
                    {
                        name: 'memoey',
                        type: 'line',
                        stack: '使用率',
                        data: memory
                    }
                ]
            });
        },
        getHardwareInfo() {
            Vue.axios
                .get('http://192.168.1.106:65221/api/v1/host/info')
                .then(res => {
                    this.drawBarChart(res.data);
                    this.getSingleUsage(this.address[0]);
                })
                .catch(err => {
                    console.log(err);
                });
        },
        getSingleUsage(addr) {
            console.log(this.address);
            Vue.axios({
                mothod: 'get',
                url: 'http://192.168.1.106:65221/api/v1/host/single',
                params: {
                    address: addr
                }
            }).then(res => {
                this.drawLineChart(res.data);
            });
        },
        drawCharts() {
            this.getHardwareInfo();
        },
        changeSelector() {
            this.getSingleUsage(this.selectAddress);
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