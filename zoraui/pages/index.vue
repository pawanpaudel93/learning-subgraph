<template>
  <div>
    <div
      class="
        grid
        sm:grid-cols-2
        md:grid-cols-2
        lg:grid-cols-4
        gap-4
        px-10
        py-10
      "
    >
      <div
        v-for="token in tokens"
        :key="token.id"
        class="shadow-lg bg-transparent rounded-2xl overflow-hidden"
      >
        <div class="w-100% h-100%">
          <div
            v-if="token.type === 'image'"
            :style="{ height: '320px', overflow: 'hidden' }"
          >
            <img :style="{ minHeight: '320px' }" :src="token.contentURI" />
          </div>
          <div v-else-if="token.type === 'video'" class="relative">
            <div
              :style="{
                width: '288px',
                height: '320px',
                boxSizing: 'border-box',
              }"
            />
            <div
              :style="{
                position: 'absolute',
                top: 0,
                left: 0,
                bottom: 0,
                right: 0,
              }"
            >
              <video
                height="auto"
                controls
                :style="{
                  position: 'absolute',
                  width: '100%',
                  height: '100%',
                  display: 'block',
                  objectFit: 'cover',
                }"
              >
                <source :src="token.contentURI" />
              </video>
            </div>
          </div>
          <audio v-else-if="token.type === 'audio'" controls>
            <source :src="token.contentURI" type="audio/ogg" />
            <source :src="token.contentURI" type="audio/mpeg" />
            Your browser does not support the audio element.
          </audio>

          <div class="px-2 pt-2 pb-10">
            <h3 :style="{ height: 100 }" class="text-xl p-4 pt-6 font-semibold">
              {{ token.meta.name }}
            </h3>
          </div>
        </div>
      </div>
    </div>
    <infinite-loading
      spinner="spiral"
      @infinite="infiniteScroll"
    ></infinite-loading>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { gql } from 'graphql-tag'

type Token = {
  id: string
  tokenId: string
  contentURI: string
  metadataURI: string
  type?: string
  meta?: {
    description: string
    mimeType: string
    name: string
    version: string
  }
}

export default Vue.extend({
  name: 'IndexPage',
  layout: 'default',
  data() {
    return {
      skip: 0,
      tokens: [] as Token[],
    }
  },
  async fetch() {
    this.tokens = await this.fetchTokens()
  },

  methods: {
    getQuery() {
      return gql`
      query {
        tokens(
          orderBy: createdAtTimestamp
          orderDirection: desc
          first: 12
          skip: ${this.skip}
        ) {
          id
          tokenID
          contentURI
          metadataURI
        }
      }
    `
    },
    async fetchTokens(): Promise<Token[]> {
      const query = this.getQuery()
      const data = await this.$apollo.query({ query })
      return await Promise.all(
        data.data.tokens.map(async (token: Token) => {
          let meta
          try {
            const metaData = await fetch(
              token.metadataURI.replace('ipfs://', 'https://ipfs.io/ipfs/')
            )
            meta = await metaData.json()
          } catch (err) {}
          if (!meta) return
          if (meta?.body) {
            const body = meta.body
            meta = {
              mimeType: body.mimeType,
              name: body.title,
              description: body.notes,
              version: body.version,
            }
          }
          if (!meta.mimeType) {
            meta.mimeType = 'video/mp4'
          }

          if (meta.mimeType.includes('mp4')) {
            token.type = 'video'
          } else if (meta.mimeType.includes('wav')) {
            token.type = 'audio'
          } else {
            token.type = 'image'
          }
          token.contentURI = token.contentURI.replace(
            'ipfs://',
            'https://ipfs.io/ipfs/'
          )
          token.meta = meta
          return token
        })
      )
    },
    infiniteScroll($state: any) {
      setTimeout(async () => {
        this.skip += 12
        const tokens: Token[] = await this.fetchTokens()
        if (tokens.length > 0) {
          this.tokens = this.tokens.concat(tokens)
          $state.loaded()
        } else {
          $state.complete()
        }
      }, 500)
    },
  },
})
</script>
