<template>
    <layout>
        <content>
            <i-form ref="gatesForm" :model="points" :rules="ruleValidate" :label-width="80">
                <div class="nameBox">
                    <form-item label="景点名称" prop="pointName">
                        <i-input
                            v-model="points.pointName"
                            placeholder="请填写景点名称"
                            style="width:280px;"
                            :clearable="true"
                        ></i-input>
                    </form-item>
                </div>
                <div class="cardList">
                    <Card class="gateItem" v-for="(item, index) in points.gates" :key="index">
                        <div class="delIcon">
                            <Icon type="md-close" @click="delGate(index)"/>
                        </div>
                        <form-item
                            label="闸机编号："
                            :prop="'gates.' + index + '.gateNo'"
                            :label-width="90"
                            :rules="[{required: true, message: '请填写闸机编号', trigger: 'blur'}, {pattern: /^[0-9]*$/, message: '请填写数字', trigger: 'blur'}]"
                        >
                            <i-input size="large" v-model="item.gateNo" placeholder="闸机编号" :clearable="true"></i-input>
                        </form-item>
                        <form-item
                            label="闸机IP"
                            :prop="'gates.' + index + '.gateip'"
                            :label-width="90"
                            :rules="[{required: true, message: '请填写闸机IP', trigger: 'blur'}, {pattern: /^(1\d{2}|2[0-4]\d|25[0-5]|[1-9]\d|[1-9])(\.(1\d{2}|2[0-4]\d|25[0-5]|[1-9]\d|\d)){3}$/, message: '请填写正确的IP', trigger: 'blur'}]"
                        >
                            <i-input size="large" v-model="item.gateip" placeholder="闸机IP" :clearable="true"></i-input>
                        </form-item>
                    </Card>
                    <Card class="gateItem addItem">
                        <div class="addBox" @click="addGate">
                            <div>
                                <Icon type="md-add" class="addIcon"/>
                            </div>
                            <span>添加</span>
                        </div>
                    </Card>
                </div>
            </i-form>
        </content>
    </layout>
</template>
<script>
module.exports = {
    data() {
        return {
            isEdit: false,
            points: {
                pointName: "",
                gates: [
                    {
                        gateNo: "",
                        gateip: ""
                    }
                ]
            },
            tempName: "",
            ruleValidate: {
                pointName: [
                    {
                        required: true,
                        message: "请填写景点名称",
                        trigger: "blur"
                    }
                ]
            }
        };
    },
    methods: {
        // 提交闸机数据
        submitForm() {
            this.$refs.gatesForm.validate(valid => {
                if (valid) {
                    axios
                        .post("/postGates", this.points)
                        .then(res => {
                            this.$Notice.success({
                                title: "成功",
                                content: res.message
                            });
                            this.$emit("close-dialog");
                            // 清空表单
                            this.$refs.gatesForm.resetFields();
                        })
                        .catch(err => {
                            this.$Notice.error({
                                title: "失败",
                                content: err
                            });
                        })
                        .then(() => {
                            this.$emit("save-finish");
                        });
                } else {
                    this.$emit("save-finish");
                }
            });
        },
        // 添加闸机
        addGate() {
            this.points.gates.push({
                gateNo: "",
                gateip: ""
            });
        },
        // 删除闸机
        delGate(i) {
            this.points.gates.splice(i, 1);
        }
    }
};
</script>
<style scoped>
.editNameIcon {
    margin-left: 5px;
    font-size: 18px;
    cursor: pointer;
}
.nameBox {
    padding: 0 26px;
    display: flex;
    align-items: center;
}
.nameBox h3 {
    font-size: 18px;
}
.nameBox .ivu-form-item {
    margin-bottom: 0;
}
.cardList {
    display: flex;
    justify-content: flex-start;
    flex-wrap: wrap;
    padding: 20px;
}
.gateItem {
    width: 320px;
    margin: 0 5px 10px;
    padding: 10px 0; 
}
.gateItem .ivu-form-item {
    margin-bottom: 20px;
}
.gateItem .ivu-form-item:last-child {
    margin-bottom: 0;
}
.item-input {
    margin-bottom: 10px;
    display: flex;
}
.item-input:last-child {
    margin-bottom: 0;
}
.item-input label {
    flex: 0 0 80px;
    line-height: 32px;
}
.ivu-card .delIcon {
    display: none;
    width: 20px;
    height: 20px;
    line-height: 20px;
    position: absolute;
    right: 2px;
    top: 2px;
    cursor: pointer;
}
.ivu-card:hover .delIcon {
    display: block;
    font-size: 20px;
}
.ivu-card.addItem {
    display: flex;
    align-items: center;
    justify-content: center;
}
.addBox {
    display: block;
    flex-direction: column;
    border-radius: 50%;
    width: 78px;
    height: 78px;
    padding-top: 25px;
    background-color: #f5f5f5;
    text-align: center;
    line-height: 60px;
    color: #3476f1;
    cursor: pointer;
}
.addBox span {
    display: block;
    font-size: 12px;
    height: 16px;
    line-height: 16px;
}
.ivu-card .addIcon {
    display: block;
    font-size: 12px;
    cursor: pointer;
    margin-bottom: 2px;
    font-weight: bold;
}
</style>
