<template>
  <a-layout>
    <a-card>
      <a-form-model ref="ruleForm" :model="form" :rules="rules" :label-col="labelCol" :wrapper-col="wrapperCol">
        <a-form-model-item label="出库单编码" required prop="OutCode">
          <a-input v-model="form.OutCode" placeholder="请输入出库单编码" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="elect">自动获取</a-button>
          </a-input>
        </a-form-model-item>

        <a-form-model-item label="出库类型编码" required prop="OutTypeCode">
          <a-input v-model="form.OutTypeCode" placeholder="请输入出库类型编码" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="showModal">选择</a-button>
          </a-input>
        </a-form-model-item>

        <a-form-model-item ref="name" label="关联单据" prop="RelatedDocuments">
          <a-input v-model="form.RelatedDocuments" placeholder="请选择关联单据" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="showModal">选择</a-button>
          </a-input>
        </a-form-model-item>
        <a-form-model-item ref="name" label="供应商编码" prop="SupplierCode">
          <a-input v-model="form.SupplierCode" placeholder="请输入供应商编码" @blur="() => {}"></a-input>
        </a-form-model-item>
        <a-form-model-item ref="name" label="客户编码" prop="CustomerCode">
          <a-input v-model="form.CustomerCode" placeholder="请输入客户编码" @blur="() => {}"></a-input>
        </a-form-model-item>
        <a-form-model-item ref="name" label="部门编码" prop="DepartmentCode">
          <a-input v-model="form.DepartmentCode" placeholder="请输入部门编码" @blur="() => {}"></a-input>
        </a-form-model-item>
        <a-form-model-item label="出库日期" prop="OutDate">
          <a-date-picker v-model="form.OutDate" show-time type="date" placeholder="请输入出库日期" style="width: 100%;" />
        </a-form-model-item>
        <a-form-model-item label="商品清单：" prop="ProductList">
          <a-input v-model="form.ProductList" placeholder="请输入存货编码" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="showModal">选择</a-button>
          </a-input>
          <a-table
            :locale="{emptyText: '暂无数据'}"
            :columns="selectcolumns"
            :scroll="{ x: 1200 }"
            :data-source="numberRow"
            :pagination="false"
            bordered
          ></a-table>
        </a-form-model-item>
        <a-modal v-model="visible" title="选择编码" width="1200px" @ok="handleOk">
          <a-input-search placeholder="搜索" style="width: 400px;margin-bottom:20px" @search="onSearch" />
          <a-table :columns="columns" :scroll="{ x: 1500 }" :data-source="data" :pagination="false" bordered>
            <span slot="checked" style="margin: 0" slot-scope="text, record">
              <a-checkbox v-model="record.checked" @change="onChange(record)" />
            </span>
            <a slot="name" slot-scope="text">{{ text }}</a>
          </a-table>
        </a-modal>
      </a-form-model>
    </a-card>
    <a-layout-footer :style="{ position: 'fixed', width: '100%', bottom: '0px', marginLeft: '-25px',zIndex:'999'}">
      <a-card>
        <a-row>
          <a-col :span='1' :offset='4'>
             <a-button type="primary" @click="resetForm">重置表单</a-button>
          </a-col>
          <a-col :span='1' :offset='1'>
             <a-button type="primary"  @click="onSubmit">保存</a-button>
          </a-col>
          <a-col :span='1' :offset='1'>
             <a-button type  @click="Back">返回</a-button>
          </a-col>
        </a-row>
      </a-card>
    </a-layout-footer>
  </a-layout>
</template>
<script>
import Vue from 'vue'
import { formModel, Button } from 'ant-design-vue'
Vue.use(formModel, Button)
import { postOutManagementAdd } from '@/api/manage'

const columns = [
  {
    title: '选择',
    dataIndex: 'checked',
    key: 'checked',
    width: 100,
    scopedSlots: { customRender: 'checked' }
  },
  {
    title: '出库单编码',
    dataIndex: 'OutCode',
    key: 'OutCode',
    scopedSlots: { customRender: 'OutCode' },
    width: 100
  },
  {
    title: '存货编码',
    dataIndex: 'InventoryCode',
    key: 'InventoryCode',
    scopedSlots: { customRender: 'InventoryCode' },
    width: 100
  },
  {
    title: '存货名称',
    dataIndex: 'InventoryName',
    key: 'InventoryName',
    width: 100
  },
  {
    title: '货位编码',
    dataIndex: 'CargoLocationCode',
    key: 'CargoLocationCode',
    width: 100
  },
  {
    title: '批次编码',
    dataIndex: 'BatchCode',
    key: 'BatchCode',
    width: 100
  },
  {
    title: '数量',
    dataIndex: 'Quantity',
    key: 'Quantity',
    width: 100
  },
  {
    title: '计量单位',
    dataIndex: 'Unit',
    key: 'Unit',
    width: 100
  },
  {
    title: '包装数量',
    dataIndex: 'PackingQuantity',
    key: 'PackingQuantity',
    width: 100
  },
  {
    title: '包装单位',
    dataIndex: 'PackingUnit',
    key: 'PackingUnit',
    width: 100
  },
  {
    title: '单价',
    dataIndex: 'UnitPrice',
    key: 'UnitPrice',
    width: 100
  },
  {
    title: '金额',
    dataIndex: 'Amount',
    key: 'Amount',
    width: 100
  }
]
const selectcolumns = [
  {
    title: '出库单编码',
    dataIndex: 'OutCode',
    key: 'OutCode',
    scopedSlots: { customRender: 'OutCode' },
    width: 100
  },
  {
    title: '存货编码',
    dataIndex: 'InventoryCode',
    key: 'InventoryCode',
    scopedSlots: { customRender: 'InventoryCode' },
    width: 100
  },
  {
    title: '存货名称',
    dataIndex: 'InventoryName',
    key: 'InventoryName',
    width: 100
  },
  {
    title: '货位编码',
    dataIndex: 'CargoLocationCode',
    key: 'CargoLocationCode',
    width: 100
  },
  {
    title: '批次编码',
    dataIndex: 'BatchCode',
    key: 'BatchCode',
    width: 100
  },
  {
    title: '数量',
    dataIndex: 'Quantity',
    key: 'Quantity',
    width: 100
  },
  {
    title: '计量单位',
    dataIndex: 'Unit',
    key: 'Unit',
    width: 100
  },
  {
    title: '包装数量',
    dataIndex: 'PackingQuantity',
    key: 'PackingQuantity',
    width: 100
  },
  {
    title: '包装单位',
    dataIndex: 'PackingUnit',
    key: 'PackingUnit',
    width: 100
  },
  {
    title: '单价',
    dataIndex: 'UnitPrice',
    key: 'UnitPrice',
    width: 100
  },
  {
    title: '金额',
    dataIndex: 'Amount',
    key: 'Amount',
    width: 100
  }
]

const data = [
  {
    key: '1',
    Type: '食品',
    Num: 32,
    Warehouse: 'A1',
    OutProduct: '牛肉'
  },
  {
    key: '2',
    Type: '消费品',
    Num: 42,
    Warehouse: 'A1',
    OutProduct: '卫生纸'
  },
  {
    key: '3',
    Type: '奢侈品',
    Num: 32,
    Warehouse: 'A1',
    OutProduct: '香莱尔口红'
  }
]
const numberRow = []
export default {
  data() {
    return {
      numberRow,
      selectedRow: [],
      selectcolumns,
      visible: false,
      selectedRowKeys: [],
      data,
      columns,
      headers: {
        authorization: 'authorization-text'
      },
      size: 'small',
      labelCol: { span: 2 },
      wrapperCol: { span: 22 },
      other: '',
      form: {
        type: '', //类型
        InventoryCode: '', //存货编码
        InventoryName: '', //存货名称
        CargoLocationCode: '', //货位编码
        BatchCode: '', //批次编码
        Quantity: '', //数量
        Unit: '', //计量单位
        PackingQuantity: '', //包装数量
        PackingUnit: '', //包装单位
        UnitPrice: '', //单价
        Amount: '', //金额
        OutTypeCode: '', //出库类型编码
        OutDate: '', //出库日期
        ProductList: '', //商品清单：
        RelatedDocuments: '', //关联单据
        OutProduct: '', //出库产品
        Desc: '',
        OutCode: '',
        CustomerCode: '', //客户编码
        DepartmentCode: '', //部门编码
        SupplierCode: '' //供应商编码
      },
      rules: {
        OutTypeCode: [{ required: true, message: '请输入出库类型编码', trigger: 'blur' }],
        type: [
          { required: true, message: '请选择出库类型', trigger: 'blur' },
          { min: 1, max: 3, message: '', trigger: 'blur' }
        ],
        RelatedDocuments: [{ required: true, message: '请选择关联单据', trigger: 'blur' }],
        SupplierCode: [{ required: true, message: '请输入供应商编码', trigger: 'blur' }],
        CustomerCode: [{ required: true, message: '请输入客户编码', trigger: 'blur' }],
        DepartmentCode: [{ required: true, message: '请输入部门编码', trigger: 'blur' }],
        OutCode: [{ required: true, message: '请输入出库单编码', trigger: 'blur' }],

        OutDate: [{ required: true, message: '请选择出库日期', trigger: 'change' }],
        RelatedDocuments: [{ required: true, message: '请选择需要关联的单据', trigger: 'blur' }],
        OutProduct: [{ required: true, message: '请输入出库产品', trigger: 'blur' }]
      }
    }
  },
  computed: {
    rowSelection() {
      const { selectedRowKeys } = this
      return {
        selectedRowKeys,
        onChange: this.onSelectChange,
        hideDefaultSelections: true,
        onSelection: this.onSelection
      }
    }
  },
  methods: {
    onSearch(value) {
      console.log(value)
    },
    handleChange(info) {
      if (info.file.status !== 'uploading') {
        console.log(info.file, info.fileList)
      }
      if (info.file.status === 'done') {
        this.$message.success(`${info.file.name} file uploaded successfully`)
      } else if (info.file.status === 'error') {
        this.$message.error(`${info.file.name} file upload failed.`)
      }
    },
    handleSearchChange(value) {
      console.log(`selected ${value}`)
    },
    onSubmit() {
      this.$refs.ruleForm.validate(valid => {
        console.log('name--->', this.form)
        if (valid) {
          postOutManagementAdd(this.form).then(res => {
            console.log('res------->', res)
          })
          alert('保存成功，点击确认回到档案界面!')
          this.$router.push({ name: 'OutManagementList' })
        } else {
          console.log('error submit!!')
          alert('亲，您的填写内容不符合要求，请重新填写！！！')
          return false
        }
      })
    },
    Back() {
      this.$router.push({ name: 'OutManagementList' })
    },
    resetForm() {
      this.$refs.ruleForm.resetFields()
    },
    handleBlur() {
      console.log('blur')
    },
    handleFocus() {
      console.log('focus')
    },
    filterOption(input, option) {
      return option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
    },
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    elect() {
      this.form.OutCode = 'PT2020062200001'
    },
    showModal() {
      this.visible = true
    },

    handleOk(e) {
      console.log(e)
      this.visible = false
      this.numberRow = this.selectedRow
      console.log(this.numberRow)
    },
    onChange(record) {
      console.log('check', record)
      if (record.checked) {
        this.selectedRow.push(record)
        console.log(this.selectedRow)
      }
    }
  }
}
</script>
