<template>
    <div class="layout">
        <layout>
            <header style="height:40px"></header>
            <content style="width: 70%; margin: 0 auto;">
                <Row type="flex" justify="space-between">
                    <i-col span="16">
                        <h3
                            v-if="!isEdit && scenicTitleName"
                            style="font-size:24px; font-weight: 400; display:inline-block;"
                            @click="nameEdit"
                        >{{scenicTitleName}}</h3>
                        <Row v-else :gutter="4">
                            <i-col span="16">
                                <i-input
                                    ref="scenicTitle"
                                    placeholder="请填写景区名称"
                                    :clearable="true"
                                    v-model="scenicTitleName"
                                    @on-focus="isEdit = true"
                                ></i-input>
                            </i-col>
                            <i-col span="8">
                                <i-button
                                    v-if="!!scenicTempName"
                                    type="primary"
                                    ghost
                                    @click="nameReset"
                                >重置</i-button>
                            </i-col>
                        </Row>
                    </i-col>
                    <i-col span="8">
                        <Row type="flex" justify="end">
                            <i-col>
                                <i-button
                                    v-if="!isEdit && scenicTitleName"
                                    size="large"
                                    type="primary"
                                    ghost
                                    @click="nameEdit"
                                >编辑</i-button>
                                <i-button
                                    v-else
                                    size="large"
                                    type="primary"
                                    ghost
                                    @click="saveScenicName"
                                >保存</i-button>
                                <i-button
                                    size="large"
                                    icon="ios-add"
                                    type="primary"
                                    ghost
                                    @click="modalForm = true"
                                    :disabled="!nameSaved"
                                >添加</i-button>
                            </i-col>
                        </Row>
                    </i-col>
                </Row>
                <br>
                <Tabs type="card" :name="curTabName" :animated="false" @on-click="selectTab">
                    <tab-pane
                        :label="item.renderFun"
                        :name="item.name"
                        v-for="item in tabOptions"
                        :key="item.name"
                    >
                        <component :ref="item.tableComponent" :is="item.tableComponent"></component>
                    </tab-pane>
                </Tabs>
            </content>
            <footer></footer>
        </layout>

        <Modal ref="addModal" v-model="modalForm" :width="modalWidth" :closable="false" footer-hide scrollable :fullscreen="isFullscreen">
            <template slot="header">
                <header class="modalTitle">
                    <template v-if="!isGate">{{modelTitleName}}</template>
                    <template v-else>
                        <row type="flex" justify="space-between" style="margin:0 24px">
                            <i-col span="16" style="text-align: left; display:flex; align-items:center;">
                                <h3 class="titleName">闸机监控项</h3>
                                <Icon type="ios-expand" @click="isFullscreen = !isFullscreen" style="margin-left:8px; cursor: pointer"/>
                            </i-col>
                            <i-col span="8" style="text-align: right">
                                <i-button type="primary" @click="submitGateList" :loading="btnLoading">保存提交</i-button>
                            </i-col>
                        </row>
                    </template>
                </header>
            </template>
            <component ref="addForm" :is="currentFormComponent" @close-dialog="closeModal" @save-finish="saveDone"></component>
        </Modal>
    </div>
</template>

<script>
module.exports = {
    components: {
        databaseTable: httpVueLoader("./database-table.vue"),
        portsTable: httpVueLoader("./ports-table.vue"),
        gatesTable: httpVueLoader("./gates-table.vue"),
        databaseForm: httpVueLoader("./database-form.vue"),
        portsForm: httpVueLoader("./ports-form.vue"),
        gatesForm: httpVueLoader("./gates-form.vue")
    },
    data() {
        return {
            curTabName: 'database',
            modalForm: false,
            scenicTitleName: "景区名称",
            scenicTempName: "",
            nameSaved: false,
            tabNames: [
                {
                    name: "database",
                    icon: "ios-list-box-outline",
                    label: "数据库监控",
                    tableComponent: "databaseTable"
                },
                {
                    name: "ports",
                    icon: "ios-qr-scanner",
                    label: "端口监控",
                    tableComponent: "portsTable"
                },
                {
                    name: "gates",
                    icon: "ios-barcode-outline",
                    label: "闸机监控",
                    tableComponent: "gatesTable"
                }
            ],
            isEdit: false,
            currentFormComponent: "databaseForm",
            isFullscreen: false,
            btnLoading: false
        };
    },
    created() {
        this.nameSaved = !!this.scenicTitleName
    },
    watch: {
        'curTabName'() {
            this.getCurTabList();
        }
    },
    computed: {
        // 自定义label结构
        tabOptions: function() {
            let arr = [];
            this.tabNames.forEach(item => {
                arr.push({
                    name: item.name,
                    tableComponent: item.tableComponent,
                    renderFun: h => {
                        return h("div", [
                            h("Icon", {
                                attrs: {
                                    type: item.icon
                                }
                            }),
                            h("p", item.label)
                        ]);
                    }
                });
            });
            return arr;
        },
        // 弹出框宽度
        modalWidth() {
            return this.currentFormComponent === "gatesForm" ? 960 : 420;
        },
        modelTitleName() {
            return (
                this.tabNames.filter(item =>
                    this.currentFormComponent.includes(item.name)
                )[0].label + "项"
            );
        },
        isGate() {
            return (
                this.currentFormComponent !== "databaseForm" &&
                this.currentFormComponent !== "portsForm"
            );
        }
    },
    methods: {
        // 查询当前tab的列表
        getCurTabList() {
            this.$refs[this.curTabName+ 'Table'][0].getList()
        },
        // 组件内部关闭弹层
        closeModal() {
            this.$refs.addModal.cancel();
        },
        // 修改当前添加form组件
        selectTab(name) {
            this.currentFormComponent = name + "Form";
            this.curTabName = name
        },
        // 展开名称编辑
        nameEdit() {
            this.isEdit = true;
            this.scenicTempName = this.scenicTitleName;
        },
        // 景区名称重置
        nameReset() {
            if (this.scenicTempName) {
                this.isEdit = false;
                this.scenicTitleName = this.scenicTempName;
            }
        },
        // 保存景区名称
        saveScenicName() {
            // 未做修改不提交
            if (!this.scenicTitleName) {
                this.$Message.warning({
                    content: "请填写景区名称",
                    duration: 3
                });
                return;
            }
            if (this.scenicTitleName === this.scenicTempName) {
                this.$Message.warning({
                    content: "请修改景区名称",
                    duration: 3
                });
                return;
            }
            const params = {
                name: this.scenicTitleName
            };
            // 提交保存， 失败回复之前名称；成功改成新值
            axios
                .post("/saveName", params)
                .then(res => {
                    this.isEdit = false;
                    this.scenicTitleName = this.scenicTempName;
                    this.nameSaved = true
                })
                .catch(err => {
                    this.$Notice.error({
                        title: "失败",
                        desc: err
                    });
                    this.scenicTitleName = this.scenicTempName || this.scenicTitleName;
                });
        },
        // 提交闸机列表
        submitGateList() {
            this.btnLoading = true;
            this.$refs.addForm.submitForm()
        },
        // 组件内部保存完毕
        saveDone() {
            this.btnLoading = false;
            this.getCurTabList();
        }
    }
};
</script>

<style>
.ivu-layout {
    background-color: #fff;
}

.ivu-tabs-bar {
    margin-bottom: 4px;
}
.ivu-tabs.ivu-tabs-card > .ivu-tabs-bar .ivu-tabs-nav-container {
    height: auto;
}
.ivu-tabs-nav {
    display: flex;
    justify-content: space-between;
    width: 100%;
}
.ivu-tabs.ivu-tabs-card > .ivu-tabs-bar .ivu-tabs-tab {
    flex: 1;
    height: auto;
    border: none;
    border-radius: 0;
    background-color: #f6f6f6;
    text-align: center;
    padding: 25px 16px;
}
.ivu-tabs.ivu-tabs-card > .ivu-tabs-bar .ivu-tabs-tab-active {
    background-color: #3476f1;
    color: #fff;
}
.ivu-tabs.ivu-tabs-card > .ivu-tabs-bar .ivu-tabs-tab .ivu-icon {
    font-size: 32px;
    width: 32px;
    height: 32px;
    margin-right: 0;
}
.ivu-tabs.ivu-tabs-card > .ivu-tabs-bar .ivu-tabs-tab p {
    margin-top: 4px;
}
.modalTitle {
    padding: 4px 0;
    font-weight: bold;
    font-size: 15px;
    text-align: center;
    color: #333;
}
.titleName {
    height:32px;
    line-height: 32px;
}
</style>