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
            btnLoading: false,
            columns: [
                {
                    title: "序号",
                    type: "index",
                    align: "center",
                    width: 60
                },
                {
                    title: "库链接地址",
                    key: "address",
                    align: "center",
                    width: 500
                },
                {
                    title: "用户名",
                    key: "username",
                    align: "center"
                },
                {
                    title: "密码",
                    key: "password",
                    align: "center"
                },
                {
                    title: "操作栏",
                    key: "action",
                    align: "center",
                    width: 140,
                    render: (h, params) => {
                        return [
                            h(
                                "Button",
                                {
                                    props: {
                                        type: "primary",
                                        size: "small",
                                        loading: params.row.btnLoading
                                    },
                                    attrs: {
                                        style: "margin-right: 6px"
                                    },
                                    on: {
                                        click: () => {
                                            this.$set(
                                                params.row,
                                                "btnLoading",
                                                true
                                            );
                                            this.testRow(params);
                                        }
                                    }
                                },
                                "测试"
                            ),
                            h(
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
                            )
                        ];
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
    mounted() {
        this.getList()
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
                .post("/datebaseGrid", params)
                .then(res => {
                    this.listData = res.data;
                    this.page.total = res.total
                })
                .catch(error => {
                    // 删除
                    this.listData = [
                        {
                            id: 0,
                            username: "John Brown",
                            password: 18,
                            address: "New York No. 1 Lake Park"
                        },
                        {
                            id: 1,
                            username: "Jim Green",
                            password: 24,
                            address: "London No. 1 Lake Park"
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
                        .post("/test", { id: rowData.id })
                        .then(res => {
                            this.$Notice.success({
                                title: "成功",
                                desc: res.message
                            });
                            this.$Modal.remove();
                        })
                        .catch(err => {
                            this.$Notice.error({
                                title: "失败",
                                desc: err
                            });
                            this.$Modal.remove();
                        });
                }
            });
        },
        // 测试当前行
        testRow(column) {
            const rowData = column.row;
            this.btnLoading = true;
            axios
                .post("/test", { id: rowData.id })
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
                })
                .then(() => {
                    rowData.btnLoading = false;
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
.pageWrap {
    margin-top: 10px;
}
</style>

