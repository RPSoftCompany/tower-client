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
  <v-card
    flat
    class="py-3 px-3"
  >
    <v-expansion-panels
      v-model="panel"
      accordion
    >
      <draggable
        style="width: 100%"
        :list="items"
        :options="{ handle: '.handler' }"
        @end="dragEnded"
      >
        <v-expansion-panel
          v-for="(item, i) of items"
          :key="item.id"
        >
          <v-expansion-panel-header>
            <template v-slot:default>
              <v-row no-gutters>
                <v-col cols="12">
                  <v-icon class="handler mr-3">
                    {{ icons.mdiDotsVertical }}
                  </v-icon>
                  {{ item.url }}
                </v-col>
              </v-row>
            </template>
          </v-expansion-panel-header>
          <v-expansion-panel-content>
            <v-divider />
            <v-form
              ref="existingRowForm"
              v-model="newRowValid"
            >
              <v-row align="center">
                <v-col
                  class="d-flex"
                  cols="6"
                >
                  <v-text-field
                    v-model="item.url"
                    :rules="[rules.required, rules.validateUrl]"
                    label="URL"
                    autocomplete="off"
                    @blur="saveConfig(i)"
                  />
                </v-col>
                <v-col
                  class="d-flex"
                  cols="3"
                  offset="3"
                >
                  <v-select
                    v-model="item.returnType"
                    :items="types"
                    label="Type"
                    @blur="saveConfig(i)"
                  />
                </v-col>
                <v-col
                  class="d-flex"
                  cols="12"
                >
                  <div
                    :ref="`divTextArea_${i}`"
                    class="divTextArea"
                    v-html="changeToHtml(item.template)"
                  />
                  <v-textarea
                    :id="`RestTextArea_${i}`"
                    :ref="`RestTextArea_${i}`"
                    v-model="item.template"
                    solo
                    label="Template"
                    auto-grow
                    class="newTextArea"
                    @blur="saveConfig(i)"
                    @keydown="keyDown"
                  />
                </v-col>
              </v-row>
            </v-form>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </draggable>
    </v-expansion-panels>
    <v-divider class="mt-5" />
    <div class="mx-5">
      <v-form
        ref="newRowForm"
        v-model="newRowValid"
      >
        <v-row align="center">
          <v-col
            class="d-flex"
            cols="6"
          >
            <v-text-field
              v-model="newItem.url"
              :rules="[rules.validateUrl]"
              label="New URL"
            />
          </v-col>
          <v-col
            class="d-flex"
            cols="3"
            offset="3"
          >
            <v-select
              v-model="newItem.returnType"
              :items="types"
              label="New type"
            />
          </v-col>
          <v-col
            class="d-flex"
            cols="12"
          >
            <v-textarea
              id="RestTextArea_-1"
              v-model="newItem.template"
              solo
              label="New template"
              :rules="[rules.required]"
              auto-grow
              @keydown="keyDown"
            />
          </v-col>
        </v-row>
        <v-row
          align="center"
          justify="end"
        >
          <v-col
            md="auto"
            class="d-flex"
            cols="12"
          >
            <v-btn
              class="primary"
              :disabled="addDisabled"
              @click="addConfiguration"
            >
              Add Configuration
            </v-btn>
          </v-col>
        </v-row>
      </v-form>
    </div>
  </v-card>
</template>

<script>
  import draggable from 'vuedraggable'
  import { mdiDotsVertical } from '@mdi/js'

  export default {
    name: 'RestSettings',
    components: {
      draggable,
    },
    data: props => ({
      panel: null,
      items: [],
      types: ['json', 'xml', 'plain text'],

      bases: [],

      icons: {
        mdiDotsVertical,
      },

      newItem: {
        url: null,
        returnType: 'json',
        template: '',
      },
      rules: {
        validateUrl: props.validateUrl,
        required: value => !!value || 'Required.',
      },

      newRowValid: true,
    }),
    computed: {
      addDisabled () {
        if (this.newItem.url === null || this.newItem.url === '') {
          return true
        }

        if (this.newItem.template === '' || this.newItem.template === null) {
          return true
        }

        return false
      },
    },
    watch: {
      panel (actual) {
        for (let i = 0; i < this.items.length; i++) {
          const id = `divTextArea_${i}`
          console.log(id)
          if (this.$refs[id] !== undefined) {
            if (this.$refs[id][0].className.includes('showDivTextArea')) {
              this.$refs[id][0].className = this.$refs[id][0].className.replace(' showDivTextArea', '')
            }
          }
        }

        if (actual !== undefined) {
          this.$refs[`divTextArea_${actual}`][0].className += ' showDivTextArea'
          const component = this.$refs[`RestTextArea_${actual}`][0]
          component.$el.className += ' showDivTextArea'
          const inputControl = component.$el.children[0]
          const inputSlot = inputControl.children[0]
          if (!inputSlot.style.cssText.includes('background-color')) {
            inputSlot.style.cssText = 'background-color:rgba(0,0,0,0)'
          }
          const textFieldSlot = inputSlot.children[0]
          const textArea = textFieldSlot.children[0]

          if (!textArea.style.cssText.includes('background-color')) {
            textArea.style.cssText += ';color: rgba(0,0,0,0)'
          }
        }
      },
    },
    async mounted () {
      this.resetData()
    },
    methods: {
      async resetData () {
        const response = await this.axios.get(
          `${this.$store.state.mainUrl}/restConfigurations?filter={"order":"sequenceNumber ASC"}`
        )

        this.items = response.data

        const responseBase = await this.axios.get(
          `${this.$store.state.mainUrl}/baseConfigurations?filter={"order":"sequenceNumber ASC"}`
        )

        this.bases = responseBase.data
      },
      validateUrl (value) {
        if (value === '' || value === null) {
          return true
        }

        const exists = this.bases.some(base => {
          return value.includes(`{${base.name}}`)
        })

        const firstSplit = value.split('{')
        firstSplit.shift()
        const every = firstSplit.every(split => {
          const baseName = split.substring(0, split.indexOf('}'))
          return this.bases.some(base => {
            return base.name === baseName
          })
        })

        if (exists) {
          if (every) {
            return true
          } else {
            return 'Invalid base model'
          }
        } else {
          return 'You need to use at least one base model in URL'
        }
      },
      async addConfiguration () {
        if (!this.$refs.newRowForm.validate()) {
          return
        }

        const newConfig = {
          url: this.newItem.url,
          template: this.newItem.template,
          returnType: this.newItem.returnType,
        }

        await this.axios.post(
          `${this.$store.state.mainUrl}/restConfigurations`,
          newConfig
        )

        this.newItem.url = null
        this.newItem.template = null
        this.newItem.returnType = 'json'

        this.resetData()
      },
      async saveConfig (i) {
        const find = this.$refs.existingRowForm.find(el => {
          return el.$parent.isActive === true
        })

        if (!find.validate()) {
          return
        }

        await this.axios.put(
          `${this.$store.state.mainUrl}/restConfigurations`,
          this.items[i]
        )
      },
      async dragEnded (item) {
        const changedItem = this.items[item.newIndex]
        if (changedItem.sequenceNumber === item.newIndex) {
          return
        }

        changedItem.sequenceNumber = item.newIndex

        await this.axios.post(
          `${this.$store.state.mainUrl}/restConfigurations/changeSequence`,
          changedItem
        )
      },
      keyDown (event) {
        const keyCode = event.keyCode || event.which

        if (keyCode === 9) {
          event.preventDefault()

          const start = event.target.selectionStart

          const item = /-*[0-9]+$/.exec(event.target.id)[0]

          if (item === '-1') {
            const text = this.newItem.template
            const newText = text.substring(0, start) + '\t' + text.substring(start, text.length)

            event.target.value = newText
            event.target.selectionStart = start + 1
            event.target.selectionEnd = start + 1
            this.newItem.template = newText
          } else {
            const text = this.items[item].template
            const newText = text.substring(0, start) + '\t' + text.substring(start, text.length)

            event.target.value = newText
            event.target.selectionStart = start + 1
            event.target.selectionEnd = start + 1

            this.items[item].template = newText
          }
        }
      },
      changeToHtml (text) {
        text = text.replace(/</g, '&lt;')
        text = text.replace(/>/g, '&gt;')

        text = text.replace(/\r\n/g, '\n')

        let splitText = text.split(/\s+/)
        const splitSpaces = text.split(/\S+/)

        let forEachPrev = false
        let ifPrev = 0

        splitText = splitText.map(el => {
          if (forEachPrev) {
            forEachPrev = false
            if (el !== 'END') {
              el = `<span class="textAfterForEach">${el}</span>`
            }
          }

          if (/^[~<>=]+$/.test(el)) {
            el = `<span class="textCompareSign">${el}</span>`
          }

          if (el === '%%ELSE%%') {
            el = `<span class="textElse">${el}</span>`
          }

          if (ifPrev > 0) {
            ifPrev++

            if (ifPrev === 4) {
              ifPrev = 0
            }
          }

          if (el.includes('forEach')) {
            el = el.replace(/forEach/g, '<span class="textForEach">forEach</span>')
            forEachPrev = true
          }

          if (el.includes('%%if')) {
            el = el.replace(/if/g, '<span class="textForEach">if</span>')
            ifPrev = 1
          }

          if (/variables\[['"`]+\S+['"`]+\]/.test(el)) {
            const withoutVariables = /\[['"`]+\S+['"`]+\]/.exec(el)[0]
            el = el.replace(/\[['"`]+\S+['"`]+\]/, `<span class="textRegExp">${withoutVariables}</span>`)
            el = el.replace(/^variables/, '<span class="textVariables">variables</span>')
          }

          if (el.includes('%%')) {
            el = el.replace(/%%/g, '<span class="textPercent">%%</span>')
          }

          return el
        })

        let newText = ''

        for (let i = 0; i < splitSpaces.length; i++) {
          newText += splitSpaces[i]
          if (splitText[i] !== undefined) {
            newText += splitText[i]
          }
        }

        return newText
      },
    },

  }
</script>

<style lang="scss">
.textPercent {
  color: crimson;
}
.textForEach {
  color: blue;
}
.textAfterForEach {
  color: green;
}
.textVariables {
  color: blueviolet;
}
.textRegExp {
  color: brown;
}
.textCompareSign {
  color: darkorange;
}
.textElse {
  color: maroon;
}
</style>

<style lang="scss" scoped>

.handler {
  cursor: grab;
}

.newTextArea{
  color: rgba(0,0,0,0) !important;
  background-color: rgba(0,0,0,0) !important;
}

.divTextArea {
  opacity: 0;
  position: absolute;
  width: 100%;
  height: 100%;
  font-family: "Roboto", sans-serif;
  font-size: 16px;
  font-stretch: 100%;
  line-height: 18px;
  padding-left: 12px;
  padding-top: 7px;
  margin-top: 12px;

  font-variant-ligatures: inherit;
  font-variant-caps: inherit;
  font-variant-numeric: inherit;
  font-variant-east-asian: inherit;
  font-weight: inherit;

  white-space: pre-wrap;
  overflow-wrap: break-word;
}

.showDivTextArea {
  opacity: 1 !important;
}
</style>
