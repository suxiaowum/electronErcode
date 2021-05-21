<template>
  <div id="a">
    <h1>二维码生成</h1>
    <!-- <input type="file" name="" value="" ref="exc" /> -->
    <el-button
      style="margin-top: 20px"
      size="small"
      type="primary"
      @click="getSave"
      >选择路径</el-button
    >
    <p style="margin-top: 20px">{{ dia }}</p>

    <el-upload
      style="margin-top: 40px"
      class="upload-demo"
      ref="upload"
      action=""
      :on-change="handlePreview"
      :auto-upload="false"
      :multiple="false"
      :limit="1"
    >
      <div slot="tip" class="el-upload__tip">
        只能上传xls/xlsx文件，且不超过500kb
      </div>
      <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
    </el-upload>
    <div style="position: fixed; bottom: 60px; left: 0; right: 0; margin: auto">
      <el-button @click="startFun" type="success">开始生成</el-button>
    </div>
    <div id="jindu" v-if="saveIng">
      <div id="jinduText">{{ this.index }}/{{ this.outputs.length }}</div>
    </div>
    <div
      id="qrcodeDive"
      ref="qrCodeDiv"
      style="width: 150px; height: 150px; position: absolute; left: -300px"
    ></div>
    <!-- <button @click="getSave">选择路径</button> -->
  </div>
</template>
<script>
import XLSX from "xlsx";
import QRCode from "qrcodejs2";
import html2canvas from "html2canvas";
import { resolve } from "url";
const { dialog } = require("electron").remote;
const fs = require("fs");
export default {
  data() {
    return {
      outputs: [],
      dia: "",
      index: 1,
      saveIng: false,
    };
  },
  mounted() {},
  methods: {
    submitUpload() {
      this.$refs.upload.submit();
    },
    handlePreview(file) {
      console.log(file);
      // console.log(file.raw.target);
      this.readExcel([file.raw]);
    },
    readExcel(e) {
      //表格导入
      var that = this;
      const files = e;
      console.log(files);
      if (files.length <= 0) {
        //如果没有文件名
        return false;
      } else if (!/\.(xls|xlsx)$/.test(files[0].name.toLowerCase())) {
        this.$Message.error("上传格式不正确，请上传xls或者xlsx格式");
        return false;
      }

      const fileReader = new FileReader();
      fileReader.onload = (ev) => {
        try {
          const data = ev.target.result;
          const workbook = XLSX.read(data, {
            type: "binary",
          });
          console.log(workbook);
          const wsname = workbook.SheetNames[0]; //取第一张表
          const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]); //生成json表格内容
          console.log(ws);
          that.outputs = []; //清空接收数据
          //编辑数据
          for (var i = 0; i < ws.length; i++) {
            var sheetData = {
              value: ws[i].value,
              name: ws[i].name,
            };
            that.outputs.push(sheetData);
          }
          // this.$refs.upload.value = "";
          console.log(that.outputs);
          // this.startFun();
        } catch (e) {
          return false;
        }
      };
      fileReader.readAsBinaryString(files[0]);
    },
    getSave() {
      this.dia = dialog.showOpenDialog({
        properties: ["openDirectory"],
      });
      console.log(this.dia);
      this.dia = this.dia[0] + "\\";
      console.log(this.dia);
    },
    async startFun() {
      if (this.dia == "") {
        this.$message.error("请选择存储路径！");
      } else if (this.outputs.length < 1) {
        this.$message.error("请上传正确的Excel表格！");
      } else {
        this.saveIng = true;
        const loading = this.$loading({
          lock: true,
          text: "正在生成",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)",
        });
        var that = this;
        console.log(Date.parse(new Date()));
        console.log(this.outputs);
        // console.log(this.$refs.qrCodeDivs);
        for (let i = 0; i < this.outputs.length; i++) {
          if (i == this.outputs.length - 1) {
            console.log(Date.parse(new Date()));
            loading.close();
            this.$message.success("生成完成！");
            this.saveIng = false;
          } else {
            await this.saveImg(i);
          }
        }
      }
    },

    async saveImg(i) {
      var that = this;
      new QRCode(this.$refs.qrCodeDiv, {
        width: 150,
        height: 150,
        text: this.outputs[i].value, // 二维码地址
        colorDark: "#000", // 二维码颜色
        colorLight: "#fff", // 二维码背景色
      });
      await html2canvas(this.$refs.qrCodeDiv)
        .then((canvas) => {
          // console.log(canvas);
          let base64 = canvas
            .toDataURL()
            .replace(/^data:image\/\w+;base64,/, "");
          let dataBuffer = new Buffer(base64, "base64");
          // console.log(that.dia + that.outputs[i].name + ".png");
          fs.writeFile(
            that.dia + that.outputs[i].name + ".png",
            dataBuffer,
            function (err) {
              document.getElementById("qrcodeDive").innerHTML = "";
              console.log(that.index);
              that.index = that.index + 1;
              if (err) {
                // console.log(err);
              } else {
                console.log("123");
              }
            }
          );
        })
        .catch((e) => {
          console.log(e);
        });
    },
  },
};
</script>
<style lang="scss" scoped>
#a {
  text-align: center;
}
#jindu {
  position: fixed;
  z-index: 999999999;
  font-size: 16px;
  color: #409eff;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;

  vertical-align: middle;
  // background: red;
}
#jinduText{
  height: 200px;
  line-height: 300px;
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
}
</style>