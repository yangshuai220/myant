<template>
  <a-card :bordered="false">
    <a-row :gutter="8">
      <a-col :span="4">
        <div class="tree">
          <s-tree
            :dataSource="FormSettingTree"
            :openKeys.sync="openKeys"
            :search="true"
            @click="handleClick"
            @add="handleAdd"
            @titleClick="handleTitleClick"
            v-model="selectedKeys"
          ></s-tree>
        </div>
      </a-col>
      <a-col :span="20">
        <a-table
          :columns="columns"
          :data-source="formSettingList.data"
          bordered
          :scroll="{ x: 2000, y: 600 }"
          :pagination="{ hideOnSinglePage: true, pageSize: 500 }"
        >
          <a slot="fieldsort" slot-scope="text, record" @click="showSort('fieldsort', record)">{{
            record.fieldsort
          }}</a>
          <a slot="fieldsortlist" slot-scope="text, record" @click="showSort('fieldsortlist', record)">{{
            record.fieldsortlist
          }}</a>

          <span slot="fielddecription" slot-scope="text, record">
            <a-input @change="(e) => fieldname(e.target.value, record)" :value="record.fielddecription" />
          </span>

          <span slot="fieldwidthlist" slot-scope="text, record">
            <a-input
              @change="(e) => vdef2Change(e.target.value, record)"
              :value="record.fieldwidthlist"
              style="width: 50px"
            />px
          </span>
          <span slot="fielddisplay" style="margin: 0" slot-scope="text, record">
            <a-checkbox @change="(e) => fielddisplayChange(e.target.checked, record)" :checked="record.fielddisplay" />
          </span>
          <span slot="fieldedit" style="margin: 0" slot-scope="text, record">
            <a-checkbox @change="(e) => fieldeditChange(e.target.checked, record)" :checked="record.fieldedit" />
          </span>

          <span slot="fielddisplaylist" style="margin: 0" slot-scope="text, record">
            <a-checkbox
              @change="(e) => fielddisplaylistChange(e.target.checked, record)"
              :checked="record.fielddisplaylist"
            />
          </span>
          <span slot="fieldmust" style="margin: 0" slot-scope="text, record">
            <a-checkbox @change="(e) => fieldmustChange(e.target.checked, record)" :checked="record.fieldmust" />
          </span>
        </a-table>

        <a-row :style="{ marginTop: '30px' }">
          <a-col :span="1" :offset="10">
            <a-button type="primary" @click="onSubmit">保存</a-button>
          </a-col>

          <a-col :span="1" :offset="1">
            <a-button type @click="onBack">返回</a-button>
          </a-col>
        </a-row>
      </a-col>
    </a-row>
    <a-modal title="提示" :visible="sortVisible" @ok="sortOk" @cancel="sortCancel">
      <p>请输入想要移动位置：</p>
      <a-input v-model="sortAfter" />
    </a-modal>
    <org-modal ref="modal" @ok="handleSaveOk" @close="handleSaveClose" />
  </a-card>
</template>

<script>
import STree from '@/components/Tree/Tree'
import { STable } from '@/components'
import OrgModal from '../other/modules/OrgModal'
import { getFormSettingTree, getServiceList, getFormSettingList, getFormSettingColumns, updateForm } from '@/api/manage'
import { logininfo } from '@/store/mutation-types'
import Vue from 'vue'
const columns = []
//表单设置
export default {
  name: 'TreeList',
  components: {
    STable,
    STree,
    OrgModal,
  },
  data() {
    return {
      openKeys: ['01'],
      selectedKeys: ['01-01'],
      menuid: '',
      // 查询参数
      queryParam: {},
      // 表头
      columns,
      // 加载数据方法 必须为 Promise 对象

      FormSettingTree: [],
      selectedRowKeys: [],
      selectedRows: [],
      formSettingList: {},
      sortVisible: false,
      sortAfter: '',

      sortMax: 100,
      currentItem: '',
      editVisible: false,
      currentId: '',
    }
  },
  created() {
    getFormSettingColumns().then((res) => {
      console.log('columns-->', JSON.stringify(res))
      this.columns = res.result.columns
    })
    getFormSettingTree().then((res) => {
      console.log('tree-->', JSON.stringify(res.result))
      this.FormSettingTree = res.result
      this.menuid = this.FormSettingTree[0].children[0].memuid
      this.getlist()
    })
  },
  methods: {
    getlist() {
      const parameter = {
        enterpriseid: Vue.ls.get(logininfo).basepersonPO.enterpriseid,
        menuid: this.menuid,
        pageNo: 1,
        pageSize: 10,
      }

      getFormSettingList(parameter).then((res) => {
        console.log('getFormSettingList-->', JSON.stringify(res))
        if (res.status == 'FAILED') {
          this.$message.error(res.errorMsg)
        } else {
          this.formSettingList = res.result
          this.sortMax = this.formSettingList.length
        }
      })
    },
    sortOk(e) {

      if (this.currentId == 'fieldsort') {
        if (this.currentItem.fieldsort > this.sortAfter) {
          for (var i = this.sortAfter - 1; i < this.currentItem.fieldsort; i++) {
            this.formSettingList.data[i].fieldsort = parseInt(i) + 2
          }
        } else {
          for (var i = this.currentItem.fieldsort; i < this.sortAfter; i++) {
            this.formSettingList.data[i].fieldsort = i
          }
        }
        this.currentItem.fieldsort = this.sortAfter
      }else{
        //fieldsortlist
           if (this.currentItem.fieldsortlist > this.sortAfter) {
          for (var i = this.sortAfter - 1; i < this.currentItem.fieldsortlist; i++) {
            this.formSettingList.data[i].fieldsortlist = parseInt(i) + 2
          }
        } else {
          for (var i = this.currentItem.fieldsortlist; i < this.sortAfter; i++) {
            this.formSettingList.data[i].fieldsortlist = i
          }
        }
        this.currentItem.fieldsortlist = this.sortAfter
      }
      this.sortAfter = ''
      this.sortVisible = false
    },
    sortCancel(e) {
      this.sortVisible = false
    },
    showSort(text, record) {
      this.sortVisible = true
      this.currentId = text
      this.currentItem = record
    },
    onSubmit(e) {
      updateForm(this.formSettingList).then((res) => {
        this.$message.success('更改成功')
        this.getlist()
      })
    },
    onBack(e) {},

    vdef2Change(value, record) {
      for (const key in this.formSettingList.data) {
        if (record.id == this.formSettingList.data[key].id) {
          this.formSettingList.data[key].vdef2 = value
        }
      }
    },
    fieldname(value, record) {
      record.fieldname = value
    },
    fieldeditChange(value, record) {
      record.fieldedit = value
    },
    fielddisplayChange(value, record) {
      record.fielddisplay = value
    },
    fielddisplaylistChange(value, record) {
      record.fielddisplaylist = value
    },
    fieldmustChange(value, record) {
      record.fieldmust = value
    },
    onChange(e) {
      console.log(`checked = ${e.target.checked}`)
    },
    handleClick(e) {
      for (const key in this.FormSettingTree) {
        for (const i in this.FormSettingTree[key].children) {
          if (e.key == this.FormSettingTree[key].children[i].key) {
            this.menuid = this.FormSettingTree[key].children[i].memuid
          }
        }
      }
      this.getlist()
    },
    handleAdd(item) {
      console.log('add button, item', item)
      this.$message.info(`提示：你点了 ${item.key} - ${item.title} `)
      this.$refs.modal.add(item.key)
    },
    handleTitleClick(item) {
      console.log('handleTitleClick', item)
    },
    titleClick(e) {
      console.log('titleClick', e)
    },
    handleSaveOk() {},
    handleSaveClose() {},
  },
}
</script>

<style lang="less">
.tree {
  border: 1px solid #e8e8e8;
  border-radius: 4px;
  overflow: auto;
  padding: 8px 2px;
  height: 680px;
}
.custom-tree {
  /deep/ .ant-menu-item-group-title {
    position: relative;
    &:hover {
      .btn {
        display: block;
      }
    }
  }

  /deep/ .ant-menu-item {
    &:hover {
      .btn {
        display: block;
      }
    }
  }

  /deep/ .btn {
    display: none;
    position: absolute;
    top: 0;
    right: 10px;
    width: 20px;
    height: 40px;
    line-height: 40px;
    z-index: 1050;

    &:hover {
      transform: scale(1.2);
      transition: 0.5s all;
    }
  }
}
</style>
