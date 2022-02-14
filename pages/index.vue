<template>
  <div>
    <section class="hero">
      <div class="hero-body">
        <div class="container">
          <div v-for="post in posts.slice(0, 2)" v-bind:key="post.slug">
            <div class="columns">
              <div class="column is-8 is-offset-2">
                <figure class="image">
                  <datocms-image :data="post.coverImage.responsiveImage" />
                </figure>
              </div>
            </div>

            <section class="section">
              <div class="columns">
                <div class="column is-8 is-offset-2">
                  <div class="content is-medium">
                    <h2 class="subtitle is-4">
                      {{ formatDate(post.publicationDate) }}
                    </h2>
                    <h1 class="title">
                      <nuxt-link :to="`/posts/${post.slug}`">{{
                        post.title
                      }}</nuxt-link>
                    </h1>
                    <div v-html="post.excerpt" />
                  </div>
                </div>
              </div>
            </section>

            <div class="is-divider" />
          </div>
        </div>
      </div>
    </section>

 

    <!-- Articles -->

    <section class="hero ">
      <div class="hero-body">
        <div class="container">
          <div
            v-for="group in Math.ceil((posts.length - 2) / 2)"
            v-bind:key="group"
          >
            <section class="section">
              <div class="columns is-variable is-8">
                <div
                  v-for="(post, index) in posts.slice(group * 2, group * 2 + 2)"
                  v-bind:key="post.slug"
                  :class="['column is-5', index === 0 && 'is-offset-1']"
                >
                  <div class="content is-medium">
                    <h2 class="subtitle is-5 has-text-grey">
                      {{ formatDate(post.publicationDate) }}
                    </h2>
                    <h1 class="title has-text-black is-3">
                      <nuxt-link :to="`/posts/${post.slug}`">{{
                        post.title
                      }}</nuxt-link>
                    </h1>
                    <div class="has-text-dark" v-html="post.excerpt" />
                  </div>
                </div>
              </div>
            </section>

            <div class="is-divider" />
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { request, gql, imageFields, seoMetaTagsFields } from '~/lib/datocms'
import { toHead } from 'vue-datocms'
import format from 'date-fns/format'
import parseISO from 'date-fns/parseISO'

export default {
  async asyncData({ params }) {
    const data = await request({
      query: gql`
        {
          site: _site {
            favicon: faviconMetaTags {
              ...seoMetaTagsFields
            }
          }

          posts: allPosts(first: 10, orderBy: _firstPublishedAt_DESC) {
            id
            title
            slug
            publicationDate: _firstPublishedAt
            excerpt
            coverImage {
              responsiveImage(imgixParams: { fit: crop, ar: "16:9", w: 860 }) {
                ...imageFields
              }
            }
            author {
              name
              picture {
                responsiveImage(imgixParams: { fit: crop, ar: "1:1", w: 40 }) {
                  ...imageFields
                }
              }
            }
          }
        }

        ${imageFields}
        ${seoMetaTagsFields}
      `
    })

    return { ready: !!data, ...data }
  },
  methods: {
    formatDate(date) {
      return format(parseISO(date), 'PPP')
    }
  },
  head() {
    if (!this.ready) {
      return
    }

    return toHead(this.site.favicon)
  }
}
</script>
