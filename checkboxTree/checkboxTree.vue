<template>
  <div>
    <template v-for="item in roleList">
      <template v-for="treeData in item.menuTreeList">
        <div :key="treeData.id">
          <p class="check-group">
            <el-checkbox v-model="treeData.mychecked" :indeterminate="treeData.isIndeterminate" @change="handleCheckAllChange({ val: treeData, checked: $event })">
              {{ treeData.name }}
            </el-checkbox>
          </p>
          <checkboxTreeRender :tree-data="treeData" @handle-check-all-change="handleCheckAllChange"></checkboxTreeRender>
        </div>
      </template>
    </template>
  </div>
</template>
 
<script>
  import checkboxTreeRender from './checkboxTreeRender'
  const returnCheckTree = (data, checkArr = []) => {
    data.forEach((v) => {
      if (v.mychecked || v.isIndeterminate) {
        !checkArr.includes(v.id) && checkArr.push(v.id)
      }

      if (v.children && v.children.length) {
        returnCheckTree(v.children, checkArr)
      }
    })

    return checkArr
  }

  const returnEditRoleTreeIds = (data, checkArr = []) => {
    data.forEach((v) => {
      !checkArr.includes(v.id) && checkArr.push(v.id)

      if (v.children && v.children.length) {
        returnEditRoleTreeIds(v.children, checkArr)
      }
    })

    return checkArr
  }

  const fmtTreeData = (data) => {
    data.forEach((v) => {
      v.mychecked = false
      v.isIndeterminate = false

      if (v.children && v.children.length > 0) {
        fmtTreeData(v.children)
      }
    })
    return data
  }

  export default {
    name: 'checkTree',
    components: {
      checkboxTreeRender,
    },
    props: {
      roleList: {
        type: Array,
        default: () => [],
      },
    },
    data() {
      return {}
    },
    watch: {},
    created() {},
    mounted() {},
    methods: {
      handleCheckAllChange(data) {
        let { val, checked } = data
        if (val.children.length > 0) {
          // 处理下级
          this.findChildren(val.children, checked)
        } else {
          // 处理本级
          val.children.forEach((v) => {
            v.mychecked = checked
          })
        }
        if (val.parentId !== -1) {
          // 处理上级
          this.findParent(this.roleList, val.parentId)
        }
        val.isIndeterminate = false
      },
      // 设置子级
      findChildren(list, checked) {
        list.forEach((child) => {
          child.mychecked = checked
          child.isIndeterminate = false
          if (child.children.length > 0) {
            this.findChildren(child.children, checked)
          }
        })
      },
      // 设置这一整条线
      findParent(list, parentId) {
        list.forEach((k) => {
          if (k.menuTreeList) {
            k.menuTreeList.forEach((child) => {
              this.handleList(child, parentId)
            })
          } else {
            this.handleList(k, parentId)
          }
        })
      },
      // 设置这一整条线具体方法
      handleList(child, parentId) {
        let parentCheckedLength = 0
        let parentIndeterminateLength = 0
        if (child.id === parentId) {
          child.children.forEach((children) => {
            if (children.isIndeterminate) {
              parentIndeterminateLength++
            } else if (children.mychecked) {
              parentCheckedLength++
            }
          })
          child.mychecked = parentCheckedLength === child.children.length
          child.isIndeterminate = (parentIndeterminateLength > 0 || parentCheckedLength > 0) && parentCheckedLength < child.children.length
          if (child.parentId !== -1) {
            this.findParent(this.roleList, child.parentId)
          }
        } else if (child.children.length > 0) {
          this.findParent(child.children, parentId)
        }
      },
      // 返回所有已选或权限的role
      returnAllCheckIds(currentData) {
        let roleIds = []
        currentData.forEach((k) => {
          roleIds = [...returnCheckTree(k.menuTreeList), ...roleIds]
        })

        return roleIds.join(',')
      },
      // 编辑时回显权限数据
      refurbishTreeCheckStatus(checkData, allData) {
        let roleIds = []
        checkData.forEach((k) => {
          roleIds = [...returnEditRoleTreeIds(k.menuTreeList), ...roleIds]
        })
        allData.forEach((k) => {
          this.setTreeCheckStatus(k.menuTreeList, roleIds)
        })

        allData.forEach((k) => {
          this.setTreeIndeterminateStatus(k.menuTreeList)
        })
      },
      // 初始化树状数据
      formatTreeData(currentData) {
        currentData.forEach((k) => {
          fmtTreeData(k.menuTreeList)
        })

        return currentData
      },
      // 所有已选择的role全部设置为已选
      setTreeCheckStatus(data, roleIds = []) {
        data.forEach((v) => {
          if (roleIds.includes(v.id)) {
            v.mychecked = true
          }

          if (v.children && v.children.length) {
            this.setTreeCheckStatus(v.children, roleIds)
          }
        })
      },
      // 重新递归设置半选状态
      setTreeIndeterminateStatus(data) {
        data.forEach((v) => {
          let parentCheckedLength = 0
          let parentIndeterminateLength = 0
          v.children.forEach((children) => {
            if (children.isIndeterminate) {
              parentIndeterminateLength++
            } else if (children.mychecked) {
              parentCheckedLength++
            }
          })
          v.isIndeterminate = (parentIndeterminateLength > 0 || parentCheckedLength > 0) && parentCheckedLength < v.children.length

          if (v.children && v.children.length) {
            this.setTreeIndeterminateStatus(v.children)
          }
        })
      },
    },
  }
</script>
 
<style scoped>
  .check-group {
    margin: 0 -20px 15px -20px;
    padding: 10px 20px;
    background: rgb(242, 242, 242);
  }
</style>
