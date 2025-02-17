<script setup lang="ts">
import { ref, shallowRef, onMounted, useRoute } from '#imports'

const INITIAL_CODE = `---
title: MDC
---

# {{ $doc.title}}

MDC stands for _**M**ark**D**own **C**omponents_.

This syntax supercharges regular Markdown to write documents interacting deeply with any Vue component from your \`components/content/\` directory or provided by a module.

## Next steps

- [Install Nuxt Content](/get-started)
- [Explore the MDC syntax](/guide/writing/mdc)

::code-group
  \`\`\`mdc [Source]
  ::alert{type="success"}
    Hooray!
  ::
  \`\`\`

  ::code-block{label="Preview"}
    ::alert{type="success"}
      Hooray!
    ::
  ::
::
`

const route = useRoute()

const content = ref(route.query.content || INITIAL_CODE)

const tab = ref(0)

const tabs = ref([{ label: 'Preview' }, { label: 'AST' }])

const astEditorComponent = shallowRef()

const updateTab = async (index: number) => {
  tab.value = index
  if (tab.value === 1 && !astEditorComponent.value) {
    const { default: component } = await import('~/editor/Editor.vue')

    astEditorComponent.value = component
  }
}

const editorComponent = shallowRef()

const mdcOptions = shallowRef({})
onMounted(async () => {
  const { default: component } = await import('~/editor/Editor.vue')

  editorComponent.value = component

  // @ts-ignore
  const { useShikiHighlighter } = await import('@nuxtjs/mdc/runtime')
  const shikiHighlighter = useShikiHighlighter({})
  mdcOptions.value = {
    highlight: {
      highlighter: (code: string, lang: string, theme: any, highlights: any) => {
        return shikiHighlighter.getHighlightedAST(code, lang, theme, { highlights })
      },
      theme: {
        light: 'material-theme-lighter',
        default: 'material-theme',
        dark: 'material-theme-palenight'
      }
    }
  }
})
</script>

<template>
  <div class="playground">
    <div class="tab-container">
      <TabsHeader :tabs="[{ label: 'Editor' }]" :active-tab-index="0" />
      <TabsHeader :tabs="tabs" :active-tab-index="tab" @update:active-tab-index="updateTab" />
    </div>
    <div class="playground-main">
      <div ref="editor">
        <component :is="editorComponent" v-if="editorComponent" v-model="content" />
        <div v-else class="loading">
          <Alert type="primary">
            <span>Editor is loading...</span>
            <Icon name="file-icons:sandbox" class="ml-2 inline" />
          </Alert>
        </div>
      </div>
      <MDC v-slot="{ data, body }" :value="content" :parser-options="mdcOptions">
        <div v-if="body?.children?.length == 0" class="p-8">
          <Alert type="warning">
            <p class="font-semibold">
              Content is empty!
            </p>
            <br><br>
            <p>
              Type any
              <span class="font-semibold">Markdown</span> or
              <span class="font-semibold">MDC code</span>
              in editor to see it replaced by rendered nodes in this panel.
            </p>
          </Alert>
        </div>
        <MDCRenderer v-if="tab === 0" :key="data?.updatedAt" class="content" :body="body" :data="data" />
        <component
          :is="astEditorComponent"
          v-else-if="astEditorComponent"
          language="json"
          read-only
          :model-value="JSON.stringify(body, null, 2)"
        />
      </MDC>
    </div>
  </div>
</template>

<style scoped lang="ts">
css({
  '.flex': { display: 'flex' },
  '.flex-1': { flex: 1 },
  '.p-8': { padding: '{space.8}' },
  '.overflow-hidden': { overflow: 'hidden' },
  '.relative': { position: 'relative' },
  '.playground': {
    height: 'calc(100vh - 114px)',
    maxHeight: '100%',
    display: 'flex',
    flexDirection: 'column'
  },
  '.tab-container': {
    display: 'flex',
    alignItems: 'center',
    '> div': {
      flex: 1
    }
  },
  '.playground-main': {
    display: 'flex',
    overflow: 'hidden',
    flex: 1,
  },
  '.playground-main > div': {
    flex: 1,
    overflowY: 'auto',
    position: 'relative',
  },
  '.loading': {
    position: 'absolute',
    left: 0,
    top: 0,
    height: '100%',
    width: '100%',
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center'
  },
  '.content': {
    padding: '{space.8}',
  }
})
</style>
