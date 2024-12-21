<template>
  <div class="app-container">
    <!-- 查询条件居中显示 -->
    <el-row justify="center" style="padding-top: 5vh">
      <el-col :span="14">
        <el-form ref="form" :model="form" label-width="120px" class="query-form">
          <el-form-item label="电影类型">
            <el-autocomplete v-model="form.genre" :fetch-suggestions="genreSearchSuggest" placeholder="请输入电影类型"
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
  name: "genreSearch",
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

    genreSearchSuggest(queryString, cb) {
      const fixedGenres = [
        "Comedy",
        "Action",
        "Special Interest",
        "Adventure",
        "Drama",
        "Romance",
        "Music Videos & Concerts",
        "Kids & Family",
        "Mystery & Thrillers",
        "Fantasy",
        "Anime & Manga",
        "Exercise & Fitness",
        "Documentary",
        "Action & Adventure"
      ];
      const result = fixedGenres.map((genre) => ({ value: genre }));

      // 调用回调函数返回结果
      cb(result);
    },

    search(form) {
      if (form.columns.length == 0) {
        this.$message.warning("请至少输入一个条件!");
      } else {
        this.isLoading = true;

        // // Process form values
        // this.prepareFormValues(form);

        // // Query MySQL for count
        // this.queryMySQLCount(form);

        // // Query MySQL for results
        // this.queryMySQLResults(form);

        // // Query Hive (Spark)
        // this.queryHiveResults(form);
        this.queryNeo4jResults(form);
      }
    },

    queryNeo4jResults(form){
      this.$axios
          .get("/neo4j/actors/genre", {
            params:{
              genre: form.genre,
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
    },

    // queryMySQLResults(form) {
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
    //         this.isLoading = false;
    //         this.result = res.data;
    //         this.mysql_speed = res.consuming_time;
    //       })
    //       .catch((err) => {
    //         this.$message.error("当前mysql网络异常，请稍后再试");
    //       });
    // },

    // queryHiveResults(form) {
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
    //         this.spark_speed = res.consuming_time;
    //       })
    //       .catch((err) => {
    //         this.$message.error("当前spark网络异常，请稍后再试");
    //       });
    // },

    getNewPage(form) {
      this.isLoading = true;
      this.$axios
          .get("/neo4j/actors/genre", {
            params:{
              genre: form.genre,
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
  }
};
</script>

<style scoped>
.app-container {
  padding: 20px;
}
</style>
