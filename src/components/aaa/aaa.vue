<template>
  <div class="search">
    <div class="search-box-wrapper">
      <search-box ref="searchBox" @query="onQueryChange"></search-box>
    </div>
    <div ref="shortcutWrapper" class="shortcut-wrapper" v-show="!query">
      <scroll ref="shortcut" class="shortcut" :data="shortcut" :refreshDelay="refreshDelay">
        <div>
          <div class="hot-key">
            <h1 class="title">热门搜索</h1>
            <ul>
              <li class="item" v-for="item in hotKey" :key="item.id" @click="addQuery(item.k)">
                <span>{{item.k}}</span>
              </li>
            </ul>
          </div>
          <!-- <div class="search-history"> -->
            <div class="search-history" v-show="searchHistory.length > 0">
            <h1 class="title">
              <span class="text">搜索历史</span>
              <span @click="showConfirm" class="clear">
                <i class="icon-clear"></i>
              </span>
            </h1>
            <search-list @delete="deleteSearchHistory" @select="addQuery" :searches="searchHistory"></search-list>
          </div>
        </div>
      </scroll>
    </div>
    <div ref="searchResult" class="search-result" v-show="query">
      <suggest @select="saveSearch" :query="query" @listScroll="blurInput" ref="suggest"></suggest>
    </div>
    <confirm ref="confirm" text="是否清空所有搜索历史" confirmBtnText="清空"
    @confirm="clearSearchHistory"></confirm>
    <router-view></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
import SearchBox from '../../base/search-box/search-box';
import { getHotKey } from '../../api/search';
import { ERR_OK } from '../../api/config';
import Suggest from '../suggest/suggest';
import { mapActions } from 'vuex';
import SearchList from '../../base/search-list/search-list';
import Confirm from '../../base/confirm/confirm';
import Scroll from '../../base/scroll/scroll';
import {playlistMixin, searchMixin} from '../../common/js/mixin';

export default {
  mixins: [playlistMixin, searchMixin],
  components: {
    SearchBox,
    Suggest,
    SearchList,
    Confirm,
    Scroll
  },
  computed: {
    shortcut () {
      return this.hotKey.concat(this.searchHistory);
    }
  },
  created () {
    this._getHotKey();
  },
  data () {
    return {
      hotKey: []
    };
  },
  methods: {
    handlePlaylist (playlist) {

        // const bottom = this.playlist.length > 0 ? '60px' : '';

        // this.$refs.searchResult.style.bottom = bottom;
        // this.$refs.suggest.refresh();

        // this.$refs.shortcutWrapper.style.bottom = bottom;
        // this.$refs.shortcut.refresh();
    },
    // deleteAll () {
    //   this.clearSearchHistory();
    // },
    showConfirm () {
      this.$refs.confirm.show();
    },
    _getHotKey () {
      getHotKey().then(res => {
        if (res.code === ERR_OK) {
          // 取前10个数据作为热门数据
          this.hotKey = res.data.hotkey.slice(0, 10);
        }
      });
    },
    ...mapActions([
      'clearSearchHistory'
      ])
  },
  watch: {
    query (newQuery) {
      if (!newQuery) {
        setTimeout(() => {
          this.$refs.shortcut.refresh();
        }, 20);
      }
    }
  }
};
</script>

<style lang="less" rel="stylesheet/less">
@import "../../common/less/variable";
@import "../../common/less/mixin";

.search {
  .search-box-wrapper {
    margin: 20px;
  }
  .shortcut-wrapper {
    position: fixed;
    top: 178px;
    bottom: 0;
    width: 100%;
    .shortcut {
      height: 100%;
      overflow: hidden;
      .hot-key {
        margin: 0 20px 20px 20px;
        .title {
          margin-bottom: 20px;
          font-size: @font-size-medium;
          color: @color-text-l;
        }
        .item {
          display: inline-block;
          padding: 5px 10px;
          margin: 0 20px 10px 0;
          border-radius: 6px;
          background: @color-highlight-background;
          font-size: @font-size-medium;
          color: @color-text-d;
        }
      }
      .search-history {
        position: relative;
        margin: 0 20px;
        .title {
          display: flex;
          align-items: center;
          height: 40px;
          font-size: @font-size-medium;
          color: @color-text-l;
          .text {
            flex: 1;
          }
          .clear {
            .extend-click();
            .icon-clear {
              font-size: @font-size-medium;
              color: @color-text-d;
            }
          }
        }
      }
    }
  }
  .search-result {
    position: fixed;
    width: 100%;
    top: 178px;
    bottom: 0;
  }
}
</style>
