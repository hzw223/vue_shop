<template>
  <div>
    <!-- 面包屑区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区 -->
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-input
              placeholder="请输入内容"
              v-model="queryInfo.query"
              clearable
              @clear="getOrderList"
              ><el-button
                slot="append"
                icon="el-icon-search"
                @click="getOrderList"
              ></el-button
            ></el-input>
          </el-col>
        </el-row>
      </div>
      <!-- 用户列表区 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="order_number" label="订单编号">
        </el-table-column>
        <el-table-column prop="order_price" label="订单价格(元)">
        </el-table-column>
        <el-table-column prop="pay_status" label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'"
              >已付款</el-tag
            >
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"> </el-table-column>
        <el-table-column prop="create_time" label="下单时间">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200px">
          <el-button
              @click="showBox"
              type="primary"
              icon="el-icon-edit"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              @click="showProgressBox"
            ></el-button>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 30]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 修改地址对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressVisible"
      width="30%"
      @close="addressDialogClosed"
    >
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            v-model="addressForm.address1"
            :options="cityData"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 查看物流信息 -->
    <el-dialog title="物流信息" :visible.sync="progressVisible" width="30%">
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time"
        >
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
      <span slot="footer" class="dialog-footer">
        <el-button @click="progressVisible = false">取 消</el-button>
        <el-button type="primary" @click="progressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from "./citydata";
export default {
  data() {
    return {
      // 获取订单列表的参数对象
      queryInfo: {
        query: "",
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 5,
      },
      orderList: [],
      total: 0,
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: "",
      },
      cityData,
      addressFormRules: {
        address1: [
          { required: true, message: "请选择省市区县", trigger: "blur" },
        ],
        address2: [
          { required: true, message: "请填写详细地址", trigger: "blur" },
        ],
      },
      progressVisible: false,
      progressInfo: [
        {
          time: "2017-02-14 15:06:21" /*时间 */,

          ftime: "2017-02-14 15:06:21" /*时间*/,

          context: "快件已签收,签收人是朋友" /*内容 */,
        },

        {
          time: "2017-02-14 15:06:21",

          ftime: "2017-02-14 15:06:21",

          context: "快件已签收,签收人是朋友",
        },

        {
          time: "2017-02-14 09:31:42",

          ftime: "2017-02-14 09:31:42",

          context: "武昌光谷二部(15337180407)的王战胜15717179427正在派件",
        },

        {
          time: "2017-02-14 08:15:43",

          ftime: "2017-02-14 08:15:43",

          context:
            "快件到达武昌光谷二部(15337180407)，上一站是武汉(027-84639979)扫描员是02730",
        },

        {
          time: "2017-02-13 23:58:23",

          ftime: "2017-02-13 23:58:23",

          context: "快件由武汉(027-84639979)发往武昌光谷二部(15337180407)",
        },

        {
          time: "2017-02-13 11:17:54",

          ftime: "2017-02-13 11:17:54",

          context: "快件由武汉分拨中心发往武汉(027-84639979)",
        },

        {
          time: "2017-02-13 01:48:02",

          ftime: "2017-02-13 01:48:02",

          context: "快件由郑州分拨中心发往武汉分拨中心",
        },

        {
          time: "2017-02-13 01:46:45",

          ftime: "2017-02-13 01:46:45",

          context: "快件到达郑州分拨中心，上一站是无扫描员是刘会丹",
        },

        {
          time: "2017-02-12 20:52:22",

          ftime: "2017-02-12 20:52:22",

          context: "快件由洛阳(037963602588、2566,2599、2511)发往郑州分拨中心",
        },

        {
          time: "2017-02-12 19:31:40",

          ftime: "2017-02-12 19:31:40",

          context: "快件由洛阳(037963602588、2566,2599、2511)发往郑州分拨中心",
        },

        {
          time: "2017-02-12 19:31:39",

          ftime: "2017-02-12 19:31:39",

          context: "洛阳(037963602588、2566,2599、2511)已进行装袋扫描",
        },

        {
          time: "2017-02-12 17:51:40",

          ftime: "2017-02-12 17:51:40",

          context:
            "洛阳(037963602588、2566,2599、2511)的骆康鞋业已收件，扫描员是司机5",
        },
      ],
    };
  },
  created() {
    this.getOrderList();
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品列表失败");
      }
      console.log(res.data);
      this.orderList = res.data.goods;
      this.total = res.data.total;
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getOrderList();
    },
    // 监听当前页数
    handleCurrentChange(newPage) {
      // console.log(newPage);
      this.queryInfo.pagenum = newPage;
      this.getOrderList();
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields();
    },
    showBox() {
      this.addressVisible = true;
    },
    // async showProgressBox() {
    //   const { data: res } = await this.$http.get("kuaidi/804909574412544580");

    //   if (res.meta.status !== 200) {
    //     return this.$message.error("获取物流进度失败！");
    //   }

    //   this.progressInfo = res.data;

    //   this.progressVisible = true;
    //   console.log(this.progressInfo);
    // },
    showProgressBox() {
      this.progressVisible = true;
    },
  },
};
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>