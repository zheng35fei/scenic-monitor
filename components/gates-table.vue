<template>
    <div>
        <i-table :columns="columns" :loading="tableLoading" :data="listData" :stripe="true"></i-table>
        <row type="flex" justify="end" class-name="pageWrap" >
            <i-col>
                <Page :total="page.total" :page-size="page.pageSize" show-elevator show-sizer show-total @on-change="changeSize" />
            </i-col>
        </row>
    </div>
</template>

<script>
module.exports = {
    data() {
        return {
            columns: [
                {
                    title: "序号",
                    type: "index",
                    align: "center",
                    width: 60
                },
                {
                    title: "景点名称",
                    key: "pointName",
                    align: "center",
                    width: 300
                },
                {
                    title: "闸机编号",
                    key: "gateNo",
                    align: "center",
                    width: 100
                },
                {
                    title: "闸机IP",
                    key: "gateip",
                    align: "center"
                },
                {
                    title: "操作栏",
                    key: "action",
                    align: "center",
                    width: 100,
                    render: (h, params) => {
                        return h(
                            "span",
                            {
                                attrs: {
                                    class: "btn-del-box"
                                },
                                on: {
                                    click: () => {
                                        this.delRow(params);
                                    }
                                }
                            },
                            [
                                h("Icon", {
                                    attrs: {
                                        type: "ios-trash-outline"
                                    }
                                })
                            ]
                        );
                    }
                }
            ],
            listData: [],
            tableLoading: false,
            // 分页字段
            page: {
                total: 4,
                pageSize: 2,
                curpage: 1
            }
        };
    },
    methods: {
        changeSize(cur) {
            this.page.curpage = cur
            this.getList('page')
        },
        // 列表数据
        getList(from) {
            if(this.listData.length > 0 && !from) return
            this.tableLoading = true;
            const params = {
                current: this.page.curpage
            }
            axios
                .post("/gatesGrid", params)
                .then(res => {
                    this.listData = res.data;
                })
                .catch(error => {
                    // 删除下面数据赋值
                    this.listData = [
                        {
                            id: 0,
                            pointName: "John Brown",
                            gateNo: 18,
                            gateip: "New York No. 1 Lake Park"
                        },
                        {
                            id: 1,
                            pointName: "Jim Green",
                            gateNo: 24,
                            gateip: "London No. 1 Lake Park"
                        },
                        {
                            id: 2,
                            pointName: "Joe Black",
                            gateNo: 30,
                            gateip: "Sydney No. 1 Lake Park"
                        },
                        {
                            id: 3,
                            pointName: "Jon Snow",
                            gateNo: 26,
                            gateip: "Ottawa No. 2 Lake Park"
                        }
                    ];
                    this.$Notice.error({
                        title: "失败",
                        desc: error
                    });
                })
                .then(() => {
                    this.tableLoading = false;
                });
        },
        /**
         * 删除当前行
         * @param column 选中行
         */
        delRow(column) {
            const rowData = column.row;
            this.$Modal.confirm({
                title: "删除",
                content: `库链接地址：${rowData.address}<br>用户名：${
                    rowData.username
                }`,
                loading: true,
                scrollable: true,
                okText: "确认删除",
                onOk: () => {
                    axios
                        .post("/aa", { id: rowData.id })
                        .then(res => {
                            this.$Notice.success({
                                title: "成功",
                                desc: res.message
                            });
                            this.$Modal.remove();
                        })
                        .catch(err => {
                            console.log("error", err);
                            this.$Notice.error({
                                title: "失败",
                                desc: err
                            });
                            this.$Modal.remove();
                        });
                }
            });
        }
    }
};
</script>

<style scope>
.btn-del-box {
    display: inline-block;
    border-radius: 50%;
    background-color: #eee;
    color: #666;
    font-size: 20px;
    width: 28px;
    height: 28px;
    line-height: 26px;
    cursor: pointer;
}
.btn-del-box:hover {
    background-color: #eaf1fe;
    color: #9ebdf8;
}
.ivu-table-stripe .ivu-table-body tr.ivu-table-row-hover td,
.ivu-table-stripe .ivu-table-fixed-body tr.ivu-table-row-hover td {
    background-color: #dbdfe2;
}
</style>

