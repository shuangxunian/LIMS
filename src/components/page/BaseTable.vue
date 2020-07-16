<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-menu"></i> 基础表格
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <!--
            <div class="handle-box">
                <el-button
                    type="primary"
                    icon="el-icon-delete"
                    class="handle-del mr10"
                    @click="delAllSelection"
                >批量删除</el-button>
                <el-select v-model="query.address" placeholder="地址" class="handle-select mr10">
                    <el-option key="1" label="广东省" value="广东省"></el-option>
                    <el-option key="2" label="湖南省" value="湖南省"></el-option>
                </el-select>
                <el-input v-model="query.name" placeholder="用户名" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
            </div>
            -->
            <div class="handle-box">
                <el-input type="text" placeholder="直接输入房间号即可" class="handle-input mr10" v-model="searchVal"></el-input>
                
                <tr v-for='item in list' :key="item.name">
                    会议室名称：<td>{{item.id}}丨</td>
                    房间号：<td>{{item.location}}丨</td>
                    可容纳：<td>{{item.capacity}}人</td>

                    丨投影仪：
                    <td v-if="item.projectorState===0">正常丨</td>
                    <td v-if="item.projectorState===1">故障丨</td>
                    共有：<td>{{item.computerAll}}台电脑，其中故障</td>
                    <td>{{item.computerBad}}台丨当前房间处于</td>
                    <td v-if="item.state===0">正常</td>
                    <td v-if="item.state===1">占用</td>
                    状态
                </tr>
                
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column prop="id" label="会议室名称" width="150" align="center"></el-table-column>

                <el-table-column prop="location" label="房间号" width="120" align="center"></el-table-column>

                <el-table-column label="容量" width="120" align="center">
                    <template slot-scope="scope">{{scope.row.capacity}}人</template>
                </el-table-column>

                <el-table-column label="计算机数量" width="250" align="center">
                    <template
                        slot-scope="scope"
                    >可用{{scope.row.computerAll - scope.row.computerBad}}台；故障{{scope.row.computerBad}}台</template>
                </el-table-column>

                <el-table-column label="投影仪状态" align="center" width="180">
                    <template slot-scope="scope">
                        <el-tag
                            :type="scope.row.projectorState=== 0 ?'success':(scope.row.projectorState=== 1 ?'danger':'')"
                        >
                            <div v-if="scope.row.projectorState===0">正常</div>
                            <div v-else>故障</div>
                        </el-tag>
                    </template>
                </el-table-column>

                <el-table-column label="当前房间状态" align="center">
                    <template slot-scope="scope">
                        <el-tag
                            :type="scope.row.state=== 0 ?'success':(scope.row.state=== 1 ?'danger':'')"
                        >
                            <div v-if="scope.row.state===0">空闲</div>
                            <div v-else>占用</div>
                        </el-tag>
                    </template>
                </el-table-column>

                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                            type="text"
                            icon="el-icon-edit"
                            @click="handleEdit(scope.$index, scope.row)"
                        >编辑</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <div class="pagination">
                <el-pagination
                    background
                    layout="total, prev, pager, next"
                    :current-page="query.pageIndex"
                    :page-size="query.pageSize"
                    :total="pageTotal"
                    @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="计算机故障的数量" label-width="70">
                    <el-input-number
                        v-model="form.computerBad"
                        @change="handleChange"
                        :min="0"
                        :max="form.computerAll"
                        label="描述文字"
                    ></el-input-number>
                    <!--<el-input v-model="form.computerBad"></el-input>-->
                </el-form-item>

                <el-form-item label="投影仪情况" label-width="70">
                    <el-select v-model="form.projectorState" placeholder="请选择">
                        <el-option
                            v-for="item in options1"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value"
                        ></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="当前房间状态" label-width="70">
                    <el-select v-model="form.state" placeholder="请选择">
                        <el-option
                            v-for="item in options2"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value"
                        ></el-option>
                    </el-select>

                    <!--
                    <el-input v-model="form.address"></el-input>
                    -->
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { fetchData } from '../../api/index';
export default {
    name: 'basetable',
    data() {
        return {
            options1: [
                {
                    value: 0,
                    label: '正常'
                },
                {
                    value: 1,
                    label: '故障'
                }
            ],
            options2: [
                {
                    value: 0,
                    label: '空闲'
                },
                {
                    value: 1,
                    label: '占用'
                }
            ],
            query: {
                address: '',
                name: '',
                pageIndex: 1,
                pageSize: 10
            },
            tableData: [
                {
                    id: '会议室1',
                    location: '501',
                    capacity: 40,
                    projectorState: 0,
                    computerAll: 10,
                    computerBad: 0,
                    state: 1
                },
                {
                    id: '会议室2',
                    location: '502',
                    capacity: 80,
                    projectorState: 0,
                    computerAll: 20,
                    computerBad: 0,
                    state: 0
                },
                {
                    id: '会议室3',
                    location: '503',
                    capacity: 80,
                    projectorState: 0,
                    computerAll: 20,
                    computerBad: 0,
                    state: 0
                },
                {
                    id: '会议室4',
                    location: '601',
                    capacity: 120,
                    projectorState: 0,
                    computerAll: 30,
                    computerBad: 0,
                    state: 0
                },
                {
                    id: '会议室5',
                    location: '602',
                    capacity: 120,
                    projectorState: 1,
                    computerAll: 40,
                    computerBad: 1,
                    state: 0
                }
            ],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            pageTotal: 5,
            form: {},
            idx: -1,
            id: -1,
            searchVal: '' //默认输入为空
        };
    },
    created() {
        this.getData();
    },
    methods: {
        // 获取 easy-mock 的模拟数据
        getData() {
            fetchData(this.query).then(res => {
                console.log(res);
            });
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
        },
        // 保存编辑
        saveEdit() {
            this.editVisible = false;
            this.$message.success(`修改第 ${this.idx + 1} 行成功`);
            this.$set(this.tableData, this.idx, this.form);
        },
        // 分页导航
        handlePageChange(val) {
            this.$set(this.query, 'pageIndex', val);
            this.getData();
        },
        inputFunc() {
            if (this.issue_content.length > 0) {
                this.serch_result_issue = true;
            } else {
                this.serch_result_issue = false;
            }
        }
    },
    computed: {
        list: function() {
            var _this = this;
            //逻辑-->根据input的value值筛选tableData中的数据
            var arrByZM = []; //声明一个空数组来存放数据
            for (var i = 0; i < this.tableData.length; i++) {
                //for循环数据中的每一项（根据name值）
                if (this.tableData[i].location.search(this.searchVal) != -1) {
                    //判断输入框中的值是否可以匹配到数据，如果匹配成功
                    arrByZM.push(this.tableData[i]);
                    //向空数组中添加数据
                }
            }
            //一定要记得返回筛选后的数据
            return arrByZM;
        }
    }
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>
