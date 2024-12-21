<!-- eslint-disable no-unused-vars -->
<!-- eslint-disable vue/singleline-html-element-content-newline -->
<!-- eslint-disable vue/max-attributes-per-line -->
<!-- eslint-disable vue/require-v-for-key -->
<template>
  <div class="app-container">
    <!-- 查询条件居中显示 -->
    <el-row justify="center" style="padding-top: 5vh">
      <el-col :span="14">
        <el-form ref="form" :model="form" label-width="120px" class="query-form">
          <el-form-item label="评分">
            <el-input-number v-model="form.min_score" size="mini" :precision="2" :step="1" :max="form.max_score"
                             :min="0.0" style="width: 100px" />
            <span style="margin-left: 10px; margin-right: 10px">至</span>
            <el-input-number v-model="form.max_score" size="mini" :precision="2" :step="1" :max="5.0"
                             :min="form.min_score" style="width: 100px" />
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
            <el-table-column prop="movieScore" label="评分" width="80" v-if="columns.score" />
            <el-table-column prop="genre" label="类型" width="120" v-if="columns.genre" />
            <el-table-column prop="reviewnum" label="评论数量" width="120" v-if="columns.reviewnum" />
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
  name: "evaluateSearch",
  data() {
    return {
      activeName: "search_res",
      isLoading: false,
      totalPage: 10,
      currentPage: 1,
      neo4j_speed: 0,
      mysql_speed: 0,
      spark_speed: 0,
      form: {
        min_score: 0.0,
        max_score: 5.0,
        title: "",
        genre: "",
        columns: [""],
      },
      columns: {
      movieId: true,
      movieScore: true,
      genre: true,
      reviewnum: true,
      movieTitle:true
      },
      result: [],
      test: "",
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

    search(form) {
      if (form.columns.length == 0) {
        this.$message.warning("请至少输入一个条件!");
      } else {
        this.isLoading = true;
      //   //调用mysql查询总数
      //   this.$axios
      //       .post("/movie/count", {
      //         genre_name: form.genre,
      //         min_score: form.min_score,
      //         max_score: form.max_score,
      //         title: form.title,
      //         director: form.director,
      //         actor: form.actor,
      //         year: form.year,
      //         month: form.month,
      //         season: form.season,
      //         weekday: form.weekday,
      //         day: form.day,
      //         positive: form.positive / 100,
      //         page: 1,
      //         per_page: 10,
      //       })
      //       .then((res) => {
      //         console.log("pages:", res.pages);
      //         this.totalPage = res.pages;
      //       });

      //   //调用mysql查询
      //   this.$axios
      //       .post("/movie/detail", {
      //         genre_name: form.genre,
      //         min_score: form.min_score,
      //         max_score: form.max_score,
      //         columns: form.columns,
      //         title: form.title,
      //         director: form.director,
      //         actor: form.actor,
      //         year: form.year,
      //         month: form.month,
      //         season: form.season,
      //         weekday: form.weekday,
      //         day: form.day,
      //         positive: form.positive / 100,
      //         page: 1,
      //         per_page: 10,
      //       })
      //       .then((res) => {
      //         console.log("这是mysql的结果", res);
      //         console.log("data", res.data);
      //         this.isLoading = false;
      //         this.result = res.data;
      //         console.log("test", this.result);
      //         if (this.columns["title"] == true) {
      //           for (var i = 0; i < this.result.length; i++) {
      //             console.log("标题", this.result[i].movieTitle);
      //             this.result[i].movieTitle = this.result[i].movieTitle.replace(
      //                 /\"/g,
      //                 ""
      //             );
      //             console.log("标题后", this.result[i].movieTitle);
      //           }
      //         }
      //         this.isLoading = false;
      //         this.mysql_speed = res.consuming_time;
      //         console.log("mysql速度", this.mysql_speed);
      //       })
      //       .catch((err) => {
      //         this.$message.error("当前mysql网络异常，请稍后再试");
      //       });

      //   //调用hive查询
      //   this.$axios
      //       .post("/hive/movie/detail", {
      //         genre_name: form.genre,
      //         min_score: form.min_score,
      //         max_score: form.max_score,
      //         columns: form.columns,
      //         title: form.title,
      //         director: form.director,
      //         actor: form.actor,
      //         year: form.year,
      //         month: form.month,
      //         season: form.season,
      //         weekday: form.weekday,
      //         day: form.day,
      //         page: 1,
      //         per_page: 10,
      //       })
      //       .then((res) => {
      //         console.log("这是spark的结果", res);
      //         this.spark_speed = res.consuming_time;
      //       })
      //       .catch((err) => {
      //         this.$message.error("当前spark网络异常，请稍后再试");
      //       });
        //调用neo4j查询
        this.$axios
            .get("/neo4j/actors/score", {
              params: {
                minscore: form.min_score,
                maxscore: form.max_score,
              }
            })
            .then((res) => {
              console.log("这是结果", res);
              console.log("data", res.movieList);
              this.isLoading = false;
              // this.result = res.movieList;              ;
              console.log("test", this.result);
              this.isLoading = false;
              this.neo4j_speed = res.time;
              console.log("neo4j速度", this.neo4j_speed);
            })
            .catch((err) => {
              this.$message.error("当前neo4j网络异常，请稍后再试");
            });


      }
    },

    getNewPage(form) {
      //分页获取数据
      console.log("切换页面");
      console.log("当前页数", this.currentPage);
      this.isLoading = true;

        
        this.$axios
            .post("/movie/detail", {
              genre_name: form.genre,
              min_score: form.min_score,
              max_score: form.max_score,
              columns: form.columns,
              title: form.title,
              director: form.director,
              actor: form.actor,
              year: form.year,
              month: form.month,
              season: form.season,
              weekday: form.weekday,
              day: form.day,
              positive: form.positive / 100,
              page: this.currentPage,
              per_page: 10,
            })
            .then((res) => {
              console.log("这是mysql的结果", res);
              console.log("data", res.data);
              this.result = res.data;
              console.log(this.result);
              this.isLoading = false;
              this.mysql_speed = res.consuming_time;
            })
            .catch((err) => {
              this.$message.error("当前mysql网络异常，请稍后再试");
            });

        //调用hive查询
        this.$axios
            .post("/hive/movie/detail", {
              genre_name: form.genre,
              min_score: form.min_score,
              max_score: form.max_score,
              columns: form.columns,
              title: form.title,
              director: form.director,
              actor: form.actor,
              year: form.year,
              month: form.month,
              season: form.season,
              weekday: form.weekday,
              day: form.day,
              page: this.currentPage,
              per_page: 10,
            })
            .then((res) => {
              console.log("这是spark的结果", res);
              this.spark_speed = res.consuming_time;
            })
            .catch((err) => {
              this.$message.error("当前spark网络异常，请稍后再试");
            });
    },

    echartsInit() {
      //使用时只需要把setOption里的对象换成echarts中的options或者自己的参数即可
      console.log("开始初始化");
      this.$echarts.init(document.getElementById("speed")).setOption({
        title: {
          text: "组合查询耗时对比(s)",
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
