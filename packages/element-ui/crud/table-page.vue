<template>
  <el-card :shadow="crud.isCard"
           v-if="pageFlag && vaildData(crud.tableOption.page,true)"
           :class="b('pagination')">
    <slot name="page"></slot>
    <el-pagination :small="crud.size=='mini'"
                   :disabled="defaultPage.disabled"
                   :hide-on-single-page="defaultPage.single"
                   :pager-count="defaultPage.pagerCount"
                   :current-page.sync="defaultPage.currentPage"
                   :background="defaultPage.background"
                   :page-size="defaultPage.pageSize"
                   :page-sizes="defaultPage.pageSizes"
                   @size-change="sizeChange"
                   @prev-click="prevClick"
                   @next-click="nextClick"
                   @current-change="currentChange"
                   :layout="defaultPage.layout"
                   :total="defaultPage.total"></el-pagination>
  </el-card>
</template>

<script>
import config from "./config";
import create from "core/create";
export default create({
  name: "crud",
  inject: ["crud"],
  data () {
    return {
      config: config,
      defaultPage: {
        single: false,
        total: 0, // 总页数
        pagerCount: 7,//超过多少条隐藏
        currentPage: 1, // 当前页数
        pageSize: 10, // 每页显示多少条
        pageSizes: [10, 20, 30, 40, 50, 100],
        layout: 'total, sizes, prev, pager, next, jumper',
        background: true // 背景颜色
      }
    };
  },
  created () {
    if (this.crud.isMobile) {
      this.defaultPage.layout = 'total, sizes, prev, pager, next'
    }
    this.pageInit();
    this.crud.$emit("on-load", this.defaultPage);
  },
  watch: {
    'crud.page': {
      handler () {
        this.pageInit();
      },
      deep: true,
    },
    pageFlag () {
      this.crud.getTableHeight();
    },
    //如果当前页面删除没数据了调用第一页
    'defaultPage.total' (val) {
      if (this.defaultPage.total === (this.defaultPage.currentPage - 1) * this.defaultPage.pageSize && this.defaultPage.total != 0) {
        this.defaultPage.currentPage = this.defaultPage.currentPage - 1;
        this.currentChange(this.defaultPage.currentPage)
      }
    }
  },
  computed: {
    pageFlag () {
      return this.defaultPage.total != 0
    }
  },
  methods: {
    pageInit () {
      this.defaultPage = Object.assign(this.defaultPage, this.crud.page)
      this.updateValue();
    },
    updateValue () {
      this.crud.$emit('update:page', this.defaultPage)
    },
    //下一页事件
    nextClick (val) {
      this.crud.$emit("next-click", val)
    },
    //上一页事件
    prevClick (val) {
      this.crud.$emit("prev-click", val)
    },
    // 页大小回调
    sizeChange (val) {
      this.defaultPage.currentPage = 1;
      this.defaultPage.pageSize = val;
      this.updateValue();
      this.crud.$emit("on-load", this.defaultPage);
      this.crud.$emit("size-change", val);
    },
    // 页码回调
    currentChange (val) {
      this.updateValue();
      this.crud.$emit("on-load", this.defaultPage);
      this.crud.$emit("current-change", val);
    }
  }
});
</script>