<template>
  <div id="vue-custom-form">
    <el-form :model="ObjformModel" ref="VueCustomForm" :label-width="ObjFormInfo.formLabelWidth" class="demo-dynamic" :rules="formCustomRules">
      <el-form-item v-for="item in ArrformFields" :key="item.type"
        :prop="item.prop"
        :label='item.label'
      >  
      <!-- input开始 -->  
        <el-input :type="item.inputType" v-model="ObjformModel[item.model]" :rows="item.rows" :disabled="item.disabled" :readonly="item.readonly" v-if="item.type =='input'"></el-input>
      <!-- input结束 -->

      <!-- radioGroup开始 -->
       <el-radio-group v-model="ObjformModel[item.model]" :disabled="item.disabled" v-if="item.type =='radios'">
        <el-radio v-for="radio in item.values" :key="radio.label" :label="radio.value">{{radio.label}}</el-radio>
      </el-radio-group>
      <!-- radioGroup结束 -->

      <!-- inputNumber开始 -->
        <el-input-number :min="item.min" v-model="ObjformModel[item.model]" :max="item.max" :disabled="item.disabled" :readonly="item.readonly" :controls="item.controls" style="width:100%;" v-if="item.type =='inputNumber'">
        </el-input-number>
      <!-- inputNumber结束 -->

        <!-- 权限popover开始 -->
        <div v-if="item.type == 'PermissionSelect'">

          <el-input v-model="ObjformModel[item.model]" :placeholder="item.placeholder" readonly>
             <el-button slot="append" icon="el-icon-edit" style="position:relative;">选择权限
                <el-select value-key="Permission"  multiple collapse-tags @change="permissionChange" v-model="Licence" style="opacity:0;position:absolute;right:21px;top:21px;z-index:100" >
            <el-option-group
              v-for="group in item.permissionMenu"
              :key="group.label"
              :label="group.label">
              <el-option
                v-for="child in group.children"
                :disabled="child.disabled"
                :key="child.permission"
                :label="child.label"
                :value="child.permission"
                >
              </el-option>
            </el-option-group>
          </el-select>
             </el-button>
          
          </el-input>
        </div>
        <!-- 权限pop over结束 -->
      
      </el-form-item>
      <el-form-item>
        <el-button  @click="resetForm">重置信息</el-button>
        <el-button type="primary" @click="submit">{{ObjFormInfo.submitButtonaName}}</el-button>
      </el-form-item>
</el-form>
  </div>
</template>

<script>
import QS from "qs";
// 导入验证的JS
import formValidator from "static/FormValidator.js"
export default {
  props: ["model", "fields", "formInfo"],
  data() {
    var validateLat = (rule, value, callback) => {
      console.log(11)
      
        if (value.toString().split(".")[1] !== undefined) {
          callback(new Error('纬度格式不符合规范'));
        }else {
          callback();
        }
        };
       var validateNormalRate = (rule, value, callback) => {
         console.log(22)
            //  判断是否为空
            if (value === '') {
                callback(new Error('内容不能为空'));
                //  判断是否等于某个值`
            } else if (Number(value) + Number(this.ObjformModel.ExperimentRate) != 100) {
                callback(new Error('成绩比例的和必须为100'));
            } else {
                callback();
            }
        }
    return {
      Licence: [],
      NumSum: 0,
      ObjformModel: {},
      ArrformFields: [],
      ObjFormInfo: {},
      ArrFormEvent: [],
      // 是否可以提交，true表示可以提交
      IsSubmit: true,
      // 发送表单参数
      FormActionParams:{
        
      },
      // 自定义验证规则
      formCustomRules:{
        NormalRate:[
          {validator: validateLat, trigger: 'blur' },
          {validator: validateNormalRate, trigger: 'blur' }
        ]
      }
    };
  },
  created() {
    // 重置状态
    this.ObjformModel = {}
    this.ArrformFields = []
    this.ObjFormInfo = {}
    this.ArrFormEvent = []
    // 接受新的值
    this.ObjformModel = this.model
    this.ArrformFields = this.fields;
    this.ObjFormInfo = this.formInfo
    this.ArrFormEvent = this.formInfo.formRules;
  
  },
  methods: {
    generateFormCustomRules(){
      var _this = this
      for (var i = 0; i < this.ArrFormEvent.length; i++) {
        if(this.formCustomRules[this.ArrFormEvent[i].field2] === undefined){
          this.formCustomRules[this.ArrFormEvent[i].field2] = []
        }
        this.formCustomRules[this.ArrFormEvent[i].field2].push(
          {
              validator:formValidator[this.ArrFormEvent[i].function](_this,this.ArrFormEvent[i].field1,this.ArrFormEvent[i].msg),
              trigger:'blur'
            }
        )    
      }
    },
    // 生成参数
    generateParameters() {
      this.FormActionParams.sys_table_name = this.ObjFormInfo.tableName,
      this.FormActionParams.sys_table_action = this.ObjFormInfo.formAction
      if (this.ObjFormInfo.formAction == "insert") {
        this.FormActionParams["sys_table_where"] = "true";
        this.ArrformFields.forEach((item, index) => {
          if(this.ObjformModel[item.model] === undefined){
              this.FormActionParams[item.model] = ""
          }else {
            this.FormActionParams[item.model] = this.ObjformModel[item.model];
          }
        });
      } else {
        this.ArrformFields.forEach((item, index) => {
          if (item.primaryKey) {
            if(this.ObjformModel[item.model] === undefined){
              this.FormActionParams[item.model] = ""
          }else {
            this.FormActionParams[item.model] = this.ObjformModel[item.model];
          }
          }
        });
        this.FormActionParams["sys_table_where"] = "true";
        this.ArrformFields.forEach((item, index) => {
          if (!item.primaryKey) {
            if(this.ObjformModel[item.model] === undefined){
              this.FormActionParams[item.model] = ""
          }else {
            this.FormActionParams[item.model] = this.ObjformModel[item.model];
          }
          }
        });
      }


    },
    // 发送表单请求
    sendSubmit() {
      if(this.ObjFormInfo.formAction == "query"){
            this.$emit("UpdataTable")
            return;
      }
      this.generateParameters();
      // 调用生成参数的函数
      this.$http.put(this.ObjFormInfo.formUrl, QS.stringify(this.FormActionParams)).then(res => {
        if(res.data.result == "OK"){
          this.$emit("UpdataTable")
          this.$message({
            type:'success',
            message:'更新成功',
            duration:1500
          })
        }else{
          this.$message({
            type:'error',
            message:'更新失败',
            duration:1500
          })
        }
      });
    },
    // 重置信息
    resetForm() {
      this.$refs.VueCustomForm.resetFields();
    },
    // 提交表单
    submit() {
      this.$refs.VueCustomForm.validate(valid => {
        if (valid) {
          // 表单每项通过验证
          // 验证表单
          this.validateForm();
         
        }
      });
    },
    // 调用表单的整体验证规则
    validateForm() {
      // 当表单验证存在的时候，验证表单数据
      if (this.ArrFormEvent.length) {
        // 触发表单事件
        for (var i = 0; i < this.ArrFormEvent.length; i++) {
          this[this.ArrFormEvent[i].function](
            this.ObjformModel[this.ArrFormEvent[i].field1],
            this.ObjformModel[this.ArrFormEvent[i].field2]
          );
          //  遇到一个错误则退出循环
          if (!this.IsSubmit) {
            return;
          } else {
            // 验证全部通过，调用发送请求
            this.sendSubmit();
          }
        }
      }else {
        // 不存在直接调用，发送请求
        this.sendSubmit(); 
      }
    },
    // 判断字符串是不是相等
    StringEqual(str1, str2) {
      if (str1 === "" || str2 === "") {
        this.IsSubmit = false;
        return;
      }

      if (str1 !== str2) {
        this.$message({
          type: "error",
          message: "两个元素的内容不一样",
          duration: 1500
        });
        return false;
      } else {
        return true;
      }
    },
    // 判断两个数是不是100
    SumNumber(num1, num2) {
      if (num1 === "" || num2 === "") {
        this.IsSubmit = false;
        this.$message({
          type: "error",
          message: "请输入正确的参数",
          duration: 1500
        });
        return;
      }
      if (Number(num1) + Number(num2) != 100) {
        this.IsSubmit = false;
        this.$message({
          type: "error",
          message: `${num1}和${num2}总和必须是100`,
          duration: 1500
        });
        return false;
      } else {
        return true;
      }
    },
    // 选择权限发生变化
    permissionChange() {
     
      this.NumSum = 0;
      this.Licence.forEach(item => {
        this.NumSum = this.NumSum ^ item;
      });
      this.ObjformModel.Permission = this.NumSum.toString(16);
    }
  },
  mounted(){
    this.resetForm();
  },
  watch: {
    model(val) {
      this.ObjformModel = val;
    },
    fields(val) {
      this.ArrformFields = val;
    },
    formInfo(val) {
      this.ObjFormInfo = val;
      this.ArrFormEvent = val.formRules;
      // this.generateFormCustomRules()
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
#vue-custom-form .el-input-number input {
  text-align: left;
}
.el-select .el-input {
  width: 110px;
  height: 36px;
  overflow: hidden;
}

.el-scrollbar {
  width: 158px;
}
</style>
