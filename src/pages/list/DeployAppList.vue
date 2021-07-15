<template>
  <a-card>
    <div :class="advanced ? 'search' : null">
      <a-form layout="horizontal">
        <div :class="advanced ? null: 'fold'">
          <a-row>
            <a-col :md="8" :sm="24">
              <a-form-item
                  label="应用名称"
                  :labelCol="{span: 5}"
                  :wrapperCol="{span: 18, offset: 1}"
              >
                <a-input placeholder="请输入"/>
              </a-form-item>
            </a-col>
          </a-row>
        </div>
        <span style="float: right; margin-top: 3px;">
          <a-button type="primary">查询</a-button>
          <a-button style="margin-left: 8px">重置</a-button>
        </span>
      </a-form>
    </div>
    <div>
      <a-space class="operator">
        <a-button @click="addNew" type="primary">新建</a-button>
        <a-modal
            :visible="visible"
            :title="modelTitle"
            okText='确认'
            cancel-text="取消"
            @cancel="handleCancel"
            @ok="handleOk"
        >
          <!--表单 并将表单的值绑定到this.from-->
          <a-form layout='vertical' :form="form">
            <!--每一项元素-->
            <a-form-item label='应用名称'>
              <a-select placeholder="请选择" v-decorator="['appName',{ rules: [{ required: true, message: 'Please select your appName!' }] },]">
                <a-select-option value="1">关闭</a-select-option>
                <a-select-option value="2">运行中</a-select-option>
              </a-select>
            </a-form-item>
            <a-form-item label="部署路径">
              <a-input placeholder="请输入" v-decorator="['deployPath',{ rules: [{ required: true, message: 'Please input your deployPath!' }] },]"/>
            </a-form-item>
          </a-form>
        </a-modal>
      </a-space>
      <standard-table
          :columns="columns"
          :dataSource="dataSource"
          :selectedRows.sync="selectedRows"
          @clear="onClear"
          @change="onChange"
          @selectedRowChange="onSelectChange"
      >
        <div slot="action">
          <a style="margin-right: 8px">
            <a-icon type="save"/>
            保存
          </a>
          <a style="margin-right: 8px">
            <a-icon type="edit"/>
            编辑
          </a>
        </div>
        <template slot="statusTitle">
          <a-icon @click.native="onStatusTitleClick" type="info-circle"/>
        </template>
      </standard-table>
    </div>
  </a-card>
</template>

<script>
import StandardTable from '@/components/table/StandardTable'
import {METHOD, request} from "@/utils/request";

const columns = [
  {
    title: '应用名称',
    dataIndex: 'appName'
  },
  {
    title: '部署节点',
    dataIndex: 'appServer'
  },
  {
    title: '部署路径',
    dataIndex: 'deployPath'
  },
  {
    title: '更新时间',
    dataIndex: 'modifiedTime',
    sorter: true
  },
  {
    title: '操作',
    scopedSlots: {customRender: 'action'}
  }
]

const dataSource = []


export default {
  beforeCreate() {
    //创建表单
    this.form = this.$form.createForm(this, {name: 'form_in_modal'});
  },
  name: 'DeployAppList',
  components: {StandardTable},
  data() {
    return {
      advanced: true,
      columns: columns,
      dataSource: dataSource,
      selectedRows: [],
      visible: false,
      //模态对话框标题
      modelTitle: '新增应用',
    }
  },
  authorize: {
    deleteRecord: 'delete'
  },
  methods: {
    deleteRecord(key) {
      this.dataSource = this.dataSource.filter(item => item.key !== key)
      this.selectedRows = this.selectedRows.filter(item => item.key !== key)
    },
    toggleAdvanced() {
      this.advanced = !this.advanced
    },
    remove() {
      this.dataSource = this.dataSource.filter(item => this.selectedRows.findIndex(row => row.key === item.key) === -1)
      this.selectedRows = []
    },
    onClear() {
      this.$message.info('您清空了勾选的所有行')
    },
    onStatusTitleClick() {
      this.$message.info('你点击了状态栏表头')
    },
    onChange() {
      this.$message.info('表格状态改变了')
    },
    onSelectChange() {
      this.$message.info('选中行改变了')
    },
    addNew() {
      this.visible = true;
    },
    handleOk() {
      const form = this.form;
      form.validateFields((err) => {
        if (!err) {
          const url = 'http://localhost:8888/appName/' + form.getFieldValue('appName') +'/deployPath/' + form.getFieldValue('deployPath');
          console.log(url);
          request(url,METHOD.PUT).then(this.afterSubmitForm);
        }
        form.resetFields();
        this.visible = false;
      });
    },
    afterSubmitForm(res){
      console.log(res);
    },
    handleCancel() {
      this.visible = false;
    },
    handleMenuClick(e) {
      if (e.key === 'delete') {
        this.remove()
      }
    }
  }
}
</script>

<style lang="less" scoped>
.search {
  margin-bottom: 54px;
}

.fold {
  width: calc(100% - 216px);
  display: inline-block
}

.operator {
  margin-bottom: 18px;
}

@media screen and (max-width: 900px) {
  .fold {
    width: 100%;
  }
}
</style>
