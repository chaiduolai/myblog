<template>
<html>
  <Head></Head>

  <body>
    <!--
	作者：xzx19950624@qq.com
	时间：2018-07-15
	描述：顶部标题
    -->
    <BlogHead></BlogHead>

    <div class="pagebg classify"></div>
    <div class="container">
      <h1 class="t_nav">
        <span>不要轻易放弃。学习成长的路上，我们长路漫漫，只因学无止境。</span>
        <a href="/" class="n1">网站首页</a>
        <a href="javascript:void(0);" class="n2">分类</a>
      </h1>

      <div class="sortBox">
        <div class="time">
          <div class="block">
            <div class="radio" style="margin-bottom:20px;"></div>
            <el-timeline :reverse="reverse">
              <el-timeline-item v-for="(activity, index) in activities" hide-timestamp :key="index">
                <span
                  @click="getBlogList(activity.uid)"
                  :class="[activity.uid == selectBlogUid ? 'sortBoxSpan sortBoxSpanSelect' : 'sortBoxSpan']"
                >{{activity.sortName}}</span>
              </el-timeline-item>
            </el-timeline>
          </div>
        </div>

        <div class="article">
          <div class="block" v-infinite-scroll="load">
            <el-timeline>
              <el-timeline-item
                v-for="item in itemByDate"
                :key="item.uid"
                :timestamp="item.createTime"
                placement="top"
              >
                <el-card>
                  <h4 @click="goToList('blogContent', item.uid)" class="itemTitle">{{item.title}}</h4>
                  <br>
                  <el-tag class="elTag" v-if="item.isOriginal==1" type="danger">原创</el-tag>
                  <el-tag class="elTag" v-if="item.isOriginal==0" type="info">转载</el-tag>

                  <el-tag
                    class="elTag"
                    v-if="item.author"
                    @click="goToList('author', item.author)"
                  >{{item.author}}</el-tag>

                  <el-tag
                    class="elTag"
                    type="success"
                    v-if="item.blogSort != null"
                    @click="goToList('blogSort', item.blogSort.uid)"
                  >{{item.blogSort.sortName}}</el-tag>
                  <el-tag
                    class="elTag"
                    v-for="tagItem in item.tagList"
                    v-if="tagItem != null"
                    :key="tagItem.uid"
                    @click="goToList('tag', tagItem.uid)"
                    type="warning"
                  >{{tagItem.content}}</el-tag>
                </el-card>
              </el-timeline-item>
            </el-timeline>
          </div>
        </div>
      </div>
    </div>

    <!--
	作者：xzx19950624@qq.com
	时间：2018-07-15
	描述：博客底部
    -->
    <BlogFooter></BlogFooter>

    <!--返回顶部-->
    <CdTop></CdTop>
  </body>
</html>
</template>

<script>
import Head from "../components/Head";
import BlogHead from "../components/BlogHead";
import BlogFooter from "../components/BlogFooter";
import CdTop from "../components/CdTop";
import { recorderVisitPage } from "../api/index";
import { getBlogSortList, getArticleByBlogSortUid } from "../api/classify";
export default {
  data() {
    return {
      selectBlogUid: "",
      reverse: false,
      activities: [],
      itemByDate: [],
      articleByDate: {},
      count: 0,
      currentPage: 1,
      pageSize: 10
    };
  },
  components: {
    //注册组件
    BlogHead,
    BlogFooter,
    Head,
    CdTop
  },
  mounted() {},
  created() {
    var that = this;
    getBlogSortList().then(response => {
      if (response.code == "success") {
        var activities = response.data;
        var result = [];
        for (var a = 0; a < activities.length; a++) {
          var dataForDate = {
            sortName: activities[a].sortName,
            uid: activities[a].uid
          };
          result.push(dataForDate);
        }
        this.activities = result;

        // 默认选择第一个
        this.getBlogList(activities[0].uid);
      }
    });

    var params = new URLSearchParams();
    params.append("pageName", "CLASSIFY");
    recorderVisitPage(params).then(response => {});
  },
  methods: {
    getBlogList(blogSortUid) {
      console.log("点击获取列表", blogSortUid);
      this.selectBlogUid = blogSortUid;
      var params = new URLSearchParams();
      params.append("blogSortUid", blogSortUid);
      getArticleByBlogSortUid(params).then(response => {
        console.log("返回的内容", response);
        if (response.code == "success") {
          this.itemByDate = response.data.records;
          this.currentPage = response.data.current;
          this.pageSize = response.data.size;
        }
      });
    },
    load() {      
      var params = new URLSearchParams();
      if(this.selectBlogUid == null || this.selectBlogUid == "" || this.selectBlogUid == undefined) {
        return;
      }
      params.append("blogSortUid", this.selectBlogUid);
      params.append("currentPage", this.currentPage + 1);
      getArticleByBlogSortUid(params).then(response => {        
        if (response.code == "success") {
          this.itemByDate = this.itemByDate.concat(response.data.records);
          this.currentPage = response.data.current;
          this.pageSize = response.data.size;
        }
      });
    },
    //跳转到搜索详情页
    goToList(type, uid) {
      switch (type) {
        case "tag":
          {
            let routeData = this.$router.resolve({
              path: "/list",
              query: { tagUid: uid }
            });
            window.open(routeData.href, "_blank");
          }
          break;
        case "blogSort":
          {
            let routeData = this.$router.resolve({
              path: "/list",
              query: { sortUid: uid }
            });
            window.open(routeData.href, "_blank");
          }
          break;

        case "author":
          {
            let routeData = this.$router.resolve({
              path: "/list",
              query: { author: uid }
            });
            window.open(routeData.href, "_blank");
          }
          break;

        case "blogContent":
          {
            let routeData = this.$router.resolve({
              path: "/info",
              query: { blogUid: uid }
            });
            window.open(routeData.href, "_blank");
          }
          break;
      }
    }
  }
};
</script>


<style>
.sortBox {
  color: #555;
}
.sortBox .time {
  float: left;
  width: 17%;
  height: 800px;
  overflow: scroll;
  overflow-x: hidden;
  overflow-y: auto;
}

.sortBox .article {
  margin-left: 20px;
  float: left;
  width: 78%;
  height: 800px;
  overflow: scroll;
  overflow-x: hidden;
  overflow-y: auto;
}

.sortBoxSpan {
  cursor: pointer;
}
.sortBoxSpan:hover {
  color: #409eff;
}
.sortBoxSpanSelect {
  color: #409eff;
}

.itemTitle {
  cursor: pointer;
}
.itemTitle:hover {
  color: #409eff;
}
.elTag {
  cursor: pointer;
}
</style>
