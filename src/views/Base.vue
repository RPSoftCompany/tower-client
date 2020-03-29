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
  <div>
    <div class="d-flex justify-center">
      <div class="halfWidth">
        <v-autocomplete
          ref="modelSelect"
          v-model="currentModel"
          :search-input.sync="modelText"
          :label="modelLabel"
          autocomplete="off"
          item-text="name"
          return-object
          :prepend-icon="baseIcon"
          :items="models"
          clearable
          :append-outer-icon="icons.mdiPlus"
          @change="modelChanged"
          @click:append-outer="addModel"
        >
          <template slot="no-data">
            <div
              class="pa-3"
              v-html="noDataText"
            />
          </template>
        </v-autocomplete>
      </div>
    </div>
    <transition
      name="slowfade"
      mode="out-in"
    >
      <div
        v-if="currentModel !== null && currentModel !== undefined"
        class="mt-4"
      >
        <v-divider />
        <v-tabs
          v-model="currentTab"
          class="elevation-1"
        >
          <v-tab>Rules</v-tab>
          <v-tab>Constant Variables</v-tab>
          <v-tab>Restrictions</v-tab>
        </v-tabs>
        <v-tabs-items
          v-model="currentTab"
          class="elevation-1 pt-5"
        >
          <v-tab-item>
            <v-card flat>
              <v-text-field
                v-model="rules.filter"
                :label="
                  rules.caseSensitive
                    ? 'Filter (case sensitive)'
                    : 'Filter (case insensitive)'
                "
                outlined
                :append-icon="
                  rules.caseSensitive
                    ? icons.mdiFormatLetterCaseLower
                    : icons.mdiFormatLetterCase
                "
                clearable
                dense
                class="px-4"
                @click:append="rules.caseSensitive = !rules.caseSensitive"
              />
              <transition-group
                name="fade-transition"
                duration="100"
                mode="out-in"
                tag="form"
              >
                <v-rule
                  v-for="rule of rulesList"
                  :key="rule._id"
                  :rule_name="rule.name"
                  :new_rule="false"
                  :rule_regex="rule.value"
                  :rule_error="rule.error"
                  :rule_id="rule._id"
                  @delete_rule="deleteRule"
                  @modify_rule="modifyRule"
                />
              </transition-group>
              <v-rule @add_rule="addRule" />
            </v-card>
          </v-tab-item>
          <v-tab-item>
            <v-card flat>
              <v-text-field
                v-model="variables.filter"
                :label="
                  variables.caseSensitive
                    ? 'Filter (case sensitive)'
                    : 'Filter (case insensitive)'
                "
                outlined
                :append-icon="
                  variables.caseSensitive
                    ? icons.mdiFormatLetterCaseLower
                    : icons.mdiFormatLetterCase
                "
                clearable
                dense
                class="px-4"
                @click:append="
                  variables.caseSensitive = !variables.caseSensitive
                "
              />
              <transition-group
                name="fade-transition"
                duration="100"
                mode="out-in"
                tag="form"
              >
                <v-variable
                  v-for="variable of variablesList"
                  :id="variable._id"
                  :key="variable._id"
                  :name="variable.name"
                  :is-new="false"
                  :value="variable.value"
                  @remove_variable="removeVariable"
                  @modify_variable="modifyVariable"
                />
              </transition-group>
              <v-variable @add_variable="addVariable" />
            </v-card>
          </v-tab-item>
          <v-tab-item>
            <v-card flat>
              <v-checkbox
                v-model="restrictions.hasRestrictions"
                color="primary"
                label="Allow restrictions"
                class="px-4"
                @change="modifyOptions"
              />
              <v-card class="restrictionList ma-4">
                <v-text-field
                  v-if="restrictions.hasRestrictions"
                  v-model="restrictions.filter"
                  :label="
                    restrictions.caseSensitive
                      ? 'Filter (case sensitive)'
                      : 'Filter (case insensitive)'
                  "
                  outlined
                  :append-icon="
                    restrictions.caseSensitive
                      ? icons.mdiFormatLetterCaseLower
                      : icons.mdiFormatLetterCase
                  "
                  clearable
                  dense
                  class="px-4 pt-4"
                  @click:append="
                    restrictions.caseSensitive = !restrictions.caseSensitive
                  "
                />
                <v-data-table
                  v-if="restrictions.hasRestrictions"
                  v-model="restrictions.table.selected"
                  :headers="restrictions.table.headers"
                  :items="restrictions.table.items"
                  :hide-default-footer="true"
                  :single-select="false"
                  :search="restrictions.filter"
                  :custom-filter="restrictionList"
                  show-select
                  class="elevation-1"
                  @item-selected="modifyRestriction"
                  @toggle-select-all="modifyAllRestrictions"
                />
              </v-card>
              <v-divider />
            </v-card>
          </v-tab-item>
        </v-tabs-items>
      </div>
    </transition>
  </div>
</template>

<script>
  import rule from '../components/base/rule'
  import variable from '../components/base/variable'
  import { mdiPlus, mdiFormatLetterCase, mdiFormatLetterCaseLower } from '@mdi/js'

  export default {
    name: 'Base',
    components: {
      'v-rule': rule,
      'v-variable': variable,
    },
    data: () => ({
      base: null,
      models: null,

      currentModel: null,
      modelText: null,

      currentTab: null,

      icons: {
        mdiPlus,
        mdiFormatLetterCase,
        mdiFormatLetterCaseLower,
      },
      rules: {
        items: [],
        filter: '',
        caseSensitive: false,
      },
      variables: {
        items: [],
        filter: '',
        caseSensitive: false,
      },
      restrictions: {
        hasRestrictions: false,
        filter: '',
        caseSensitive: false,
        table: {
          headers: [
            {
              text: 'Restriction',
              align: 'left',
              value: 'name',
            },
          ],
          items: [],
          selected: [],
        },
      },
    }),
    computed: {
      baseIcon () {
        if (this.base === null) {
          return null
        } else {
          return this.base.icon
        }
      },
      noDataText () {
        if (this.base !== null && this.modelText !== null) {
          return `No results matching <strong>${this.modelText}</strong>. Press <kbd>+</kbd> button on the right to create a new ${this.base.name}`
        } else {
          return 'Type name and press <kbd>+</kbd> button to create new model'
        }
      },
      modelLabel () {
        if (this.base === null) {
          return ''
        }
        return `Choose ${this.base.name}`
      },
      variablesList () {
        if (
          this.variables.filter === undefined ||
          this.variables.filter === null ||
          this.variables.filter === ''
        ) {
          return this.variables.items
        }
        return this.variables.items.filter(el => {
          if (this.variables.caseSensitive) {
            return el.name.includes(this.variables.filter)
          } else {
            return el.name
              .toUpperCase()
              .includes(this.variables.filter.toUpperCase())
          }
        })
      },
      rulesList () {
        if (
          this.rules.filter === undefined ||
          this.rules.filter === null ||
          this.rules.filter === ''
        ) {
          return this.rules.items
        }
        return this.rules.items.filter(el => {
          if (this.rules.caseSensitive) {
            return el.name.includes(this.rules.filter)
          } else {
            return el.name
              .toUpperCase()
              .includes(this.rules.filter.toUpperCase())
          }
        })
      },
    },
    mounted () {
      this.setData()
    },
    methods: {
      restrictionList (value, search) {
        if (value === undefined) {
          return false
        } else {
          if (!this.restrictions.caseSensitive) {
            return value
              .toString()
              .toUpperCase()
              .includes(search.toUpperCase())
          } else {
            return value.toString().includes(search)
          }
        }
      },
      async setData () {
        const baseName = this.$route.params.name

        const modelData = await this.axios.get(
          `${this.$store.state.mainUrl}/configurationModels?filter={"where":{"base":"${baseName}"},"order":"name ASC"}`
        )

        this.models = modelData.data

        const base = await this.axios.get(
          `${this.$store.state.mainUrl}/baseConfigurations?filter={"where":{"name":"${baseName}"},"order":"name ASC"}`
        )
        this.base = base.data[0]
      },
      async modelChanged (data) {
        if (data !== undefined) {
          const modelData = await this.axios.get(
            `${this.$store.state.mainUrl}/configurationModels?filter={"where":{"id":"${data.id}"}}`
          )

          this.currentModel = modelData.data[0]

          this.rules.items = this.currentModel.rules
          this.rules.items = this.rules.items.sort((a, b) => {
            return a.name.localeCompare(b.name)
          })

          this.variables.items = this.currentModel.defaultValues
          this.variables.items = this.variables.items.sort((a, b) => {
            return a.name.localeCompare(b.name)
          })

          this.restrictions.hasRestrictions = this.currentModel.options.hasRestrictions

          const childBase = await this.axios.get(
            `${
              this.$store.state.mainUrl
            }/baseConfigurations?filter={"where":{"sequenceNumber":"${this.base
              .sequenceNumber + 1}"}}`
          )

          if (childBase.data.length > 0) {
            const childBaseName = childBase.data[0].name

            const children = await this.axios.get(
              `${this.$store.state.mainUrl}/configurationModels?filter={"where":{"base":"${childBaseName}"}}`
            )

            if (children !== undefined) {
              this.restrictions.table.items = children.data
              this.restrictions.table.selected = []
              this.restrictions.table.items.forEach(el => {
                if (this.currentModel.restrictions.includes(el.name)) {
                  this.restrictions.table.selected.push(el)
                }
              })
            }
          }
        } else {
          this.currentModel = null
          this.rules.items = []
          this.rules.filter = null
          this.rules.caseSensitive = false

          this.variables.items = []
          this.variables.filter = null
          this.variables.caseSensitive = false

          this.restrictions.hasRestrictions = false
          this.restrictions.table.selected = []
        }
      },
      async addModel () {
        if (this.modelText === '' || this.modelText === null) {
          return
        }

        if (this.models.includes(this.modelText)) {
          return
        }

        const newModel = await this.axios.post(
          `${this.$store.state.mainUrl}/configurationModels`,
          {
            name: this.modelText,
            rules: [],
            restrictions: [],
            defaultValues: [],
            base: this.base.name,
            options: {
              hasRestrictions: false,
            },
          }
        )

        this.models.push(newModel.data)
        this.currentModel = newModel.data

        this.modelChanged(this.currentModel)

        this.$refs.modelSelect.isMenuActive = false
        this.$refs.modelSelect.blur()
      },
      async addRule (data) {
        const rule = await this.axios.post(
          `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/rule`,
          {
            name: data.name,
            value: data.regex,
            error: data.errorMessage,
          }
        )

        if (rule !== undefined) {
          this.rules.items.push(rule.data)
          this.rules.items = this.rules.items.sort((a, b) => {
            return a.name.localeCompare(b.name)
          })
          data._this.reset()
        }
      },
      async deleteRule (data) {
        const id = data.id
        const rule = await this.axios.delete(
          `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/rule?ruleId=${id}`
        )

        if (rule !== undefined) {
          this.rules.items = this.rules.items.filter(el => {
            return el._id !== id
          })
        }
      },
      async modifyRule (data) {
        await this.axios.patch(
          `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/rule`,
          {
            _id: data.id,
            name: data.name,
            value: data.regex,
            error: data.errorMessage,
          }
        )
      },
      async addVariable (data) {
        const variable = await this.axios.post(
          `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/variable`,
          {
            name: data.name,
            value: data.value,
          }
        )

        if (variable !== undefined) {
          this.variables.items.push(variable.data)
          this.variables.items = this.variables.items.sort((a, b) => {
            return a.name.localeCompare(b.name)
          })
          data._this.reset()
        }
      },
      async removeVariable (data) {
        const id = data.id
        const variable = await this.axios.delete(
          `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/variable?variableId=${id}`
        )

        if (variable !== undefined) {
          this.variables.items = this.variables.items.filter(el => {
            return el._id !== id
          })
        }
      },
      async modifyVariable (data) {
        await this.axios.patch(
          `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/variable`,
          {
            _id: data.id,
            name: data.name,
            value: data.value,
          }
        )
      },
      async modifyOptions (restrictions) {
        await this.axios.patch(
          `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/options`,
          {
            hasRestrictions: restrictions,
          }
        )
      },
      async modifyRestriction (data) {
        if (data.value === true) {
          await this.axios.post(
            `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/restriction?restriction=${data.item.name}`
          )
        } else {
          await this.axios.delete(
            `${this.$store.state.mainUrl}/configurationModels/${this.currentModel.id}/restriction?restriction=${data.item.name}`
          )
        }
      },
      async modifyAllRestrictions (data) {
        for (const rest of this.restrictions.table.items) {
          const newData = {
            value: data.value,
            item: rest,
          }
          await this.modifyRestriction(newData)
        }
      },
    },
  }
</script>

<style scoped>
.halfWidth {
	width: 50%;
}
</style>