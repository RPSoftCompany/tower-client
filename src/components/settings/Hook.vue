//    Copyright RPSoft 2019,2020. All Rights Reserved.
//    This file is part of RPSoft Tower.
//
//    Tower is free software: you can redistribute it and/or modify
//    it under the terms of the GNU General Public License as published by
//    the Free Software Foundation; either version 3 of the License, or
//    (at your option) any later version.
//
//    Tower is distributed in the hope that it will be useful,
//    but WITHOUT ANY WARRANTY; without even the implied warranty of
//    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
//    GNU General Public License for more details.
//
//    You should have received a copy of the GNU General Public License
//    along with Tower.  If not, see <http://www.gnu.org/licenses/>.

<template>
  <v-card>
    <v-expansion-panels
      accordion
      class="pa-3"
      popout
    >
      <v-expansion-panel
        v-for="model of models"
        :key="model"
      >
        <v-expansion-panel-header>{{ model }}</v-expansion-panel-header>
        <v-expansion-panel-content>
          <v-expansion-panels>
            <v-expansion-panel
              v-for="entry of children.get(model)"
              :key="entry.id"
            >
              <v-expansion-panel-header>{{ entry.method }}</v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-divider />
                <v-text-field
                  v-for="(hook, i) of entry.hooks"
                  :key="hook.id"
                  v-model="hooks[`${model}.${entry.method}.${i}`].url"
                  label="URL"
                  :append-outer-icon="icons.mdiDelete"
                  :disabled="newHookDisabled"
                  @blur="modifyHook(entry, `${model}.${entry.method}.${i}`)"
                  @click:append-outer="
                    removeHook(entry, `${model}.${entry.method}.${i}`)
                  "
                />
                <v-text-field
                  v-model="newHookUrl"
                  label="New URL"
                  :append-outer-icon="icons.mdiPlus"
                  @click:append-outer="addHook(entry)"
                />
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>
  </v-card>
</template>

<script>
  import { mdiDelete, mdiPlus } from '@mdi/js'

  export default {
    name: 'HooksSettings',
    data: () => ({
      models: [],
      children: new Map(),

      icons: {
        mdiDelete,
        mdiPlus,
      },

      hooks: {},
      newHookUrl: null,
      newHookDisabled: false,
    }),
    async mounted () {
      await this.resetData()
    },
    methods: {
      async resetData () {
        const response = await this.axios.get(
          `${this.$store.state.mainUrl}/hooks?filter={"order":"model ASC"}`
        )

        const modelSet = new Set()
        this.hooks = {}
        this.children = new Map()

        response.data.forEach(entry => {
          modelSet.add(entry.model)

          if (this.children.has(entry.model)) {
            const array = this.children.get(entry.model)
            array.push(entry)
          } else {
            this.children.set(entry.model, [entry])
          }

          entry.hooks.forEach((hook, i) => {
            this.hooks[`${entry.model}.${entry.method}.${i}`] = hook
          })
        })

        this.models = Array.from(modelSet)
      },
      async addHook ({ id }) {
        if (this.newHookUrl === null || this.newHookUrl === '') {
          return
        }

        this.newHookDisabled = true

        await this.axios.post(
          `${this.$store.state.mainUrl}/hooks/${id}/hookObject`,
          {
            url: this.newHookUrl,
            description: '',
          }
        )

        await this.resetData()

        this.newHookUrl = null
        this.newHookDisabled = false
      },
      async removeHook ({ id }, hookName) {
        const hook = this.hooks[hookName]

        await this.axios.delete(
          `${this.$store.state.mainUrl}/hooks/${id}/hookObject/${hook._id}`
        )

        await this.resetData()
      },
      async modifyHook ({ id }, hookName) {
        const hook = this.hooks[hookName]

        await this.axios.put(
          `${this.$store.state.mainUrl}/hooks/${id}/hookObject`,
          hook
        )

        await this.resetData()
      },
    },
  }
</script>
