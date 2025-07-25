<script lang="ts" setup>
import type { PageData, Post } from 'valaxy'
import { onClickHref, onContentUpdated, scrollTo, tObject, useFrontmatter, useLayout, useSidebar, useSiteConfig } from 'valaxy'
import { computed, nextTick } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute, useRouter } from 'vue-router'
import { targetPadding } from '../client'

defineProps<{
  frontmatter: Post
  data?: PageData
}>()

const siteConfig = useSiteConfig()
const frontmatter = useFrontmatter()

const { hasSidebar } = useSidebar()
const isHome = useLayout('home')
const layout = useLayout()

const { locale } = useI18n()
const $title = computed(() => tObject(frontmatter.value.title || '', locale.value))

const route = useRoute()
const router = useRouter()

nextTick(() => {
  if (route.hash) {
    setTimeout(() => {
      scrollTo(document.body, route.hash, {
        smooth: true,
        targetPadding,
      })
    }, 0)
  }
})

onContentUpdated(() => {
  onClickHref(router)
})
</script>

<template>
  <main
    class="press-main flex" :class="{
      'has-sidebar': hasSidebar && layout !== 'post',
    }"
  >
    <div
      w="full" flex="~" :class="{
        'px-6 md:pl-12': hasSidebar,
        'has-aside': !isHome,
      }" p="t-4"
      class="relative"
    >
      <div class="container" flex="~ grow" justify="between">
        <slot name="main">
          <div class="vp-doc content" w="full" :class="{ 'm-auto': !hasSidebar }" flex="~ col grow" p="lt-md:0">
            <slot name="main-header" />
            <slot name="main-header-after" />

            <slot name="main-content">
              <slot name="main-content-before" />

              <ValaxyMd class="mx-auto w-full max-w-4xl" :frontmatter="frontmatter">
                <h1 v-if="hasSidebar && !isHome && $title" :id="$title" tabindex="-1">
                  {{ $title }}
                  <a class="header-anchor" :href="`#${$title}`" aria-hidden="true" />
                </h1>
                <slot name="main-content-md" />
                <slot />
              </ValaxyMd>
              <PressDocFooter v-if="!isHome" class="pb-8 max-w-4xl" w="full" m="auto" />

              <slot name="main-content-after" />
            </slot>
          </div>

          <slot name="main-nav-before" />

          <slot name="main-nav" />

          <slot name="main-nav-after" />

          <slot v-if="siteConfig.comment.enable && frontmatter.comment !== false" name="comment" />

          <slot name="footer" />
        </slot>

        <slot name="aside">
          <PressAside v-if="!isHome" />
        </slot>
      </div>
    </div>
  </main>
</template>

<style lang="scss" scoped>
@use 'valaxy/client/styles/mixins/index.scss' as *;

@include screen('md') {
  .press-main {
    &.has-sidebar {
      padding-top: var(--pr-nav-height);
      padding-left: var(--va-sidebar-width);
    }
  }
}

@include screen('xl') {
  .content{
    // 8px scrollbar width
    max-width: calc(100vw - 2 * var(--va-sidebar-width) - 2.5rem);

    &.no-aside {
      max-width: calc(100vw - var(--va-sidebar-width));
    }
  }
}
</style>
