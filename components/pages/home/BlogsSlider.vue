<template>
  <div>
    <div
      v-if="enableList && !isFetching"
      v-swiper:mySwiper="swiperOptionBlog"
      class="swiper-container"
    >
      <div class="swiper-wrapper">
        <template v-for="(item, index) in itemList">
          <BlogCard
            :key="'blog-slide-' + index"
            :blog="item"
            class="swiper-slide swiper-slide--rex"
          />
        </template>
      </div>
      <div
        slot="pagination"
        style="bottom: -2px"
        class="swiper-pagination md-d-none-sm"
      ></div>
    </div>
    <div
      v-if="isFetching || !enableList"
      v-swiper:mySwiper="swiperOptionBlog"
      class="swiper-container"
    >
      <div class="swiper-wrapper md-shimmer">
        <template v-for="i in 3">
          <BlogCardShimmer
            :key="'blog-shimmer-' + i"
            class="swiper-slide swiper-slide--rex"
          />
        </template>
      </div>
    </div>
  </div>
</template>

<style>
.pmh-blog__posts.swiper-slide {
  margin-top: 16px;
  margin-bottom: 32px;
}
</style>

<script>
import { mapGetters } from 'vuex'
import BlogCardShimmer from '@/components/common/BlogCardShimmer'
import BlogCard from '../../common/BlogCard'

export default {
  components: { BlogCardShimmer, BlogCard },
  data: () => ({
    assetsUrl: process.env.assetsUrl,
    isFetching: true,
    swiperOptionBlog: {
      slidesPerView: 3,
      spaceBetween: 24,
      autoplay: true,
      loop: false,
      pagination: {
        el: '.swiper-pagination',
        clickable: true
      },
      breakpoints: {
        // when window width is <= 480px
        768: {
          slidesPerView: 1,
          spaceBetween: 16
        },
        // when window width is <= 640px
        959: {
          slidesPerView: 2,
          spaceBetween: 24
        }
      }
    }
  }),
  computed: {
    ...mapGetters({
      itemList: 'blogs/getBlogs'
    }),
    enableList() {
      return this.itemList.length
    }
  },
  mounted() {
    if (!(this.itemList && this.itemList.length)) {
      this.fetchBlogs()
    } else {
      this.isFetching = false
    }
  },
  methods: {
    async fetchBlogs() {
      this.isFetching = true
      try {
        await this.$store.dispatch('blogs/fetchBlogs')
      } catch (e) {
        // eslint-disable-next-line no-console
        console.log(e)
      }
      this.isFetching = false
    }
  }
}
</script>
