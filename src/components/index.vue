<template>
  <div>
    <div class="topContent">
      <span>话术组件</span>
      <div class="topButton">
        <el-button type="primary" @click="onSubmit">保存</el-button>
        <el-button type="primary" @click="noSubmit">取消</el-button>
      </div>
    </div>

    <el-row class="warp">
      <!--
      Form 组件提供了表单验证的功能，只需要通过 rule 属性传入约定的验证规则，并 Form-Item 的 prop 属性设置为需校验的字段名即可。具体可以参考官网：http://element.eleme.io/#/zh-CN/component/form
      -->
      <el-col :span="24" class="warp-main">
        <el-form ref="infoForm" :model="messageTemplate" :rules="rules">
          <el-form-item label="控件区：" style="width: 300px">
            <ul class="leftZJ" @start="getdata" @end="datadragEnd">
              <li
                v-for="(val, index) in tags"
                draggable
                @mouseout="mouseOut($event, val)"
                :data-type="val.id"
                :key="index + 1"
              >
                <img :src="val.icon" />
                <p>{{ val.name }}</p>
              </li>
            </ul>
          </el-form-item>
          <el-form-item label="画布区：" style="width: 620px; height: 900px">
            <div
              class="edit_container"
              @drop="drog"
              @dragover="dragOver($event)"
            >
              <quill-editor
                v-model="newText"
                style="width: 600px; height: 300px"
                :options="editorOption"
                ref="editer"
                @focus="onEditorFocus"
              >
                <!-- 自定义toolar 代码优化部分-->
                <div id="toolbar" slot="toolbar">
                  <!-- Add a bold button -->
                  <button class="ql-bold" title="加粗">Bold</button>
                  <button class="ql-italic" title="斜体">Italic</button>
                  <button class="ql-underline" title="下划线">underline</button>
                  <button class="ql-strike" title="删除线">strike</button>
                  <button class="ql-blockquote" title="引用"></button>
                  <button class="ql-code-block" title="代码"></button>
                  <button class="ql-header" value="1" title="标题1"></button>
                  <button class="ql-header" value="2" title="标题2"></button>
                  <!--Add list -->
                  <button
                    class="ql-list"
                    value="ordered"
                    title="有序列表"
                  ></button>
                  <button
                    class="ql-list"
                    value="bullet"
                    title="无序列表"
                  ></button>
                  <!-- Add font size dropdown -->
                  <select class="ql-header" title="段落格式">
                    <option selected>段落</option>
                    <option value="1">标题1</option>
                    <option value="2">标题2</option>
                    <option value="3">标题3</option>
                    <option value="4">标题4</option>
                    <option value="5">标题5</option>
                    <option value="6">标题6</option>
                  </select>
                  <select class="ql-size" title="字体大小">
                    <option value="10px">10px</option>
                    <option value="12px">12px</option>
                    <option value="14px">14px</option>
                    <option value="16px" selected>16px</option>
                    <option value="18px">18px</option>
                    <option value="20px">20px</option>
                  </select>
                  <select class="ql-font" title="字体">
                    <option value="SimSun">宋体</option>
                    <option value="SimHei">黑体</option>
                    <option value="Microsoft-YaHei">微软雅黑</option>
                    <option value="KaiTi">楷体</option>
                    <option value="FangSong">仿宋</option>
                    <option value="Arial">Arial</option>
                  </select>
                  <!-- Add subscript and superscript buttons -->
                  <select
                    class="ql-color"
                    value="color"
                    title="字体颜色"
                  ></select>
                  <select
                    class="ql-background"
                    value="background"
                    title="背景颜色"
                  ></select>
                  <select class="ql-align" value="align" title="对齐"></select>
                  <button class="ql-clean" title="还原"></button>
                  <!-- You can also add your own -->
                </div>
              </quill-editor>
            </div>
          </el-form-item>

          <div class="inputDiv" style="width: 300px">
            <p>编辑区：</p>
            <el-form-item label="控件名称" label-width="120px" prop="a_title">
              <el-input v-model="messageTemplate.templateName"></el-input>
            </el-form-item>
            <el-form-item label-width="120px" v-show="show" label="报表名称">
              <el-input
                placeholder="输入关键字进行过滤"
                @focus="treeSelects"
                v-model="filterText"
                :suffix-icon="
                  treeSelect ? 'el-icon-arrow-up' : 'el-icon-arrow-down'
                "
              >
              </el-input>

              <el-tree
                class="filter-tree"
                :data="data"
                v-show="treeSelect"
                highlight-current
                @node-click="handleNodeClick"
                :props="defaultProps"
                default-expand-all
                :filter-node-method="filterNode"
                ref="tree"
              >
              </el-tree>
            </el-form-item>
            <el-form-item label-width="120px" v-show="show" label="指标名称">
              <el-select
                v-model="messageTemplate.senderName"
                placeholder="请选择"
                @change="changeSelect($event)"
                multiple
              >
                <el-option
                  label="当日核保保费"
                  value="当日核保保费"
                ></el-option>
                <el-option
                  label="当月核保保费"
                  value="当月核保保费"
                ></el-option>
                <el-option
                  label="当季核保保费"
                  value="当季核保保费"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label-width="120px" v-show="!show" label="统计日期">
              <el-select
                v-model="messageTemplate.timeData"
                placeholder="请选择"
                @change="changeSelect($event)"
              >
                <el-option
                  label="当前日期上月"
                  value="当前日期前上月"
                ></el-option>
                <el-option label="当月" value="当月"></el-option>
                <el-option
                  label="当前日期前一天"
                  value="当前日期前一天"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label-width="120px" label="渠道" v-show="show">
              <el-select
                v-model="messageTemplate.channelSelect"
                placeholder="请选择"
              >
                <el-option label="合计" value="合计"></el-option>
                <el-option label="新渠道" value="新渠道"></el-option>
                <el-option label="电销" value="电销"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label-width="120px" label="险种" v-show="show">
              <el-select
                v-model="messageTemplate.orgSelect"
                placeholder="请选择"
                @change="changeSelect($event)"
              >
                <el-option label="合计" value="合计"></el-option>
                <el-option label="新渠道" value="新渠道"></el-option>
                <el-option label="电销" value="电销"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label-width="120px" label="个团" v-show="show">
              <el-select
                v-model="messageTemplate.content"
                placeholder="请选择"
                @change="changeSelect($event)"
              >
                <el-option label="合计" value="合计"></el-option>
                <el-option label="新渠道" value="新渠道"></el-option>
                <el-option label="电销" value="电销"></el-option>
              </el-select>
            </el-form-item>
          </div>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
//富文本编辑器需要引用的
import { Quill, quillEditor } from "vue-quill-editor";
const Embed = Quill.import("blots/embed");
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import draggable from "vuedraggable";
//引入font.css  路径需要注意 要写对
import "./font.css"; //复制粘贴需要的
// import Clipboard from "clipboard";

// 自定义字体大小
let Size = Quill.import("attributors/style/size");
Size.whitelist = ["10px", "12px", "14px", "16px", "18px", "20px"];
Quill.register(Size, true);

// 自定义字体类型
var fonts = [
  "SimSun",
  "SimHei",
  "Microsoft-YaHei",
  "KaiTi",
  "FangSong",
  "Arial",
  "Times-New-Roman",
  "sans-serif",
  "宋体",
  "黑体",
];
var Font = Quill.import("formats/font");
Font.whitelist = fonts;
Quill.register(Font, true);
class myBlot extends Embed {
  static blotName = "myblot";
  static tagName = "myblot";
  static create(value) {
    const node = super.create(value);
    node.innerHTML = value;
    return node;
  }
}
// 注册
Quill.register(myBlot);
export default {
  name: "FuncFormsEdit",
  components: {
    quillEditor,
    draggable,
  },
  computed: {
    /*editor() {
        return this.$refs.myQuillEditor.quill
      }*/
  },
  mounted() {
    //初始化
    document.body.ondrop = function (event) {
      event.preventDefault();
      event.stopPropagation();
    };
  },
  data() {
    return {
      messageTemplate: {
        templateName: "",
        tableTitle: "",
        senderName: "",
        timeData: "",
        channelSelect: "",
        orgSelect: "",
        content: "",
      },
      treeSelect: false,
      tags: [
        {
          name: "日期",
          id: 1,
          icon: "https://img0.baidu.com/it/u=2713042258,860821358&fm=26&fmt=auto&gp=0.jpg",
        },
        {
          name: "指标控件",
          id: 2,
          icon: "https://img0.baidu.com/it/u=2705540191,3386342097&fm=26&fmt=auto&gp=0.jpg",
        },
        {
          name: "文本段落",
          id: 3,
          icon: "https://img0.baidu.com/it/u=1460965324,1601771048&fm=26&fmt=auto&gp=0.jpg",
        },
      ],
      filterText: "",
      data: [
        {
          id: 1,
          label: "集团",
          children: [
            {
              id: 4,
              label: "业绩快报",
              children: [
                {
                  id: 9,
                  label: "集团保费",
                },
                {
                  id: 10,
                  label: "综合金融",
                },
              ],
            },
          ],
        },
        {
          id: 2,
          label: "产险",
          children: [
            {
              id: 5,
              label: "业绩快报",
            },
          ],
        },
      ],
      defaultProps: {
        children: "children",
        label: "label",
      },
      checkList: [],
      value: "",
      //表单验证
      rules: {
        /*a_title: [
            {required: true, message: '请输入标题', trigger: 'blur'}
          ],
          a_content: [
            {required: true, message: '请输入详细内容', trigger: 'blur'}
          ]*/
      },
      screenX: "",
      editorOption: {
        placeholder: "",
        theme: "snow", // or 'bubble'
        modules: {
          toolbar: {
            container: "#toolbar",
          },
        },
      },
      newText:
        "截止（时间）传统营销标保（指标）万元，达成率123%（目标2324万元），同比34234%，增加23234万元",
      newTexts: "",
      newTextss: "",
      id: "",
      show: true,
      a: "",
      index: [],
      shows: true,
    };
  },
  watch: {
    messageTemplate: function (a, b) {
      if (a.email_sender_name != "") {
        this.checkList = eval(a.email_sender_name);
      }
    },

    filterText(val) {
      this.$refs.tree.filter(val);
    },
    a: function (val) {
      console.log(val, window.getSelection().toString().includes(val));
      if (window.getSelection().toString().includes(val)) {
        console.log(val);
      }
    },
  },
  created() {
    window.toCreated = this.toCreated;
    // this.editer = this.$refs.editer.quill
    // this.initEvent()
  },
  methods: {
    handleNodeClick(data) {
      this.treeSelect = false;
      this.filterText = data.label;
    },
    treeSelects() {
      this.treeSelect = !this.treeSelect;
    },
    filterNode(value, data) {
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },
    mouseOut(evt, item) {
      this.newTextss = item.name;
      if (item.name == "指标控件") {
      }
      this.newTexts = ` <span class='textLi' onclick="toCreated('${item.name}')"><img src=${item.icon} /><p>${item.name}</p></span>`;
      // this.newTexts = "只是我们" + newTexts +"xjehrehrr
    },
    // initEvent(){
    //   this.editer.on('text-change',function(delta,oldDelta,source){
    //     console.log(editer.on('text-change',delta,oldDelta,source))
    //   })
    // },
    toCreated(item) {
      console.log(item);
      if (item == "日期") {
        this.show = false;
      } else if (item == "指标控件") {
        this.show = true;
      }

      // console.log(111);
    },
    changeSelect(evt) {
      // console.log(evt);
    },
    getdata(evt) {
      this.$refs.editer.quill.blur();
    },
    datadragEnd(evt) {
      // console.log(this.newTexts);
      this.$nextTick((_) => {
        this.$refs.editer._events.focus;
      });
      var index = this.$refs.editer.quill.selection.savedRange.index;

      if (this.screenX > 400) {
        var newText = this.newTexts;
        if (newText.includes("日期")) {
          this.show = false;
        } else {
          this.show = true;
        }
        if (this.newTexts.includes("指标控件") == true) {
          this.index.push(this.newTexts.includes("指标控件"));
          if (this.index.length == 3) {
            return false;
          }
        }

        this.$refs.editer.quill.insertEmbed(index, "myblot", newText);
      }
    },
    // 已放置到指定位置
    drog(e) {
      e.preventDefault();
      e.stopPropagation();
      this.datadragEnd();
    },
    // 移动中
    dragOver(e) {
      e.preventDefault();
      e.stopPropagation();
      // console.log(this.$refs.editer._events.focus)
      this.$refs.editer._events.focus;
      this.screenX = e.screenX;
    },
    onEditorFocus(e) {
      console.log(this.$refs.editer)
      // this.$nextTick((_) => {
      //   this.$refs.editer.quill.focus();
      // });
    },
    onSubmit() {
      if (
        this.messageTemplate.timeData == "" ||
        this.messageTemplate.senderName == ""
      ) {
        this.$message({
          message: "保存失败，请选择设置时间或者指标",
          type: "error",
        });
        return;
      } else {
        this.$message({
          message: "保存成功",
          type: "success",
        });
      }

      console.log(this.newText);
      // console.log(this.messageTemplate.channelSelect,this.messageTemplate.orgSelect,this.messageTemplate.templateName,this.messageTemplate.senderName,this.messageTemplate.content,this.messageTemplate.tableTitle,this.messageTemplate.timeData)
      // this.messageTemplate.email_sender_name = this.checkList;
      if (this.show != true) {
        this.newText = `截止<span style="color:red;">${this.messageTemplate.timeData}</span>传统营销标保万元，达成率123%（目标2324万元），同比34234%，增加23234万元`;
      } else {
        this.newText = `截止<span style="color:red;" onclick="toCreated('${this.messageTemplate.timeData}')">${this.messageTemplate.timeData}</span>传统营销标保<span style="color:red;">${this.messageTemplate.senderName}</span>万元，达成率123%（目标2324万元），同比34234%，增加23234万元${this.filterText}`;
      }
      console.log(
        "这是控件名称：" +
          this.messageTemplate.templateName +
          "，这是报表名称：" +
          this.messageTemplate.tableTitle +
          "，这是指标名称：" +
          this.messageTemplate.senderName +
          "，这是日期：" +
          this.messageTemplate.timeData +
          "，这是渠道：" +
          this.messageTemplate.channelSelect +
          "，这是险种：" +
          this.messageTemplate.orgSelect +
          ",这是个团" +
          this.messageTemplate.content
      );
    },
    noSubmit() {
      this.$message({
        message: "取消编辑",
        type: "success",
      });
      this.newText = `<strong></strong>`;
      // location.reload();
    },
  },
};
</script>

<style scoped>
/* * {
  padding: 0;
  margin: 0;
} */
body,
html {
  margin: 0;
}
.topContent {
  /* height: 3rem; */
  width: 1260px;
  background: #f0f0f0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 7px 10px;
  font-size: 14px;
  margin: 0 auto;
}
.el-row {
  width: 1260px;
  margin: 0 auto;
  padding-top: 20px;
}
>>> myblot {
  display: inline-block;
}
>>> .topButton .el-button {
  /* margin-right: 30px; */
  padding: 8px 25px;
}
.leftZJ {
  width: 100%;
  display: flex;
  margin: 0;
  padding: 0;
}
>>> .textLi {
  /* width: 30%;
  width: 80px; */
  /* height: 80px; */
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  cursor: default;
  list-style: none;
  font-size: 14px;
  color: #666;
  /* float: left; */
  margin: 0 10px;
  border-radius: 6px;
  transition: all 0.3s;
  cursor: pointer;
}
>>> .textLi img {
  width: 20px;
  height: 20px;
}
>>> .ql-editor p {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  /* height: 20px; */
  flex-wrap: wrap;
}
/* >>>.ql-editor p[data-v-47323bf2] myblot{
  height: 20px;
  display: flex;
  justify-content: space-around;
  align-items: center;
} */
>>> .textLis {
  display: flex;
  flex-direction: initial;
  align-items: center;
  line-height: 30px;
  height: 30px;
  padding: 0;
  margin: 0;
  /* display: block; */
}
.leftZJ li {
  width: 30%;
  width: 80px;
  height: 80px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  cursor: default;
  list-style: none;
  font-size: 14px;
  color: #666;
  float: left;
  margin: 0 10px;
  border-radius: 6px;
  transition: all 0.3s;
  cursor: pointer;
}
.leftZJ li img {
  width: 20px;
  height: 20px;
}
.el-form {
  display: flex;
  justify-content: space-between;
}
.el-form-item {
  display: flex;
  flex-direction: column;
  /* flex: 1; */
}
.el-form-item:first-child {
  /* width: 25%; */
  /* padding-top: 30px; */
  border-right: 1px solid #f0f0f0;
}
.el-form-item:nth-child(2) {
  /* width: 40%; */

  border-right: 1px solid #f0f0f0;
}
.inputDiv {
  position: relative;
}
>>> .el-tree {
  width: 100%;
  position: absolute;
  /* padding: 0 1px; */
  top: 42px;
  margin: 0 auto;
  background: #fff;
  z-index: 999;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
}
.inputDiv p {
  margin: 0;
  padding: 10px 0;
  font-size: 14px;
  color: #666;
}
>>> .el-form-item__content {
  margin-left: 0 !important;
  display: flex;
  flex-wrap: wrap;
}
>>> .el-form-item__content > div span {
  /* display: flex;
  flex-wrap: wrap; */
  /* justify-content: space-around */
}
>>> .el-form-item__content > div span > .tag-read {
  /* width: 40%; */
}
>>> .el-form-item__content .el-button--mini,
.el-button--small {
  width: 40%;
  margin-right: 10px;
  margin-left: 0;
}
>>> .el-form-item__label {
  text-align: center;
  margin-left: 0 !important;
}
>>> .ql-snow .ql-picker-label {
  top: -7px;
}
.ql-snow .ql-picker.ql-font {
  width: 50px;
}
>>> .ql-snow .ql-picker:not(.ql-color-picker):not(.ql-icon-picker) svg {
  right: -8px;
  margin-top: 0;
}
.ql-snow .ql-color-picker,
.ql-snow .ql-icon-picker {
  margin-left: 10px;
}
/* .ql-snow .ql-picker{
  margin-right: 10px;
} */
.ql-snow .ql-picker.ql-size {
  width: 60px;
}
.ql-snow .ql-picker.ql-header {
  width: 50px;
}
>>> .inputDiv .el-form-item {
  /* width: 90%; */
  display: flex;
  flex-direction: initial;
}
>>> .el-input__inner {
  width: 200px;
}
.el-button--success {
  background: #fff;
  color: #000;
  border: none;
}
.el-button--success img {
  width: 20px;
  height: 20px;
}
.ql-container ql-snow {
  width: 90%;
}
</style>