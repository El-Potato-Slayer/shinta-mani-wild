<template>
  <!-- design file: https://www.figma.com/file/SiFZE7hhRKx2fWmrfZ3uy2RO/Shinta-Mani-Wild?node-id=553%3A4724 -->
  <div class="page page--search">
    <div class="page--header-content">
      <!-- header -->
      <page-header></page-header>

      <loading-progress />

      <div class="page--content">

      <!-- player -->
      <div class="hero position-relative">
        <video-player
          v-if="resort.id"
          :source="resort.name"
          :rest="{autoplay: true, loop: false}"
        ></video-player>
      </div>

      <div class="parallax-container position-relative py-5">
        <div class="container is-small page-description mb-5 clearfix">
          <article>
            <base-heading
              :show-placeholder="!resort.id"
              :text="resort.title"
              :type="'h1'"
              :class-placeholder="'heading-placeholder mb-5'"
              :class-name="'h1 text-dark text-center mb-5'"
              :border-art="true"
            ></base-heading>

            <div v-if="!resort.id">
              <content-placeholders centered rounded class="description-placeholder">
                <content-placeholders-text :lines="3" />
              </content-placeholders>
            </div>
            <p
              v-else
              class="mb-0"
              v-text="resort.description"
            ></p>
          </article>
        </div>

        <!-- featured stories -->
        <section class="container is-small featured-items is-huge">
          <base-articles-list
            :route-props="{ returnTo: 'search', resortId: $route.params.id }"
            :image-box-class="'ratio-16-9'"
            :title-class="'font-weight-normal'"
            :show-placeholder="!resort.id"
            :items-per-row="2"
            :column-classes="'col-6'"
            :items="stories.slice(0,featuredStoriesCount)"
            preview-transformations="q_auto:low,e_preview:duration_10,w_440,h_248,c_fill,ar_16:9,ac_none"
            poster-transformations="q_auto:good,w_440,h_248,c_fill,g_auto"
          ></base-articles-list>
        </section>

        <template v-for="(doodle, index) in pageDoodles.slice(0, 2)">
          <img
            @load="setItemParallax($event)"
            :class="`doodle doodle-item-0-${index} position-absolute`"
            data-aos="fade-down"
            :data-rellax-speed="getRellaxSpeed()"
            :src="transformCloudinaryUrl(doodle.url, 'q_auto:low,fl_any_format,o_20,h_700,w_700,c_limit')"
            :key="index"
            alt
          />
        </template>
      </div>

      <!-- banner action -->
      <div>
        <base-banner-action
          :image="resort.backgroundImage"
          :show-placeholder="!resort.id"
          :link="resort.ctaLink"
          :text="resort.ctaText"
          :button-text="'Book Now'"
        ></base-banner-action>
      </div>

      <div class="parallax-container position-relative py-5">
        <!-- quote -->
        <section class="container is-small mb-5">
          <base-quote :type="'grass1'">
            <div class="quote" v-html="resort.h2"></div>
          </base-quote>
        </section>

        <!-- articles (stories) -->
        <div class="container is-small">
          <!-- NOTE: slice items from `items-offset` to end (because some of items are in use in featured section) -->
          <base-articles-list
            :route-props="{ returnTo: 'search', resortId: $route.params.id }"
            :show-placeholder="!resort.id"
            :items-offset="featuredStoriesCount"
            :items="stories"
            preview-transformations="q_auto:low,e_preview:duration_8,w_212,c_fill,ar_1:1,ac_none"
            poster-transformations="q_auto:best,w_212,h_212,c_fill,g_auto"
          ></base-articles-list>
        </div>

        <template
          v-for="(doodle, index) in pageDoodles.slice(2, (relativeDoodleAmount(stories.length, featuredStoriesCount, 2) || 4))"
        >
          <img
            @load="setItemParallax($event)"
            :class="`doodle doodle-item-1-${index} position-absolute`"
            data-aos="fade-down"
            :data-rellax-speed="getRellaxSpeed()"
            :src="transformCloudinaryUrl(doodle.url, 'q_auto:low,fl_any_format,o_20,h_700,w_700,c_limit')"
            :key="index"
            alt
          />
        </template>
      </div>
    </div>
    </div>

    <page-footer></page-footer>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import PageHeader from '@/components/PageHeader.vue'
import VideoPlayer from '@/components/VideoPlayer.vue'
import PageFooter from '@/components/PageFooter.vue'
import HeroImage from '@/components/HeroImage.vue'
import BaseHeading from '@/components/BaseHeading.vue'
import BaseBannerAction from '@/components/BaseBannerAction.vue'
import BaseArticlesList from '@/components/BaseArticlesList.vue'
import BaseQuote from '@/components/BaseQuote.vue'
import doodles from '@/mixins/doodles'
import loading from '@/mixins/loading'
import { Story, Resort, Category } from '@/types'
import { get } from 'lodash-es'
import { MetaInfo } from 'vue-meta'

export default Vue.extend({
  name: 'listing',
  mixins: [doodles, loading],
  components: {
    PageHeader,
    VideoPlayer,
    PageFooter,
    HeroImage,
    BaseHeading,
    BaseBannerAction,
    BaseArticlesList,
    BaseQuote
  },
  data() {
    return {
      featuredStoriesCount: 2,
      slug: this.$route.params.id
    }
  },
  metaInfo (): MetaInfo {
    return {
      title: this.resort.title,
      meta: [
        { vmid: 'description', name: 'description', content: this.resort.description }
      ]
    }
  },
  computed: {
    resort(): Resort {
      return this.$store.getters['resort/getItem']
    },
    stories(): Story[] {
      return get(this.resort, 'stories', []).filter(
        (item: Story) => item.posterUrl
      )
    }
  },
  mounted() {
    this.$store.dispatch('resort/getItemBySlug', this.slug)
  }
})
</script>

<style lang="scss" scoped>
.featured-items {
  @include media-breakpoint-up(md) {
    min-height: rem(356px);
  }
  &::v-deep {
    .vue-content-placeholders-img {
      height: rem(248px);
    }
  }
}
</style>
