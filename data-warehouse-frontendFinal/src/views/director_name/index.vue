<template>
  <div class="app-container">
    <!-- 查询条件部分居中显示 -->
    <el-row justify="center" style="padding-top: 5vh">
      <el-col :span="14">
        <el-form ref="form" :model="form" label-width="120px" class="query-form">
          <el-form-item label="导演">
            <el-autocomplete v-model="form.director" :fetch-suggestions="directorSearchSuggest" placeholder="请输入导演名"
              style="width: 100%" clearable @select="handleSelect" size="small" />
          </el-form-item>
        </el-form>
        <!-- 查询按钮居中显示 -->
        <div style="text-align: center; margin-top: 20px">
          <el-button type="primary" @click="search(form)" size="small" plain>查询</el-button>
        </div>
      </el-col>
    </el-row>

    <!-- 查询结果和耗时对比并排显示 -->
    <el-row style="margin-top: 30px">
      <!-- 查询结果卡片 -->
      <el-col :span="12" style="padding-right: 10px">
        <el-card shadow="always" class="result-card">
          <h3 style="text-align: center; margin-bottom: 20px">查询结果</h3>
          <el-table :data="result" v-loading="isLoading" element-loading-text="正在为您查询..." stripe style="width: 100%" height="450">
            <el-table-column prop="movieTitle" label="电影名称" width="120" v-if="columns.movieTitle" />
            <el-table-column prop="movieId" label="编号" width="115" v-if="columns.movieId" />
            <el-table-column prop="movieRating" label="评分" width="80" v-if="columns.movieScore" />
            <el-table-column prop="movieGenre" label="类型" width="120" v-if="columns.genre" />
            <el-table-column prop="movieReviewNum" label="评论数量" width="120" v-if="columns.reviewnum" />
          </el-table>
          <el-row style="text-align: center; margin-top: 20px">
            <el-pagination layout="prev, pager, next, jumper" :current-page.sync="currentPage" :page-size="10"
                           :page-count="totalPage" @current-change="getNewPage(form)" small />
          </el-row>
        </el-card>
      </el-col>

      <!-- 耗时对比卡片 -->
      <el-col :span="12" style="padding-left: 10px">
        <el-card shadow="always" class="speed-card">
          <h3 style="text-align: center; margin-bottom: 20px">耗时对比</h3>
          <div id="speed" style="width: 100%; height: 400px"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  name: "DirectorSearch",
  data() {
    return {
      activeName: "search_res",
      isLoading: false,
      totalPage: 10,
      currentPage: 1,
      neo4j_speed:0,
      mysql_speed: 0,
      spark_speed: 0,
      form: {
        director: "",
        columns: ["directors"], // 默认查询导演
      },
      columns: {
      movieId: true,
      movieScore: true,
      genre: true,
      reviewnum: true,
      movieTitle:true
    },
      result: [],
    };
  },
  mounted() {
    this.echartsInit();
  },
  watch: {
    neo4j_speed: {
      handler(newValue, oldValue) {
        this.neo4j_speed = newValue;
        this.echartsInit();
      },
    },
    // mysql_speed: {
    //   handler(newValue, oldValue) {
    //     this.mysql_speed = newValue;
    //     this.echartsInit();
    //   },
    // },
    // spark_speed: {
    //   handler(newValue, oldValue) {
    //     this.spark_speed = newValue;
    //     this.echartsInit();
    //   },
    // },
  },

  methods: {
    handleSelect(item) {
      console.log(item);
    },

    handleClick(tab, event) {
      console.log(tab, event);
    },

    directorSearchSuggest(queryString, cb) {
      //导演搜索建议
      this.$axios
          .get("/neo4j/actors/findDir", {
          })
          .then((res) => {
            var result = [];
            for (var i = 0; i < 100; i++) {
              if(res[i]!="#")
                  result.push({value: res[i]});
            }
            cb(result);
          })
          .catch((err) => {
            this.$message.error("当前网络异常，请稍后再试");
          });
    },

    search(form) {
      if (form.director.trim() === "") {
        this.$message.warning("请至少输入一个导演名!");
      } else {
        this.isLoading = true;
        this.$axios
          .get("/neo4j/actors/director", {
            params:{
              director: form.director,
              page: 1,
              per_page: 10,
            }
          })
          .then((res) => {
            this.isLoading = false;
            console.log("这是结果", res);
            this.result = res.movieList;
            this.neo4j_speed = res.time;
          })
          .catch((err) => {
            this.$message.error("当前neo4j网络异常，请稍后再试");
          });

        // 调用数据库查询
        // this.$axios
        //     .post("/movie/detail", {
        //       director: form.director,
        //       columns: form.columns,
        //       page: this.currentPage,
        //       per_page: 10,
        //     })
        //     .then((res) => {
        //       this.result = res.data;
        //       this.totalPage = res.pages;
        //       this.isLoading = false;
        //       this.mysql_speed = res.consuming_time;
        //     })
        //     .catch((err) => {
        //       this.$message.error("当前mysql网络异常，请稍后再试");
        //     });

        // // 调用Hive查询
        // this.$axios
        //     .post("/hive/movie/detail", {
        //       director: form.director,
        //       columns: form.columns,
        //       page: this.currentPage,
        //       per_page: 10,
        //     })
        //     .then((res) => {
        //       this.spark_speed = res.consuming_time;
        //     })
        //     .catch((err) => {
        //       this.$message.error("当前spark网络异常，请稍后再试");
        //     });
      }
    },

    getNewPage(form) {
      this.isLoading = true;
      // this.$axios
      //     .post("/movie/detail", {
      //       director: form.director,
      //       columns: form.columns,
      //       page: this.currentPage,
      //       per_page: 10,
      //     })
      //     .then((res) => {
      //       this.result = res.data;
      //       this.mysql_speed = res.consuming_time;
      //       this.isLoading = false;
      //     })
      //     .catch((err) => {
      //       this.$message.error("当前mysql网络异常，请稍后再试");
      //     });

      // this.$axios
      //     .post("/hive/movie/detail", {
      //       director: form.director,
      //       columns: form.columns,
      //       page: this.currentPage,
      //       per_page: 10,
      //     })
      //     .then((res) => {
      //       this.spark_speed = res.consuming_time;
      //     })
      //     .catch((err) => {
      //       this.$message.error("当前spark网络异常，请稍后再试");
      //     });
      this.$axios
          .get("/neo4j/actors/director", {
            params:{
              director: form.director,
              page: this.currentPage,
              per_page: 10,
            }
          })
          .then((res) => {
            this.isLoading = false;
            console.log("这是结果", res);
            this.result = res.movieList;
            this.neo4j_speed = res.time;
          })
          .catch((err) => {
            this.$message.error("当前neo4j网络异常，请稍后再试");
          });
    },

    echartsInit() {
      //使用时只需要把setOption里的对象换成echarts中的options或者自己的参数即可
      console.log("开始初始化");
      this.$echarts.init(document.getElementById("speed")).setOption({
        title: {
          text: "查询耗时对比(s)",
        },
        tooltip: {},
        xAxis: {
          data: ["mysql", "hive","neo4j"],
        },
        yAxis: {},
        series: [
          {
            name: "查询耗时(s)",
            type: "bar",
            data: [this.mysql_speed, this.spark_speed,this.neo4j_speed],
          },
        ],
      });
    },
  },
};
</script>

<style scoped>
.el-divider--vertical {
  height: 100vh;
}
</style>
