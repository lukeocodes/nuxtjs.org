<template>
  <div class="shadow-nuxt">
    <div class="container mx-auto px-4 lg:flex pb-12">
      <TheMobileAsideNavNewDocs :links="links" />
      <TheAsideNavNewDocs :links="links" class="hidden lg:block" />
      <div
        class="min-h-screen w-full lg:static lg:max-h-full lg:overflow-visible lg:w-3/4"
      >
        <div class="-mx-4 lg:mx-0 flex flex-col-reverse lg:flex-row">
          <div
            class="w-full py-8 px-4 lg:static lg:overflow-visible lg:max-h-full lg:w-3/4"
          >
            <article>
              <h1
                class="text-light-onSurfacePrimary dark:text-dark-onSurfacePrimary transition-colors duration-300 ease-linear"
              >
                {{ page.title }}
              </h1>
              <AppResponsiveVideo v-if="page.youtube" :src="page.youtube" />
              <nuxt-content :document="page" />
            </article>
          </div>
          <AffixBlock>
            <SponsorsBlock />
            <AdsBlock key="release-notes" />
          </AffixBlock>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import groupBy from 'lodash.groupby'

export default {
  async asyncData({ $content, params, store, error, app }) {
    const path = `/en/guides/release-notes`
    let page
    let pages = []

    try {
      const locale = ['pt', 'es'].includes(app.i18n.locale)
        ? app.i18n.locale
        : app.i18n.defaultLocale

      pages = await $content(locale, 'guides', { deep: true })
        .only(['slug', 'title', 'menu', 'category', 'position'])
        .sortBy('position')
        .sortBy('title')
        .sortBy('menu')
        .where({ position: { $gte: 0 } })
        .fetch()
    } catch (e) {}

    try {
      page = await $content(path).fetch()
    } catch (err) {
      return error({
        statusCode: 404,
        message: app.i18n.t('common.page_not_found')
      })
    }

    return {
      path,
      page,
      links: groupBy(pages, 'category')
    }
  },
  computed: {
    docLink() {
      return `https://github.com/nuxt/nuxtjs.org/blob/master/content${this.path}.md`
    },
    codeSandBox() {
      return 'https://codesandbox.io'
    },
    codeSandBoxLink() {
      if (!this.page.github) {
        return ''
      }
      return `${this.codeSandBox}/embed/github/nuxt/nuxt.js/tree/dev/examples/${this.page.github}?autoresize=1&view=editor`
    },
    liveEditLink() {
      return `${this.codeSandBox}/s/github/nuxt/nuxt.js/tree/dev/examples/${this.page.github}?from-embed`
    },
    downloadLink() {
      return (
        'https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/nuxt/nuxt.js/tree/dev/examples/' +
        this.page.github
      )
    }
  },
  scrollToTop: true,
  head() {
    return {
      title: this.page.title,
      titleTemplate: '%s - NuxtJS',
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.page.description
        },
        // Open Graph
        { hid: 'og:title', property: 'og:title', content: this.page.title },
        {
          hid: 'og:description',
          property: 'og:description',
          content: this.page.description
        },
        // Twitter Card
        {
          hid: 'twitter:title',
          name: 'twitter:title',
          content: this.page.title
        },
        {
          hid: 'twitter:description',
          name: 'twitter:description',
          content: this.page.description
        }
      ]
    }
  }
}
</script>
