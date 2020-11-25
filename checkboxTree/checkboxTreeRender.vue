<template>
  <div>
    <div v-if="treeData.children && treeData.children.length" style="padding-left: 24px">
      <div v-for="childrenData in treeData.children" :key="childrenData.id" :style="returnStyle(childrenData.children)">
        <el-checkbox
          v-model="childrenData.mychecked"
          style="margin-bottom: 15px"
          :indeterminate="childrenData.isIndeterminate"
          :label="childrenData.id"
          @change="handleCheckAllChange({ val: childrenData, checked: $event })"
        >
          {{ childrenData.name }}
        </el-checkbox>
        <checkboxTreeRender :tree-data="childrenData" @handle-check-all-change="handleCheckAllChange"></checkboxTreeRender>
      </div>
    </div>
  </div>
</template>
 
<script>
  export default {
    name: 'CheckboxTreeRender',
    components: {},
    props: {
      treeData: {
        type: Object,
        default: function () {
          return {}
        },
      },
    },
    data() {
      return {}
    },
    watch: {},
    created() {},
    mounted() {},
    methods: {
      returnStyle(child) {
        const premise = child && child.length
        return {
          display: premise ? '' : 'inline-block',
          marginRight: premise ? '' : '30px',
        }
      },
      handleCheckAllChange(data) {
        this.$emit('handle-check-all-change', data)
      },
    },
  }
</script>
 
<style scoped></style>
