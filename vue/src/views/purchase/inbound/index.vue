<template>
    <el-card v-loading="config.operating">
        <search-form>
            <search-form-item label="单 号：">
                <el-input v-model="searchForm.id_fuzzy" clearable maxlength="50"/>
            </search-form-item>
            <search-form-item label="采购订单：">
                <el-input v-model="searchForm.pid_fuzzy" clearable maxlength="50"/>
            </search-form-item>
            <search-form-item label="创建人：">
                <el-input v-model="searchForm.cname" clearable maxlength="50"/>
            </search-form-item>
            <search-form-item label="创建时间：">
                <el-date-picker
                        v-model="temp.ctime"
                        format="yyyy-MM-dd"
                        range-separator="-"
                        type="daterange"
                        value-format="timestamp"
                />
            </search-form-item>
            <search-form-item label="审核人：">
                <el-input v-model="searchForm.vname" clearable maxlength="50"/>
            </search-form-item>
            <search-form-item label="审核时间：">
                <el-date-picker
                        v-model="temp.vtime"
                        format="yyyy-MM-dd"
                        range-separator="-"
                        type="daterange"
                        value-format="timestamp"
                />
            </search-form-item>
            <search-form-item label="状 态：">
                <el-select v-model="temp.status" clearable multiple>
                    <el-option :value="0" label="拟定"/>
                    <el-option :value="1" label="待审核"/>
                    <el-option :value="2" label="已审核"/>
                </el-select>
            </search-form-item>
        </search-form>
        <el-row>
            <el-button icon="el-icon-search" size="small" type="success" @click="search">查 询</el-button>
            <el-button v-if="canAdd" icon="el-icon-plus" size="small" type="primary" @click="add">添 加</el-button>
            <el-button icon="el-icon-view" size="small" type="primary" @click="see">查 看</el-button>
            <el-button v-if="canUpdate" icon="el-icon-edit" size="small" type="primary" @click="edit">编 辑</el-button>
            <el-button v-if="canDel" icon="el-icon-delete" size="small" type="danger" @click="del">删 除</el-button>
            <el-button v-if="canExport" icon="el-icon-download" size="small" type="info" @click="downloadExcel">导 出
            </el-button>
        </el-row>
        <el-row v-loading="config.loading" class="table-container">
            <el-table
                    ref="table"
                    :data="tableData"
                    current-row-key="id"
                    highlight-current-row
                    row-key="id"
                    @row-click="row=$event"
                    @expand-change="getSubList"
            >
                <el-table-column align="center" type="expand">
                    <template v-slot="{row}">
                        <liner-progress :show="row._loading"/>
                        <el-table v-show="!row._loading" :data="row.data" border row-key="id">
                            <el-table-column align="center" label="#" type="index" width="80"/>
                            <el-table-column align="center" label="商品" prop="cname" show-overflow-tooltip/>
                            <el-table-column align="center" label="入库数量" prop="num"/>
                        </el-table>
                    </template>
                </el-table-column>
                <el-table-column align="center" label="#" type="index" width="80"/>
                <el-table-column align="center" label="单 号" prop="id" width="160" show-overflow-tooltip/>
                <el-table-column align="center" label="创建人" prop="cname" show-overflow-tooltip/>
                <el-table-column align="center" label="创建时间" width="150" show-overflow-tooltip>
                    <template v-slot="{row}">{{row.ctime | timestamp2Date}}</template>
                </el-table-column>
                <el-table-column align="center" label="审核人" prop="vname" show-overflow-tooltip/>
                <el-table-column align="center" label="审核时间" width="150" show-overflow-tooltip>
                    <template v-slot="{row}">{{row.vtime | timestamp2Date}}</template>
                </el-table-column>
                <el-table-column align="center" label="状 态" width="120">
                    <template v-slot="{row}">
                        <span :class="{primary:row.status===1,success:row.status===2}" class="dot"/>
                        {{getStatus(row.status)}}
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
                    background
                    :current-page="searchForm.page"
                    :page-size="searchForm.pageSize"
                    :total="searchForm.total"
                    layout="total, prev, pager, next, jumper"
                    @current-change="pageChange"
            />
        </el-row>

        <edit-dialog
                ref="editDialog"
                v-model="editDialog"
                :base-url="baseUrl"
                :id="row?row.id:null"
                :type.sync="type"
                @search="search"
        />
    </el-card>
</template>

<script>
    import EditDialog from './components/EditDialog'
    import SearchForm from '@/bizComponents/SearchForm'
    import SearchFormItem from "@/bizComponents/SearchForm/SearchFormItem"
    import documentTableMixin from '@/mixins/bizDocumentTableMixin'
    import {del, getSubById, search} from "@/api/purchase/inbound"

    export default {
        name: "purchaseInbound",
        mixins: [documentTableMixin],
        components: {EditDialog, SearchForm, SearchFormItem},
        data() {
            return {
                baseUrl: '/purchase/inbound',
                api: {search, del, getSubById},
                searchForm: {
                    pid_fuzzy: null
                },
                excel: {
                    column: [
                        {header: '序号', alias: 'id'},
                        {header: '采购订单单号', alias: 'pid', width: 30},
                        {header: '创建人', alias: 'cname'},
                        {header: '创建时间', alias: 'ctime'},
                        {header: '审核人', alias: 'vname'},
                        {header: '审核时间', alias: 'vtime'},
                        {header: '状态', alias: 'status'},
                        {header: '备注', alias: 'remark', width: 50}
                    ]
                },
            }
        },
        methods: {
            mergeSearchForm() {
                return {
                    ...this.searchForm,
                    status: this.temp.status.join(','),
                    ctimeStart: this.temp.ctime ? this.temp.ctime[0] : null,
                    ctimeEnd: this.temp.ctime ? this.temp.ctime[1] + 86400000 : null,
                    vtimeStart: this.temp.vtime ? this.temp.vtime[0] : null,
                    vtimeEnd: this.temp.vtime ? this.temp.vtime[1] + 86400000 : null,
                }
            }
        }
    }
</script>
