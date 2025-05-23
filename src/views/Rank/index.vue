<template>
  <div class="rank">
    <div class="top">
      <Nav :menu="menu" />
      <v-date-picker
        :attributes="[
          {
            key: 'today',
            highlight: 'yellow',
            dates: date,
          },
        ]"
        :min-date="minDate"
        :max-date="maxDate"
        v-model="date"
        mode="single"
        :popover="{
          placement: 'bottom',
          visibility: 'click',
        }"
        :masks="{ title: 'YYYY年 MMM' }"
        locale="zh"
      >
        <div class="calendar">
          <div class="date">{{ dateNum }}</div>
        </div>
      </v-date-picker>
    </div>
    <Top3 v-if="artList.length >= 3" :artList="artList.slice(0, 3)" />
    <van-list
      v-if="artList.length > 3"
      class="rank-list"
      v-model="loading"
      :finished="finished"
      finished-text="没有更多了"
      :error.sync="error"
      error-text="网络异常，点击重新加载"
      @load="getRankList"
    >
      <div class="card-box__wrapper" ref="cardBox">
        <waterfall
          :col="col"
          :width="itemWidth"
          :gutterWidth="0"
          :data="artList.slice(3)"
        >
          <router-link
            :to="{
              name: 'Artwork',
              params: { id: art.id, list: artList },
            }"
            v-for="art in artList.slice(3)"
            :key="art.id"
          >
            <ImageCard mode="cover" :artwork="art" />
          </router-link>
        </waterfall>
      </div>
    </van-list>
    <van-loading
      v-show="!artList || artList.length === 0"
      class="loading"
      :size="'50px'"
    />
  </div>
</template>

<script>
import dayjs from "dayjs";
import { List, Loading, Empty } from "vant";
import ImageCard from "@/components/ImageCard";
import Nav from "./components/Nav";
import Top3 from "./components/Top3";
import { throttle, uniqBy } from "lodash-es";
import api from "@/api";
export default {
  name: "Rank",
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      document.documentElement.scrollTo(0, vm.scrollTop);
    });
  },
  beforeRouteLeave(to, from, next) {
    this.scrollTop = document.documentElement.scrollTop;
    next();
  },
  data() {
    return {
      col: 2,
      itemWidth: 0,
      scrollTop: 0,
      minDate: dayjs("2007-09-13").toDate(),
      maxDate: dayjs().subtract(2, "days").toDate(),
      date: dayjs().subtract(2, "days").toDate(),
      isDatePickerShow: false,
      curType: "daily",
      curPage: 1,
      artList: [],
      error: false,
      loading: false,
      finished: false,
      menu: {
        daily: { name: "日榜", io: "day" },
        weekly: { name: "周榜", io: "week" },
        monthly: { name: "月榜", io: "month" },
        rookie: { name: "新人榜", io: "week_rookie" },
        original: { name: "原创榜", io: "week_original" },
        male: { name: "男性向", io: "day_male" },
        female: { name: "女性向", io: "day_female" },
        r18: { name: "R-18 - 日榜", io: "day_r18", x: true },
        "r18-w": { name: "R-18 - 周榜", io: "week_r18", x: true },
        "r18-m": { name: "R-18 - 男性向", io: "day_male_r18", x: true },
        "r18-f": { name: "R-18 - 女性向", io: "day_female_r18", x: true },
      },
    };
  },
  computed: {
    dateNum() {
      return dayjs(this.date).date();
    },
  },
  watch: {
    $route() {
      if (
        this.$route.name === "Rank" &&
        this.$route.params.type !== this.curType
      ) {
        this.init();
      }
    },
    date(val, old) {
      if (val !== old) {
        this.init();
      }
    },
  },
  methods: {
    reset() {
      this.curType = "daily";
      this.curPage = 1;
      this.artList = [];
    },
    init() {
      this.reset();
      this.curType = this.$route.params.type;
      // console.log(this.curType, this.$route);
      this.getRankList();
    },
    getIOType(type) {
      return this.menu[type] ? this.menu[type].io : null;
    },
    getRankList: throttle(async function () {
      let type = this.getIOType(this.curType);
      let res = await api.getRankList(type, this.curPage, this.date);
      if (res.status === 0) {
        let newList = res.data;
        let artList = JSON.parse(JSON.stringify(this.artList));

        artList.push(...newList);
        artList = uniqBy(artList, "id");

        this.artList = artList;
        this.loading = false;
        this.curPage++;
        if (this.curPage > 5) this.finished = true;
        this.$nextTick(this.resize);
      } else {
        this.$toast({
          message: res.msg,
        });
        this.loading = false;
        this.error = true;
      }
      this.isLoading = false;
    }, 5000),
    odd(list) {
      return list.filter((_, index) => (index + 1) % 2);
    },
    even(list) {
      return list.filter((_, index) => !((index + 1) % 2));
    },
    toArtwork(id) {
      this.$router.push({
        name: "Artwork",
        params: { id, list: this.artList },
      });
    },
    showPopup() {
      this.isDatePickerShow = true;
    },
    resize() {
      if (!this.$refs.cardBox) return;
      const clientWidth = document.documentElement.clientWidth;

      if (clientWidth < 375) {
        this.col = 1;
      } else if (clientWidth <= 768) {
        this.col = 2;
      } else if (clientWidth <= 1600) {
        this.col = 3;
      } else {
        this.col = 4;
      }

      this.itemWidth = Math.floor(
        this.$refs.cardBox.firstChild.clientWidth / this.col
      );
    },
  },
  mounted() {
    this.init();
    window.addEventListener("resize", this.resize);
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.resize);
  },
  components: {
    Nav,
    Top3,
    [List.name]: List,
    [Loading.name]: Loading,
    [Empty.name]: Empty,
    ImageCard,
  },
};
</script>

<style lang="stylus" scoped>
.rank {
  padding-top: 100px;
  padding-top: calc(100px + env(safe-area-inset-top));
  // height: 100%;
  box-sizing: border-box;

  .loading {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .top {
    position: fixed;
    display: flex;
    justify-content: space-between;
    align-items: center;
    top: 60px;
    top: env(safe-area-inset-top);
    width: 100%;
    max-width: 750px;
    height: 100px;
    padding: 0 12px;
    box-sizing: border-box;
    background: #fff;
    z-index: 1;

    .calendar {
      position: relative;
      width: 60px;
      height: 60px;
      background: url('~@/assets/images/calendar.png') center no-repeat;
      background-size: 100%;
      transform: translateY(-4px);

      .date {
        position: absolute;
        top: 24px;
        left: 55%;
        transform: translateX(-50%);
        color: #666;
        font-family: Dosis;
        font-size: 24px;
        font-weight: 600;
        letter-spacing: 3px;
      }
    }

    ::v-deep .vc-popover-content-wrapper {
      top: 90px !important;
      left: auto !important;
      right: 14px;
      transform: none !important;

      .vc-popover-caret {
        left: 94% !important;
      }
    }
  }

  .rank-list {
    margin: 0 2px;

    .card-box__wrapper {
      .card-box {
        display: flex;
        flex-direction: row;
      }

      .image-card {
        max-height: 500px;
        margin: 14px 6px;
        border: 1px solid #ebebeb;
      }
    }
  }
}

@media screen and (min-width: 768px) {
  .rank {
    .top {
      max-width: 1200px;
    }
  }
}

@media screen and (min-width: 1700px) {
  .rank {
    .top {
      max-width: 1600px;
    }
  }
}
</style>