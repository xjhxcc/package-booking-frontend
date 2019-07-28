<template>
  <div>
    <div class="table-operations">
      <a-button @click="findAll">ALL</a-button>
      <a-button @click="findByPreed">已预约</a-button>
      <a-button @click="findByFetched">已取件</a-button>
      <a-button @click="findByNotFetched">未取件</a-button>
      <a-button type="primary" @click="showModal">+添加</a-button>
      <a-modal
        title="包裹入库"
        :visible="visible"
        @ok="handleOk"
        :confirmLoading="confirmLoading"
        @cancel="handleCancel"
      >
        <a-form :form="form" @submit="handleSubmit">
          <a-form-item label="运单号" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'packageId',
          {rules: [{ required: true, message: 'Please input your 运单号!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item label="收件人" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'username',
          {rules: [{ required: true, message: 'Please input your 收件人!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item label="电话" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'tel',
          {rules: [{ required: true, message: 'Please input your tel!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item label="重量" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'weight',
          {rules: [{ required: true, message: 'Please input your weight!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item :wrapper-col="{ span: 12, offset: 5 }">
            <a-button type="primary" html-type="submit">Submit</a-button>
          </a-form-item>
        </a-form>
      </a-modal>
    </div>
    <a-table :columns="columns" :dataSource="data">
      <a slot="name" slot-scope="text" href="javascript:;">{{text}}</a>
      <span slot="customTitle">
        <a-icon type="smile-o" />Id
      </span>
      <span slot="tags" slot-scope="tags">
        <a-tag v-for="tag in tags" color="blue" :key="tag">{{tag}}</a-tag>
      </span>
      <span slot="action" slot-scope="text, record">
        <a-button @click="updateStatus">确认收货</a-button>
        
      </span>
    </a-table>
  </div>
</template>
<script>
const columns = [
  {
    title: "运单号",
    dataIndex: "packageId",
    key: "packageId"
  },
  {
    title: "收件人",
    dataIndex: "username",
    key: "username"
  },
  {
    title: "电话",
    dataIndex: "tel",
    key: "tel"
  },
  {
    title: "状态",
    key: "packageStatus",
    dataIndex: "packageStatus"
  },
  {
    title: "预约时间",
    key: "preTime",
    dataIndex: "preTime"
  },
  {
    title: "Action",
    key: "action",
    scopedSlots: { customRender: "action" }
  }
];

const data = [];

export default {
  data() {
    return {
      data,
      columns,
      ModalText: "Content of the modal",
      visible: false,
      confirmLoading: false,
      formLayout: "horizontal",
      form: this.$form.createForm(this)
      // templist:[]
    };
  },
  created() {
    this.findAll();
  },
  methods: {
    showModal() {
      this.visible = true;
    },
    handleOk(e) {
      this.ModalText = "The modal will be closed after two seconds";
      this.confirmLoading = true;
      setTimeout(() => {
        this.visible = false;
        this.confirmLoading = false;
      }, 2000);
    },
    handleCancel(e) {
      console.log("Clicked cancel button");
      this.visible = false;
    },
    handleSubmit(e) {
      e.preventDefault();
      const self = this;
      this.form.validateFields((err, values) => {
        if (!err) {
          this.form.validateFields((err, fieldsValue) => {
            console.log("Received values of form: ", fieldsValue);
            let packageId = fieldsValue["packageId"];
            let username = fieldsValue["username"];
            // let weight = fieldsValue["weight"];
            let tel = fieldsValue["tel"];
            this.axios
              .post(
                "http://localhost:8088/packageOrders",
                {
                  packageId: packageId,
                  username: username,
                  tel: tel,
                  packageStatus: 3,
                  preTime: ""
                },
                "application/json"
              )
              .then(response => {
                if (response.status == 200) {
                  alert("添加成功");
                }else{
                  alert("添加失败");
                }
              });
          });
        }
      });
    },
    handleSelectChange(value) {
      console.log(value);
      this.form.setFieldsValue({
        note: `Hi, ${value === "male" ? "man" : "lady"}!`
      });
    },
    allDataFromDataBase(templist) {
      const self = this;
      self.data = [];
      templist.forEach(item => {
        self.data.push({
          key: item["id"],
          packageId: item["packageId"],
          username: item["username"],
          tel: item["tel"],
          packageStatus: this.formatStatus(item["packageStatus"]),
          preTime: item["preTime"]
        });
      });
    },
    formatStatus(status) {
      if (status == 1) {
        return "已预约";
      } else if (status == 2) {
        return "已取件";
      } else if (status == 3) {
        return "未取件";
      } else {
        return "状态错误";
      }
    },
    findAll() {
      var templist = [];
      this.axios.get("http://localhost:8088/packageOrders").then(response => {
        templist = response.data;
        this.allDataFromDataBase(templist);
      });
    },
    findByPreed() {
      var templist = [];
      this.axios.get("http://localhost:8088/packageOrders/1").then(response => {
        templist = response.data;
        this.allDataFromDataBase(templist);
      });
    },
    findByFetched() {
      var templist = [];
      this.axios.get("http://localhost:8088/packageOrders/2").then(response => {
        templist = response.data;
        this.allDataFromDataBase(templist);
      });
    },
    findByNotFetched() {
      var templist = [];
      this.axios.get("http://localhost:8088/packageOrders/3").then(response => {
        templist = response.data;
        this.allDataFromDataBase(templist);
      });
    }
  },
  updateStatus(){
    let packageId = fieldsValue["packageId"];
    this.axios.patch("http://localhost:8088/packageOrders/packageStatus"+packageId).then(response => {
        console.log( response.data);
    });
  }
};
</script>