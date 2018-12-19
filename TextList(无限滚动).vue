<template>
  <div id="textlist" ref="wrapper">
    <header :class="heaFix&&'heaFix'">我的翻译</header>
    <div class="page-infinite-wrapper" ref="wrapper" :style="{ height: wrapperHeight + 'px' }">

      <mt-loadmore :top-method="loadTop" @top-status-change="handleTopChange" ref="loadmore">

        <ul class="clearfix" v-infinite-scroll="loadMore" infinite-scroll-disabled="loading" infinite-scroll-distance="50">
          <li class="list" v-for="(item,index) in textlist" :key="index" @click='jumpfile(item)'>
            <mt-cell-swipe :right="[
              {
                content: '删除',
                style: { background: '#f45a5a', color: '#fff', lineHeight: '66px',width: '42px', textAlign: 'center' },
                handler: () => messageBoxDelete(item.id)
              }]" class="list_cell fl">
              <mt-cell slot="icon" class="list_left fl" :title="item.name" :label="item.createTimeDesc">
                <img slot="icon" class="fl" :src="getIcon(item.fileType)" alt="">
              </mt-cell>

            </mt-cell-swipe>
          </li>
        </ul>

        <p v-show="loading&&!allLoaded" class="page-infinite-loading">
          <mt-spinner type="fading-circle"></mt-spinner>
          加载中...
        </p>

        <p style='text-align:center;' v-show="allLoaded">已全部加载</p>

        <div slot="top" class="mint-loadmore-top">
          <span v-show="topStatus !== 'loading'" :class="{ 'rotate': topStatus === 'drop' }">↓</span>
          <span v-show="topStatus === 'loading'">加载中...</span>
        </div>

      </mt-loadmore>
    </div>
  </div>
</template>

<script>
import { MessageBox } from 'mint-ui';
import { mapState, mapActions, mapMutations } from "vuex";

export default {
  name: "TextList",
  data() {
    return {
      loading: false,  //若为真，则无限滚动不会被触发
      topStatus: '',
      wrapperHeight: 0,
      page: {
        size: 20, //获取个数
        total: "", //分页总数
        current: 0 //当前页
      }
    };
  },
  props: {
    textlist: Array,
    heaFix: Boolean
  },
  computed: {
    ...mapState({
      deleteData: state => state.home.deleteData,
      tableData: state => state.home.tableData,
      allLoaded: state => state.home.allLoaded,  //若为真，加载完毕
      tId: state => state.fileView.tId,
      host: state => state.home.host,
      files: state => state.fileView.files
    })
  },
  mounted() {
    this.wrapperHeight = document.documentElement.clientHeight - this.$refs.wrapper.getBoundingClientRect().top;
  },
  methods: {
    ...mapActions([
      "DELETEFILELIBSASYNC",
      "GETFILELIBSASYNC",
      "ENCRYPT",
      "LINKFILE"
    ]),
    ...mapMutations([
      "Filename"
    ]),
    //ui事件
    messageBoxDelete(id) {
      MessageBox.confirm('', {   message: '确定要删除吗', title: '提示', confirmButtonText: '确定', cancelButtonText: '取消'
      }).then(action => {
        if (action === 'confirm') this.deletefile(id)   //确认的回调
      }).catch(err => {
        if (err === 'cancel') return void (666)  //取消的回调
      });
    },

    handleTopChange(status) {
      this.topStatus = status;
    },

    async loadMore() {
      console.log(this.page.current)
      if (!this.allLoaded) {
        this.loading = true;
        await this.$emit('getfiles', this.page.current, this.textlist)
        setTimeout(() => { this.loading = false; }, 500);
        this.page.current += 1;
      }
    },
    async loadTop() {
      await setTimeout(() => {
        let textlist = [];
        this.$emit('getfiles', 0, textlist);
        this.$refs.loadmore.onTopLoaded();
      }, 500)
    },

    getIcon(fileType) {
      switch (fileType) {
        case 0:
          return "static/images/home/img_pdf.png";
          break;
        case 10:
          return "static/images/home/img_text.png";
          break;
        case 20:
          return "static/images/home/img_word.png";
          break;
        case 30:
          return "static/images/home/img_excel.png";
          break;
        case 40:
          return "static/images/home/img_ppt.png";
          break;
        case 60:
          return "static/images/home/img_jpg.png";
          break;
        default:
          return "static/images/home/img_wb.png";
          break;
      }
    },
    //删除某条
    async deletefile(id) {
      console.log(id)
      let data = { ids: [id] };
      await this.DELETEFILELIBSASYNC(data);
      if (this.deleteData.status === 200) {
        this.textlist.map((item, index) => (item.id === id) && this.textlist.splice(index, 1))
      }
    },
    //预览
    async jumpfile(item) {
      console.log(item)
      let id=item.id;
      let name=item.name;
      await Promise.all([this.LINKFILE(id), this.Filename(item)]);
      this.$router.push({ path: '/fileView', query: { tgtViewFile: this.files.data.tgtViewFile,id:id } });
    },
    //分享
    // async dingShare(item) {
    //   // item.tgtFileSize/1024
    //   await this.ENCRYPT(item.id);
    //   dd.biz.util.share({
    //     type: 0,//分享类型，0:全部组件 默认； 1:只能分享到钉钉；2:不能分享，只有刷新按钮,
    //     url: this.host + "/api/File/ShareTgtFile?t=" + encodeURIComponent(this.tId),
    //     title: item.name,
    //     content: '由云译通翻译',
    //     image: String,
    //     onSuccess: () => {
    //       //onSuccess将在调起分享组件成功之后回调
    //     },
    //     onFail: (err) => { }
    //   })
    // }
  }
};
</script>

<style scoped lang='less'>
//mixin
.line {
  content: "";
  position: absolute;
  top: 51%;
  background: #373737;
  width: 25px;
  height: 1px;
}

//cont
header {
  color: #373737;
  font-size: 13px;
  text-align: center;
  line-height: 23px;
  margin-top: 15px;
  position: relative;
  transition: all 0.3s ease-in-out;
  &::before {
    .line;
    left: 35%;
  }
  &:after {
    .line;
    right: 35%;
  }
}

.heaFix {
  position: fixed;
  width: 100%;
  z-index: 50;
  top: 0;
  margin-top: 0;
  background-color: white;
}

/deep/.list {
  &_cell {
    width: 100%;
  }
  &_left {
    width: 100%;
    height: 66px;
    display: flex;
    align-items: center;
    .mint-cell-wrapper {
      padding: 0;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      background-position: bottom;
      background-image: linear-gradient(180deg, #d9d9d9, #d9d9d9 1%, transparent 1%);
      background-position: bottom;
    }
    img {
      display: inline-block;
      margin-right: 5px;
      width: 24px;
      height: 29px;
    }
  }
  /deep/.mint-cell {
    background-position: bottom;
    border: none!important;
    background: none!important;
  }
}

.page-infinite-loading {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
