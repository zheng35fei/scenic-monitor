<template>
    <i-form
        ref="addForm"
        label-position="right"
        :model="formObj"
        :rules="ruleValidate"
        :label-width="90"
    >
        <form-item label="库类型" prop="sqlType">
            <i-select v-model="formObj.sqlType" transfer>
                <i-option
                    v-for="item in sqlTypeList"
                    :value="item.value"
                    :key="item.value"
                >{{item.label}}</i-option>
            </i-select>
        </form-item>
        <form-item label="库链接地址" prop="databaseAddr">
            <Tooltip
                :content="ipTip[formObj.sqlType]"
                placement="right"
                style="width:100%;"
                :max-width="550"
            >
                <i-input v-model="formObj.databaseAddr" placeholder="请填写库链接地址" :clearable="true"></i-input>
            </Tooltip>
        </form-item>
        <form-item label="用户名" prop="username">
            <i-input v-model="formObj.username" placeholder="请填写用户名" :clearable="true"></i-input>
        </form-item>
        <form-item label="密码" prop="password">
            <i-input
                type="password"
                v-model="formObj.password"
                placeholder="请填写密码"
                :clearable="true"
            ></i-input>
        </form-item>
        <row style="text-align: center">
            <i-col span="24">
                <i-button
                    size="large"
                    type="primary"
                    @click="submitForm('addForm')"
                    :loading="btnLoading"
                >提交</i-button>
                <i-button size="large" @click="cancelForm('addForm')">取消</i-button>
            </i-col>
        </row>
    </i-form>
</template>
<script>
module.exports = {
    data() {
        return {
            sqlTypeList: [
                {
                    value: "oracle",
                    label: "ORACLE"
                },
                {
                    value: "mysql",
                    label: "MySQL"
                },
                {
                    value: "sqlserver",
                    label: "SQLServer"
                }
            ],
            formObj: {
                sqlType: "oracle",
                databaseAddr: "192.168.1.1",
                username: "wqdwq",
                password: "213123"
            },
            ruleValidate: {
                sqlType: [
                    {
                        required: true,
                        message: "请选择数据库类型",
                        trigger: "blur"
                    }
                ],
                databaseAddr: [
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
                ],
                username: [
                    { required: true, message: "请填写用户名", trigger: "blur" }
                ],
                password: [
                    { required: true, message: "请填写密码", trigger: "blur" }
                ]
            },
            btnLoading: false,
            ipTip: {
                oracle: "jdbc:oracle:thin:@localhost:1521:orcl",
                mysql: "jdbc:mysql://localhost:3306/example?serverTimezone=UTC",
                sqlserver:
                    "jdbc:sqlserver://localhost:1433;DatabaseName=example"
            }
        };
    },
    methods: {
        // 提交
        submitForm(name) {
            this.$refs[name].validate(valid => {
                if (valid) {
                    this.btnLoading = true;
                    axios
                        .post("/createDataBase", this.formObj)
                        .then(res => {
                            this.$Notice.success({
                                title: "成功",
                                desc: res.message
                            });
                            this.$emit("close-dialog");
                            // 清空表单
                            this.$refs[name].resetFields();
                        })
                        .catch(error => {
                            this.$Notice.error({
                                title: "失败",
                                desc: error
                            });
                            if (true || error.message === "force") {
                                this.$Modal.confirm({
                                    title: "确认",
                                    content: "确认要提交吗?",
                                    onOk: () => {
                                        this.submitForm(name);
                                    }
                                });
                            }
                        })
                        .then(() => {
                            this.btnLoading = false;
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
