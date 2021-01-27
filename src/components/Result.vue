<template>
  <el-dialog
    :visible="visible"
    @close="$emit('update:visible', false)"
    width="600px"
    class="c-Result"
    :append-to-body="true"
  >
    <div class="dialog-title" slot="title">
      <span :style="{ fontSize: '18px' }"> 抽奖结果 </span>
      <span :style="{ fontSize: '14px', color: '#999', marginLeft: '10px' }">
        (点击号码可以删除)
      </span>
      <button @click="saveResult">导出抽奖结果</button>
    </div>

    <div
      v-for="(item, index) in resultList"
      :key="index"
      class="listrow"
      @click="
        (event) => {
          deleteRes(event, item);
        }
      "
    >
      <span class="name">
        {{ item.name }}
      </span>
      <span class="value">
        <span v-if="item.value && item.value.length === 0"> 暂未抽奖 </span>
        <span
          class="card"
          v-for="(data, j) in item.value"
          :key="j"
          :data-res="data"
        >
          {{ data }}
        </span>
      </span>
    </div>
  </el-dialog>
</template>
<script>
import { conversionCategoryName, getDomData } from '@/helper/index';
import { info } from '@/helper/info';
export default {
  name: 'c-Result',
  props: {
    visible: Boolean,
  },
  computed: {
    result: {
      get() {
        return this.$store.state.result;
      },
      set(val) {
        this.$store.commit('setResult', val);
      },
    },
    resultList() {
      const list = [];
      console.log('result', this.result);
      for (const key in this.result) {
        if (this.result.hasOwnProperty(key)) {
          const element = this.result[key];
          // console.log(element);
          let name = conversionCategoryName(key);
          list.push({
            label: key,
            name,
            // value: element,
            value: element.map(
              (num) => info.find((item) => item.key === num).name
            ), //element.map((item) => info[item - 1].name),//element.map((num) => info.find((item) => item.key === num)),
          });
        }
      }
      console.log('resultList', list);
      return list;
    },
  },
  methods: {
    saveResult() {
      // 创建隐藏的可下载链接
      const link = document.createElement('a');
      link.download = '抽奖结果.json';
      link.style.display = 'none';
      // 字符内容转变成blob地址
      const blob = new Blob([JSON.stringify(this.resultList, null, 4)], {
        type: 'application/json',
      });
      console.log(blob);
      link.href = URL.createObjectURL(blob);
      // 触发点击
      document.body.appendChild(link);
      link.click();
      // 然后移除
      document.body.removeChild(link);
    },
    deleteRes(event, row) {
      console.log('row', row);
      console.log('event', event);
      const Index = getDomData(event.target, 'res');
      console.log(Index);
      if (!Index) {
        return;
      }
      this.$confirm('此操作将移除该中奖号码，确认删除?', '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(() => {
          if (Index) {
            const result = this.result;
            result[row.label] = this.result[row.label].filter(
              (item) => item !== Number(info.find((i) => i.name == Index).key)
            );
            this.result = result;
            this.$message({
              type: 'success',
              message: '删除成功!',
            });
          }
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消',
          });
        });
    },
  },
};
</script>
<style lang="scss">
.c-Result {
  .el-dialog__body {
    max-height: 500px;
    overflow-y: auto;
  }
  .listrow {
    display: flex;
    line-height: 30px;
    .name {
      width: 80px;
      font-weight: bold;
    }
    .value {
      flex: 1;
    }
    .card {
      display: inline-block;
      // width: 40px;
      padding: 0 5px;
      line-height: 30px;
      text-align: center;
      font-size: 18px;
      font-weight: bold;
      border-radius: 4px;
      border: 1px solid #ccc;
      background-color: #f2f2f2;
      margin-left: 5px;
      margin-bottom: 5px;
      position: relative;
      cursor: pointer;
      &:hover {
        &::before {
          content: '删除';
          width: 100%;
          height: 100%;
          background-color: #ccc;
          position: absolute;
          left: 0;
          top: 0;
          color: red;
        }
      }
    }
  }
}
</style>
