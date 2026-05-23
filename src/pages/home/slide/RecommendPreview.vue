<script setup>
import { reactive, watch } from 'vue'
import ScrollList from '@/components/ScrollList.vue'
import { recommendedVideo } from '@/api/videos'
import { useNav } from '@/utils/hooks/useNav'
import { _checkImgUrl, _duration, _formatNumber, _stopPropagation } from '@/utils'

const props = defineProps({
  active: {
    type: Boolean,
    default: false
  }
})

const nav = useNav()
const state = reactive({
  show: false
})

watch(
  () => props.active,
  (active) => {
    if (active) state.show = true
  },
  { immediate: true }
)

function getData(params = {}) {
  const pageSize = params.pageSize ?? 10
  const pageNo = params.pageNo ?? 0
  return recommendedVideo({
    start: pageNo * pageSize,
    pageSize
  })
}

function toCards(list) {
  return list.map((item, index) => ({ item, index }))
}

function splitCards(list, mod) {
  return toCards(list).filter((card) => card.index % 2 === mod)
}

function cardClass(index) {
  return {
    tall: index % 5 === 1 || index % 5 === 2,
    compact: index % 5 === 0
  }
}

function openDetail(list, index) {
  nav('/video-detail', {}, { list, index })
}
</script>

<template>
  <div class="recommend-preview" @dragstart="(e) => _stopPropagation(e)">
    <ScrollList class="Scroll" v-if="state.show" :api="getData">
      <template v-slot="{ list }">
        <div class="preview-grid">
          <div class="preview-column">
            <article
              class="preview-card"
              :class="cardClass(entry.index)"
              :key="entry.item.aweme_id || entry.index"
              v-for="entry in splitCards(list, 0)"
              @click="openDetail(list, entry.index)"
            >
              <div class="poster-wrap">
                <img
                  class="poster"
                  v-lazy="_checkImgUrl(entry.item.video.cover.url_list[0])"
                  alt=""
                />
                <div class="metrics">
                  <span class="like">
                    <Icon icon="solar:heart-linear" />
                    {{ _formatNumber(entry.item.statistics.digg_count) }}
                  </span>
                  <span>{{ _duration(entry.item.duration / 1000) }}</span>
                </div>
              </div>
              <div class="title">{{ entry.item.desc }}</div>
              <div class="meta">
                <span class="followed" v-if="entry.index % 2 === 0">已关注</span>
                <img
                  v-else
                  class="avatar"
                  v-lazy="_checkImgUrl(entry.item.author.avatar_168x168.url_list[0])"
                  alt=""
                />
                <span class="author">{{ entry.item.author.nickname }}</span>
                <Icon class="more" icon="tabler:dots" />
              </div>
            </article>
          </div>
          <div class="preview-column">
            <article
              class="preview-card"
              :class="cardClass(entry.index)"
              :key="entry.item.aweme_id || entry.index"
              v-for="entry in splitCards(list, 1)"
              @click="openDetail(list, entry.index)"
            >
              <div class="poster-wrap">
                <img
                  class="poster"
                  v-lazy="_checkImgUrl(entry.item.video.cover.url_list[0])"
                  alt=""
                />
                <div class="metrics">
                  <span class="like">
                    <Icon icon="solar:heart-linear" />
                    {{ _formatNumber(entry.item.statistics.digg_count) }}
                  </span>
                  <span>{{ _duration(entry.item.duration / 1000) }}</span>
                </div>
              </div>
              <div class="title">{{ entry.item.desc }}</div>
              <div class="meta">
                <span class="followed" v-if="entry.index % 2 === 0">已关注</span>
                <img
                  v-else
                  class="avatar"
                  v-lazy="_checkImgUrl(entry.item.author.avatar_168x168.url_list[0])"
                  alt=""
                />
                <span class="author">{{ entry.item.author.nickname }}</span>
                <Icon class="more" icon="tabler:dots" />
              </div>
            </article>
          </div>
        </div>
      </template>
    </ScrollList>
  </div>
</template>

<style scoped lang="less">
.recommend-preview {
  height: 100%;
  padding-top: var(--home-header-height);
  box-sizing: border-box;
  background: #fff;
  color: var(--jx-text);
  font-size: 14rem;

  .Scroll {
    height: calc(
      var(--vh, 1vh) * 100 - var(--home-header-height) - var(--footer-height)
    ) !important;
  }
}

.preview-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  gap: 6rem;
  padding: 0 5rem 14rem;
  box-sizing: border-box;
}

.preview-column {
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 10rem;
}

.preview-card {
  min-width: 0;
  overflow: hidden;
  background: #fff;
  border-radius: 6rem;

  .poster-wrap {
    position: relative;
    height: 186rem;
    overflow: hidden;
    border-radius: 6rem;
    background: var(--jx-surface-soft);
  }

  &.tall {
    .poster-wrap {
      height: 244rem;
    }
  }

  &.compact {
    .poster-wrap {
      height: 154rem;
    }
  }

  .poster {
    display: block;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .metrics {
    position: absolute;
    left: 8rem;
    right: 8rem;
    bottom: 7rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    color: white;
    font-size: 13rem;
    text-shadow: 0 1rem 6rem rgba(0, 0, 0, 0.48);

    .like {
      display: inline-flex;
      align-items: center;
      gap: 3rem;

      svg {
        font-size: 16rem;
      }
    }
  }

  .title {
    padding: 8rem 4rem 0;
    color: var(--jx-text);
    font-size: 16rem;
    line-height: 1.45;
    word-break: break-all;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
    overflow: hidden;
  }

  .meta {
    min-width: 0;
    padding: 8rem 4rem 4rem;
    display: flex;
    align-items: center;
    gap: 5rem;
    color: var(--jx-text-muted);
    font-size: 13rem;

    .followed {
      color: #d76282;
      white-space: nowrap;
    }

    .avatar {
      width: 18rem;
      height: 18rem;
      flex-shrink: 0;
      border-radius: 50%;
      object-fit: cover;
    }

    .author {
      min-width: 0;
      flex: 1;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .more {
      flex-shrink: 0;
      font-size: 18rem;
      color: #a3a8b4;
    }
  }
}
</style>
