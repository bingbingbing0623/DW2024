<template>
  <div class="app-container">
    <!-- 查询条件部分居中显示 -->
    <el-row justify="center" style="padding-top: 5vh">
      <el-col :span="14">
        <el-form ref="form" :model="form" label-width="120px" class="query-form">
          <el-form-item label="上映时间">
            <el-row gutter={20}>
              <!-- 年份输入 -->
              <el-col :span="8">
                <el-input-number v-model="form.year" size="mini" :max="2022" :min="1930" controls-position="right"
                  style="width: 100%" placeholder="选择年份" />
              </el-col>

              <!-- 月份或季度选择 -->
              <el-col :span="8">
                <el-select v-model="form.month_season" size="mini" placeholder="选择月份/季度" style="width: 100%">
                  <el-option label="月份" value="month"></el-option>
                  <el-option label="季度" value="season"></el-option>
                  <el-option label="无" value=""></el-option>
                </el-select>
                <el-input-number v-model="form.month" v-if="form.month_season == 'month'" size="mini" :max="12" :min="1"
                  controls-position="right" style="width: 100%" placeholder="选择月份" />
                <el-input-number v-model="form.season" v-if="form.month_season == 'season'" size="mini" :min="1" :max="4"
                  controls-position="right" style="width: 100%" placeholder="选择季度" />
              </el-col>

              <!-- 天数或周几选择 -->
              <el-col :span="8">
                <el-select v-model="form.day_weekday" size="mini" placeholder="选择天数/周几" style="width: 100%">
                  <el-option label="天数" value="day"></el-option>
                  <el-option label="周几" value="weekday"></el-option>
                  <el-option label="无" value=""></el-option>
                </el-select>
                <el-input-number v-model="form.day" v-if="form.day_weekday == 'day'" size="mini" :min="1" :max="31"
                  controls-position="right" style="width: 100%" placeholder="选择天数" />
                <el-input-number v-model="form.weekday" v-if="form.day_weekday == 'weekday'" size="mini" :min="1" :max="7"
                  controls-position="right" style="width: 100%" placeholder="选择周几" />
              </el-col>
            </el-row>
          </el-form-item>
        </el-form>
        
        <!-- 查询按钮居中显示 -->
        <div style="text-align: center; margin-top: 20px">
          <el-button type="primary" @click="search(form)" size="small" plain>查询</el-button>
        </div>
      </el-col>
    </el-row>

    <!-- 查询结果和耗时对比并排显示 -->
    <el-row style="margin-top: 20px">
      <el-col :span="12">
        <div>
          <h3>查询结果</h3>
          <el-table :data="result" v-loading="isLoading" element-loading-text="正在为您查询..." stripe style="width: 100%"
            height="450">
            <el-table-column prop="movieId" label="编号" width="115" v-if="columns.movieId" />
            <el-table-column prop="movieRating" label="评分" width="80" v-if="columns.movieScore" />
            <el-table-column prop="movieGenre" label="类型" width="120" v-if="columns.genre" />
            <el-table-column prop="movieReviewNum" label="评论数量" width="120" v-if="columns.reviewnum" />
          </el-table>
          <el-row style="text-align: center; margin-top: 20px">
            <el-pagination layout="prev, pager, next, jumper" :current-page.sync="currentPage" :page-size="10"
              :page-count="totalPage" @current-change="getNewPage(form)" small />
          </el-row>
        </div>
      </el-col>

      <el-col :span="12">
        <div>
          <h3>耗时对比</h3>
          <div id="speed" style="width: 100%; height: 400px"></div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>




<script>
export default {
  name: "timeSearch",
  data() {
    return {
      activeName: "search_res",
      isLoading: false,
      totalPage: 10,
      currentPage: 1,
      mysql_speed: 0,
      spark_speed: 0,
      form: {
        year: null,
        month: null,
        season: null,
        day: null,
        weekday: null,
        columns: [],
        month_season: "",
        day_weekday: "",
      },
      columns: {
        movieId: true,
      movieScore: true,
      genre: true,
      reviewnum: true
      },
      result: [],
      test: "",
    };
  },

  mounted() {
    this.echartsInit();
  },

  watch: {
    //监听速度变化，重新渲染页面
    mysql_speed: {
      handler(newValue, oldValue) {
        this.mysql_speed = newValue;
        this.echartsInit();
      },
    },
    spark_speed: {
      handler(newValue, oldValue) {
        this.spark_speed = newValue;
        this.echartsInit();
      },
    },
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

        //判断年份是否为空
        if (form.year == 1930) {
          console.log("year为空");
          form.year = null;
          console.log(form.year);
        } else {
          console.log(form.year);
        }

        //判断月份或季度
        if (form.month_season == "") {
          console.log("月份季度为空");
          form.month = null;
          form.season = null;
        } else if (form.month_season == "month") {
          console.log("选择月份");
          form.season = null;
        } else if ((form.month_season = "season")) {
          console.log("选择季度");
          form.month = null;
        }

        //判断天数或星期几
        if (form.day_weekday == "") {
          console.log("天数为空");
          form.day = null;
          form.weekday = null;
        } else if (form.day_weekday == "day") {
          console.log("选择天数");
          form.weekday = null;
        } else if ((form.day_weekday = "weekday")) {
          console.log("选择周几");
          form.day = null;
        }

        for (var i in this.columns) {
          //清空原有条件
          this.columns[i] = false;
        }

        console.log("这是原始条件", form.columns);
        for (var i = 0; i < form.columns.length; i++) {
          this.columns[form.columns[i]] = true;
          console.log(
            "这是条件",
            form.columns[i],
            this.columns[form.columns[i]]
          );
        }

        //调用mysql查询总数
        this.$axios
          .post("/movie/count", {
            genre_name: form.genre,
            min_score: form.min_score,
            max_score: form.max_score,
            title: form.title,
            director: form.director,
            actor: form.actor,
            year: form.year,
            month: form.month,
            season: form.season,
            weekday: form.weekday,
            day: form.day,
            positive: form.positive / 100,
            page: 1,
            per_page: 10,
          })
          .then((res) => {
            console.log("pages:", res.pages);
            this.totalPage = res.pages;
          });

        //调用mysql查询
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
            page: 1,
            per_page: 10,
          })
          .then((res) => {
            console.log("这是mysql的结果", res);
            console.log("data", res.data);
            this.isLoading = false;
            this.result = res.data;
            console.log("test", this.result);
            if (this.columns["title"] == true) {
              for (var i = 0; i < this.result.length; i++) {
                console.log("标题", this.result[i].movieTitle);
                this.result[i].movieTitle = this.result[i].movieTitle.replace(
                  /\"/g,
                  ""
                );
                console.log("标题后", this.result[i].movieTitle);
              }
            }
            this.isLoading = false;
            this.mysql_speed = res.consuming_time;
            console.log("mysql速度", this.mysql_speed);
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
            page: 1,
            per_page: 10,
          })
          .then((res) => {
            console.log("这是spark的结果", res);
            this.spark_speed = res.consuming_time;
          })
          .catch((err) => {
            this.$message.error("当前spark网络异常，请稍后再试");
          });
      }
    },

    getNewPage(form) {
      //分页获取数据
      console.log("切换页面");
      console.log("当前页数", this.currentPage);
      this.isLoading = true;
      if (form.columns.length == 0) {
        this.$message.warning("请至少输入一个条件!");
      } else {
        //判断年份是否为空
        if (form.year == 1930) {
          console.log("year为空");
          form.year = null;
          console.log(form.year);
        } else {
          console.log(form.year);
        }

        //判断月份或季度
        if (form.month_season == "") {
          console.log("月份季度为空");
          form.month = null;
          form.season = null;
        } else if (form.month_season == "month") {
          console.log("选择月份");
          form.season = null;
        } else if ((form.month_season = "season")) {
          console.log("选择季度");
          form.month = null;
        }

        //判断天数或星期几
        if (form.day_weekday == "") {
          console.log("天数为空");
          form.day = null;
          form.weekday = null;
        } else if (form.day_weekday == "day") {
          console.log("选择天数");
          form.weekday = null;
        } else if ((form.day_weekday = "weekday")) {
          console.log("选择周几");
          form.day = null;
        }
        console.log("这是原始条件", form.columns);
        for (var i = 0; i < form.columns.length; i++) {
          this.columns[form.columns[i]] = true;
          console.log(
            "这是条件",
            form.columns[i],
            this.columns[form.columns[i]]
          );
        }
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

      }
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
          data: ["mysql", "hive"],
        },
        yAxis: {},
        series: [
          {
            name: "查询耗时(s)",
            type: "bar",
            // data: [this.mysql_speed, this.spark_speed],
            data: [this.mysql_speed, this.spark_speed],
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
