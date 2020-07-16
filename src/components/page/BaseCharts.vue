<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-pie-chart"></i> schart图表
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="schart-box">
                <div class="content-title">折线图</div>
                <schart class="schart" canvasId="line" :options="options2"></schart>
            </div>
            <div class="schart-box">
                <div class="content-title">饼状图</div>
                <schart class="schart" canvasId="pie" :options="options3"></schart>
            </div>
        </div>
    </div>
</template>

<script>
import Schart from 'vue-schart';
export default {
    name: 'basecharts',
    components: {
        Schart
    },
    data() {
        return {
            options2: {
                type: 'line',
                title: {
                    text: '最近几个月各会议室使用次数'
                },
                bgColor: '#fbfbfb',
                labels: ['1月', '2月', '3月', '4月', '5月', '6月'],
                datasets: [
                    {
                        label: '会议室1',
                        data: [20,18,17,15,10,8]
                    },
                    {
                        label: '会议室2',
                        data: [18,22,20,23,24,30]
                    },
                    {
                        label: '会议室3',
                        data: [20,24,22,27,31,28]
                    },
                    {
                        label: '会议室4',
                        data: [3,4,6,2,3,4]
                    },
                    {
                        label: '会议室5',
                        data: [2,1,2,3,2,2]
                    }
                ]
            },
            options3: {
                type: 'pie',
                title: {
                    text: '当前会议室使用情况'
                },
                legend: {
                    position: 'left'
                },
                bgColor: '#fbfbfb',
                labels: ['空闲', '占用'],
                datasets: [
                    {
                        data: [4, 1]
                    }
                ]
            }
        };
    }
};
</script>

<style scoped>
.schart-box {
    display: inline-block;
    margin: 20px;
}
.schart {
    width: 600px;
    height: 400px;
}
.content-title {
    clear: both;
    font-weight: 400;
    line-height: 50px;
    margin: 10px 0;
    font-size: 22px;
    color: #1f2f3d;
}
</style>