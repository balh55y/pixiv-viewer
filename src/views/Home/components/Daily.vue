<template>
  <div class="daily">
    <van-cell class="cell" :border="false">
      <template #title>
        <Icon class="icon random" name="random"></Icon>
        <span class="title">随便看看</span>
      </template>
    </van-cell>
    <van-list
      v-model="loading"
      class="artwork-list"
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
          :data="artList"
        >
          <router-link
            :to="{
              name: 'Artwork',
              params: { id: art.id, list: artList },
            }"
            v-for="art in artList"
            :key="art.id"
          >
            <ImageCard mode="cover" :artwork="art" />
          </router-link>
        </waterfall>
      </div>
    </van-list>
  </div>
</template>

<script>
import { Cell, Swipe, SwipeItem, Icon, List, PullRefresh } from "vant";
import ImageCard from "@/components/ImageCard";
import api from "@/api";
import { throttle, uniqBy } from "lodash-es";
export default {
  name: "Daily",
  data() {
    return {
      col: 2,
      itemWidth: 0,
      curPage: 1,
      artList: [],
      error: false,
      loading: false,
      finished: false,
    };
  },
  methods: {
    url(id, index) {
      return api.url(id, index);
    },
    getRankList: throttle(async function () {
      let res = await api.getRankList("day", this.curPage);
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
    toArtwork(id) {
      this.$router.push({
        name: "Artwork",
        params: { id, list: this.artList },
      });
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
    // this.getLatest();

    window.addEventListener("resize", this.resize);
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.resize);
  },
  components: {
    [Cell.name]: Cell,
    [Swipe.name]: Swipe,
    [SwipeItem.name]: SwipeItem,
    [Icon.name]: Icon,
    [List.name]: List,
    [PullRefresh.name]: PullRefresh,
    ImageCard,
  },
};
</script>

<style lang="stylus" scoped>
.rank-card {
  .card-box {
    padding: 0 12px;
    height: 365px;

    .swipe-wrap {
      height: 100%;
      border-radius: 20px;
      overflow: hidden;

      .swipe-item {
        &:last-child {
          .image-card {
            margin-right: 0;
          }
        }

        .image-card {
          // width: 50vw;
          font-size: 0;
          float: left;
          margin-right: 12px;
          border: 1px solid #ebebeb;
          border-radius: 18px;
          box-sizing: border-box;
        }

        .image-slide {
          border: 1px solid #ebebeb;
          border-radius: 18px;
          box-sizing: border-box;

          .link {
            position: absolute;
            width: 100%;
            height: 100%;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #efefef;

            &::before {
              content: '';
              position: absolute;
              top: 0;
              left: 0;
              right: 0;
              bottom: 0;
              background: rgba(#000, 0.6);
            }

            svg {
              position: absolute;
              top: 50%;
              left: 50%;
              transform: translate(-50%, -55%);
              font-size: 20em;
            }

            div {
              position: absolute;
              left: 50%;
              top: 50%;
              transform: translate(-50%, 80%);
              font-size: 34px;
              text-align: center;
              white-space: nowrap;
            }
          }
        }

        &.more {
          .rank {
            display: flex;
            height: 100%;
            justify-content: center;
            align-items: center;
          }
        }
      }
    }
  }
}

.daily {
  .artwork-list {
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
</style>