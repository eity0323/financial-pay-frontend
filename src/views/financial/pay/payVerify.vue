<template>
    <div class="app-container calendar-list-container">

        <!-- 搜索区域 -->
        <div class="filter-container">


            <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.search.userId_eq" placeholder="商户" v-if="isAdminRole">
                <el-option v-for="item in userIdOptions" :key="item.key" :label="item.display_name" :value="item.key">
                </el-option>
            </el-select>

            <el-date-picker clearable class="filter-item" v-model="listTimeRange" type="datetimerange" :picker-options="pickerOptions2" placeholder="对账时间" align="right">
            </el-date-picker>

        </div>


        <div class="filter-container">
            <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>
            <el-button class="filter-item" type="primary" icon="document" @click="handleDownload">导出</el-button>
            <!--<el-button class="filter-item" type="primary" icon="plus" @click="handleCreate">添加</el-button>-->
        </div>

        <!-- 列表 -->
        <el-table  :key='tableKey' :data="list" v-loading.body="listLoading" border fit highlight-current-row style="width: 100%">
            <el-table-column align="center" label="对账时间" width="160">
                <template scope="scope">
                    <span>{{scope.row.verifyTime | timeFilter('{y}-{m}-{d}')}} </span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="商户名称" width="250">
                <template scope="scope">
                    <span>{{scope.row.userName}}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="结算金额" width="140">
                <template scope="scope">
                    <span>{{scope.row.settlementMoneyYuan}}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="转账金额1" width="140">
                <template scope="scope">
                    <el-input v-show="scope.row.edit" size="small" v-model="scope.row.transfer1Yuan"></el-input>
                    <span v-show="!scope.row.edit">{{ scope.row.transfer1Yuan }}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="转账金额2" width="140">
                <template scope="scope">
                    <el-input v-show="scope.row.edit" size="small" v-model="scope.row.transfer2Yuan"></el-input>
                    <span v-show="!scope.row.edit">{{ scope.row.transfer2Yuan }}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="转账金额3" width="140">
                <template scope="scope">
                    <el-input v-show="scope.row.edit" size="small" v-model="scope.row.transfer3Yuan"></el-input>
                    <span v-show="!scope.row.edit">{{ scope.row.transfer3Yuan }}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="转账金额4" width="140">
                <template scope="scope">
                    <el-input v-show="scope.row.edit" size="small" v-model="scope.row.transfer4Yuan"></el-input>
                    <span v-show="!scope.row.edit">{{ scope.row.transfer4Yuan }}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="转账金额5" width="140">
                <template scope="scope">
                    <el-input v-show="scope.row.edit" size="small" v-model="scope.row.transfer5Yuan"></el-input>
                    <span v-show="!scope.row.edit">{{ scope.row.transfer5Yuan }}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="转账金额6" width="140">
                <template scope="scope">
                    <el-input v-show="scope.row.edit" size="small" v-model="scope.row.transfer6Yuan"></el-input>
                    <span v-show="!scope.row.edit">{{ scope.row.transfer6Yuan }}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="差额" width="140">
                <template scope="scope">
                    <span>{{scope.row.diffYuan}}</span>
                </template>
            </el-table-column>

            <el-table-column align="center" label="备注" width="160">
                <template scope="scope">
                    <el-input v-show="scope.row.edit" size="small" v-model="scope.row.remark"></el-input>
                    <span v-show="!scope.row.edit">{{ scope.row.remark }}</span>
                </template>
            </el-table-column>

            <el-table-column  align="center" label="操作" width="140">
                <template scope="scope">
                    <el-button v-show='!scope.row.edit && scope.row.id!=0 && isAdminRole' type="primary" @click='scope.row.edit=true' size="small" icon="edit">编辑</el-button>
                    <el-button v-show='scope.row.edit' type="success" @click='handleUpdate(scope.row)' size="small" icon="check">完成</el-button>
                </template>

            </el-table-column>

        </el-table>

        <!-- 分页信息 -->
        <div v-show="!listLoading" class="pagination-container">
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </div>

        <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
            <el-form class="small-space" :model="temp" label-position="left" label-width="70px" style='width: 400px; margin-left:50px;' ref="tempForm">
                <el-form-item label="商户名称">
                    <el-select clearable class="filter-item" style="width: 130px" v-model="temp.userId" :disabled="!isAdminRole || dialogStatus=='update'">
                        <el-option v-for="item in userIdOptionsWithoutAll" :key="item.key" :label="item.display_name" :value="item.key">
                        </el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="应用名称">
                    <el-input v-model="temp.appName"></el-input>
                </el-form-item>

                <el-form-item label="回调地址">
                    <el-input v-model="temp.callbackUrl"></el-input>
                </el-form-item>

                <el-form-item label="返回地址">
                    <el-input v-model="temp.nodifyUrl"></el-input>
                </el-form-item>

                <el-form-item label="秘钥" v-if="dialogStatus=='update'">
                    <el-input v-model="temp.secretkey" :disabled="true"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button v-if="dialogStatus=='create'" type="primary" @click="create">确 定</el-button>
                <el-button v-else type="primary" @click="update">确 定</el-button>
            </div>
        </el-dialog>

    </div>
</template>

<script>
    /* eslint-disable no-unused-vars,indent,linebreak-style,prefer-const,padded-blocks,no-empty-function,linebreak-style,brace-style,no-trailing-spaces,dot-notation */

    import { parseTime, objectMerge } from 'utils';
    import store from 'store';

    import { Message } from 'element-ui';
    import { appList, appCreate, appUpdate } from 'api/financial/app_list'
    import { userListNoPage } from 'api/financial/user'
    import { getUidWithUndefined, isAdminRole } from 'src/utils/permission.js'

    import { payVerifyList, updatePayVerify } from 'api/financial/pay_verify'

    const pickerOptions2 = {
        shortcuts: [{
            text: '最近一天',
            onClick(picker) {
                const end = getZeroTime();
                const start = getZeroTime();
                start.setTime(getZeroTime() - 3600 * 1000 * 24 * 1);
                picker.$emit('pick', [start, end]);
            }
        },{
            text: '最近三天',
            onClick(picker) {
                const end = new getZeroTime();
                const start = getZeroTime();
                start.setTime(getZeroTime() - 3600 * 1000 * 24 * 3);
                picker.$emit('pick', [start, end]);
            }
        },{
            text: '最近7天',
            onClick(picker) {
                const end = getZeroTime();
                const start = getZeroTime();
                start.setTime(getZeroTime() - 3600 * 1000 * 24 * 7);
                picker.$emit('pick', [start, end]);
            }
        },{
            text: '最近一个月',
            onClick(picker) {
                const end = getZeroTime();
                const start = getZeroTime();
                start.setDate(1)
                picker.$emit('pick', [start, end]);
            }
        }]
    }

    export default {
        name: 'table_demo',
        data() {
            return {
                userName: store.getters.name,
                isAdminRole: isAdminRole(),
                list: null,
                total: null,
                listLoading: true,
                listTimeRange: [],
                pickerOptions2,
                listQuery: {
                    page: 1,
                    limit: 10,
                    search: {
                        userId_eq: undefined
                    }
                },
                tableKey: 0,
                dialogStatus: '',
                dialogFormVisible: false,
                textMap: {
                    update: '编辑',
                    create: '添加'
                },
                temp: {
                    userId: undefined
                },
                userIdOptions: [],
                userIdOptionsWithoutAll: []
            }
        },
        created() {
            this.getUserList()
            this.getList();
        },
        filters: {
            timeFilter(time, format) {
                if(time == undefined) {
                    return "---"
                }
                else if(time == 0) {
                    return 0
                }
                else {
                    return parseTime(time, format)
                }
            }

        },
        methods: {
            // 查询列表信息
            getList() {
                this.listLoading = true;

                let page = this.listQuery.page;
                let size = this.listQuery.limit;
                let search = this.listQuery.search;

                // 处理商户
                if ( typeof(this.listQuery.search.userId_eq) === 'undefined' && this.listQuery.search.userId_eq !== 0) {
                    this.listQuery.search.userId_eq = getUidWithUndefined()
                }

                // 处理时间
                search.verifyTime_gte = Date.parse(this.listTimeRange[0])/1000;
                search.verifyTime_lte = Date.parse(this.listTimeRange[1])/1000;


                payVerifyList(search, page, size).then(response => {
                    this.list = response.data.verifyVOPage.list.map(v => {
                        v.edit = false;
                        return v
                    });
                    this.total = response.data.verifyVOPage.total
                    this.listLoading = false;

                    let totalDiffRes = response.data.payVerifyTotalVO
                    let totalDiff = {
                        id:0,
                        userName: "合计",
                        totalMoneyYuan: totalDiffRes.totalMoneyTotal,
                        settlementMoneyYuan: totalDiffRes.settlementMoneyTotal,
                        transfer1Yuan: totalDiffRes.transfer1Total,
                        transfer2Yuan: totalDiffRes.transfer2Total,
                        transfer3Yuan: totalDiffRes.transfer3Total,
                        transfer4Yuan: totalDiffRes.transfer4Total,
                        transfer5Yuan: totalDiffRes.transfer5Total,
                        transfer6Yuan: totalDiffRes.transfer6Total,
                        diffYuan: totalDiffRes.diffTotal
                    }
                    this.list.push({id:0})
                    this.list.push(totalDiff)

                })

            },
            getUserList() {
                userListNoPage().then(response => {
                    if (response.status === 200) {
                        this.userIdOptions.push({ key: null, display_name: '全部' })
                        response.data.list.forEach(u => {
                            this.userIdOptions.push({ key: u.id, display_name: u.name })
                            this.userIdOptionsWithoutAll.push({ key: u.id, display_name: u.name })
                        })

                    }
                })
            },
            // 点击搜索按钮事件
            handleFilter() {
                this.getList();
            },

            // 分页处理
            handleSizeChange(val) {
                this.listQuery.limit = val;
                this.getList();
            },
            handleCurrentChange(val) {
                this.listQuery.page = val;
                this.getList();
            },

            // 编辑
            handleUpdate(row) {
                updatePayVerify(row).then(response => {
                    if (response.status == 200) {
                        Message({
                            message: '修改成功',
                            type: 'success',
                            duration: 2000
                        });
                        row.edit = false;
                        this.getList()
                    } else {
                        Message({
                            message: '修改失败',
                            type: 'warning',
                            duration: 2000
                        });
                        row.edit = false;
                    }
                })

            },
            update() {
                console.log(this.temp);
                appUpdate(this.temp).then(response => {
                    if (response.data === 1) {
                        Message({
                            message: '修该成功',
                            type: 'success',
                            duration: 2000
                        });
                        this.dialogFormVisible = false;
                        this.getList()
                    } else {
                        Message({
                            message: '修改失败',
                            type: 'warning',
                            duration: 2000
                        });
                    }
                })

                this.dialogFormVisible = false;
            },

            // 添加
            handleCreate() {
                this.temp = {
                    userId: getUidWithUndefined()
                }
                this.dialogStatus = 'create';
                this.dialogFormVisible = true;
            },
            create() {
                appCreate(this.temp).then(response => {
                    if (response.data === 1) {
                        Message({
                            message: '创建成功',
                            type: 'success',
                            duration: 2000
                        });
                        this.dialogFormVisible = false;
                        this.getList()
                    } else {
                        Message({
                            message: '创建失败',
                            type: 'warning',
                            duration: 2000
                        });
                    }
                })
            },

            // 导出excel
            handleDownload() {
                require.ensure([], () => {
                    const { export_json_to_excel } = require('vendor/Export2Excel');
                    const tHeader = ['对账时间', '商户名称', '结算金额', '转账金额1', '转账金额2', '转账金额3', '转账金额4', '转账金额5', '转账金额6', '差额'];
                    const filterVal = [
                        { name: 'verifyTime', filterFunction: parseTime },
                        { name: 'userName' },
                        { name: 'settlementMoneyYuan' },
                        { name: 'transfer1Yuan' },
                        { name: 'transfer2Yuan' },
                        { name: 'transfer3Yuan' },
                        { name: 'transfer4Yuan' },
                        { name: 'transfer5Yuan' },
                        { name: 'transfer6Yuan' },
                        { name: 'diffYuan' }
                    ];
                    const data = this.formatJson(filterVal, this.list);
                    export_json_to_excel(tHeader, data, '提现数据');
                })
            },
            formatJson(filterVal, jsonData) {
                let data = jsonData.map(v => filterVal.map(j => {
                    if (j['filterOptionsObj'] !== undefined) {
                        return j['filterOptionsObj'][v[j['name']]]
                    } else if (j['filterFunction'] !== undefined) {
                        let func = eval(j['filterFunction'])
                        return func(v[j['name']])
                    } else {
                        return v[j['name']]
                    }
                }));
                return data;
            }
        }
    }
</script>
