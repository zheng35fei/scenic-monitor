<template>
    <i-form ref="addForm" label-position="right" :model="formObj" :label-width="80" :rules="ruleValidate">
        <form-item label="应用IP" prop="appip">
            <Tooltip :content="ipTip" placement="right" style="width:90%;" :max-width="550">
                <i-input v-model="formObj.appip" placeholder="应用IP" :clearable="true"></i-input>
            </Tooltip>
        </form-item>
        <form-item label="端口号" prop="port">
            <input-number v-model="formObj.port" placeholder="端口号" type="number" :max="65535" :min="0"></input-number>
        </form-item>
        <row style="text-align: center">
            <i-col span="24">
                <i-button size="large" type="primary" @click="submitForm('addForm')" :loading="btnLoading">提交</i-button>
                <i-button size="large" @click="cancelForm('addForm')">取消</i-button>
            </i-col>
        </row>
    </i-form>
</template>
<script>
module.exports = {
    data() {
        return {
            formObj: {
                appip: "192.168.2.1",
                port: "80"
            },
            ruleValidate: {
                port: [
                    {
                        required: true,
                        message: "请填写端口号",
                        trigger: "blur"
                    },{
                        pattern: /^[0-9]*$/,
                        message: "请填写正确的端口号",
                        trigger: "blur"
                    }
                ],
                appip: [
                    {
                        required: true,
                        message: "请填写数据库地址",
                        trigger: "blur"
                    },
                    { 
                        pattern: /^(1\d{2}|2[0-4]\d|25[0-5]|[1-9]\d|[1-9])(\.(1\d{2}|2[0-4]\d|25[0-5]|[1-9]\d|\d)){3}$/,
                        message: "请填写正确的数据库Ip格式",
                        trigger: "blur" 
                    }
                ]
            },
            btnLoading: false,
            ipTip: '192.168.0.1'
        };
    },
    methods: {
        submitForm(name) {
            this.$refs[name]
                .validate(valid => {
                    if (valid) {
                        this.btnLoading = true
                        axios
                            .post("/createDataBase", this.formObj)
                            .then(res => {
                                this.$Notice.success({
                                    title: "成功",
                                    desc: res.message
                                });
                                this.$emit('close-dialog')
                                // 清空表单
                                this.$refs[name].resetFields();
                            })
                            .catch(error => {
                                this.$Notice.error({
                                    title: "失败",
                                    desc: error
                                });
                                // 实际使用删除true
                                if(true || error.message === 'force') {
                                    this.$Modal.confirm({
                                        title: '确认',
                                        content: '确认要提交吗?',
                                        onOk: () => {
                                            this.submitForm(name)
                                        }
                                    })
                                }
                                this.btnLoading = false
                            });
                    } else {
                        this.$Message.error("表单验证失败!");
                    }
                });
        },
        cancelForm(name) {
            this.$refs.addForm.resetFields();
            this.$emit("close-dialog");
        }
    }
};
</script>
