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
    <div class="d-flex">
      <v-autocomplete
        v-for="base of baseArray"
        :key="base.name"
        v-model="values[base.sequenceNumber]"
        class="pa-2"
        :prepend-icon="base.icon"
        :label="base.name"
        :items="arrayOfArrays[base.sequenceNumber]"
        item-text="name"
        clearable
        autocomplete="off"
        return-object
        @change="fillNextArray(base.sequenceNumber)"
      />
    </div>
    <div
      class="headline font-weight-light text-center configurationTitle"
    >
      {{ configuration.configInfo }}
    </div>
    <v-progress-linear
      :active="configuration.loading"
      indeterminate
      :height="10"
    />
    <v-card
      v-if="configuration.items.length > 0 || configuration.editMode === true"
      class="pb-2 pt-5"
    >
      <div class="d-flex">
        <v-text-field
          v-model="configuration.filter.filter"
          :label="
            configuration.filter.caseSensitive
              ? 'Filter (case sensitive)'
              : 'Filter (case insensitive)'
          "
          :append-icon="
            configuration.filter.caseSensitive
              ? icons.mdiFormatLetterCaseLower
              : icons.mdiFormatLetterCase
          "
          clearable
          outlined
          dense
          class="px-4"
          style="max-width: 33%"
          @click:append="
            configuration.filter.caseSensitive = !configuration.filter
              .caseSensitive
          "
        />
        <v-spacer />
        <v-tooltip
          v-if="promoted.length > 0"
          bottom
        >
          <template v-slot:activator="{ on }">
            <v-icon
              style="max-height: 36px;"
              class="pr-3"
              @click="showPromotionDialog"
              v-on="on"
            >
              icons.mdiPackageUp
            </v-icon>
          </template>
          <span>Get promoted configuration</span>
        </v-tooltip>
        <v-tooltip
          v-if="configuration.items.length > 0"
          bottom
        >
          <template v-slot:activator="{ on }">
            <v-icon
              :disabled="configuration.editModeDisabled"
              class="mr-3"
              v-on="on"
              @click="changeEditMode"
              v-text="
                configuration.editMode ? icons.mdiSquareEditOutline : icons.mdiPencil
              "
            />
          </template>
          <span>Change mode</span>
        </v-tooltip>
      </div>
      <v-divider class="pb-5" />
      <template v-if="configuration.editMode">
        <div class="d-flex flex-row justify-space-around">
          <div class="subtitle-1 font-weight-bold">
            Name
          </div>
          <div class="subtitle-1 font-weight-bold">
            Type
          </div>
          <div class="subtitle-1 font-weight-bold">
            Value
          </div>
        </div>
      </template>
      <template v-else>
        <div
          v-if="configuration.versions.length > 0"
          class="d-flex flex-row justify-space-around"
        >
          <div class="subtitle-1 thirdWidth text-center font-weight-bold">
            Name
          </div>
          <div class="d-flex flex-row justify-space-between thirdWidth">
            <v-icon
              :disabled="
                configuration.shownVersion === configuration.minVersion
              "
              @click="previousVersion"
            >
              {{ icons.mdiChevronLeft }}
            </v-icon>
            <div
              :key="versionLabel"
              class="subtitle-1 font-weight-bold"
            >
              {{ versionLabel }}
            </div>
            <v-icon
              :disabled="
                configuration.shownVersion === configuration.maxVersion
              "
              @click="nextVersion"
            >
              {{ icons.mdiChevronRight }}
            </v-icon>
          </div>
          <div class="subtitle-1 thirdWidth text-center font-weight-bold">
            Value
          </div>
        </div>
        <div
          v-if="configuration.versions.length == 0"
          class="d-flex flex-row justify-space-around"
        >
          <div class="subtitle-1">
            Name
          </div>
          <div class="subtitle-1">
            Value
          </div>
        </div>
      </template>
      <div class="py-3" />
      <template v-if="configuration.editMode">
        <newConfigurationRow
          v-for="item of editModeItems"
          :key="item.name"
          :name="item.name"
          :value="item.value"
          :type="item.type"
          :added="true"
          :visible="item.visible"
          @change_row="changeConfigurationRow"
          @remove_row="removeNewConfigurationRow"
        />
        <newConfigurationRow
          ref="newConigurationRow"
          @add_row="addNewConfigurationRow"
        />
      </template>
      <template v-else>
        <configurationRow
          v-for="item of configModelItems"
          ref="configRows"
          :key="item.name"
          :name="item.name"
          :value="item.value"
          :type="item.type"
          :versions="item.versions"
          :rules="item.rules"
          :forced_value="item.forced_value"
          :force_cause="item.force_cause"
          :visible="item.visible"
          :current_version="configuration.shownVersion"
          @change="changeConfigurationRow"
        />
      </template>
      <v-divider />
      <div
        v-if="!configuration.editMode"
        class="d-flex flex-row justify-space-around mt-4 mb-2"
      >
        <v-btn
          :disabled="!differentThanPrevVersion"
          color="primary"
          @click="saveConfiguration"
        >
          Save configuration
        </v-btn>
      </div>
    </v-card>
    <v-dialog
      v-model="promotionDialog.show"
      max-width="80%"
    >
      <v-card>
        <v-card-title>
          Choose from promoted configurations
        </v-card-title>
        <v-card-text>
          <v-data-table
            :headers="promotionDialog.headers"
            :items="promotionDialog.configuration"
            @click:row="loadPromoted"
          />
        </v-card-text>
        <v-divider />
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
            text
            @click="promotionDialog.show = false"
          >
            Cancel
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
  import newConfigurationRow from '../components/configuration/newConfigurationRow'
  import configurationRow from '../components/configuration/configurationRow'
  import {
    mdiFormatLetterCaseLower, mdiFormatLetterCase, mdiPackageUp,
    mdiSquareEditOutline, mdiPencil, mdiChevronLeft, mdiChevronRight,
  } from '@mdi/js'

  export default {
    name: 'Configuration',
    components: {
      newConfigurationRow,
      configurationRow,
    },
    data: () => ({
      values: [],
      baseArray: [],
      arrayOfArrays: [],
      defaultValues: {},
      baseRules: [],

      promoted: [],

      promotionDialog: {
        show: false,
        configuration: [],
        headers: [
          {
            text: 'Configuration',
            value: 'text',
          },
        ],
      },

      icons: {
        mdiFormatLetterCaseLower,
        mdiFormatLetterCase,
        mdiPackageUp,
        mdiSquareEditOutline,
        mdiPencil,
        mdiChevronLeft,
        mdiChevronRight,
      },

      configuration: {
        configInfo: 'Find your configuration',
        loading: false,
        new: false,
        editMode: false,
        editModeDisabled: false,
        backupItems: [],
        items: [],
        versions: [],
        versionLabel: null,
        shownVersion: 0,
        minVersion: 0,
        maxVersion: 0,
        filter: {
          caseSensitive: false,
          filter: null,
        },
      },
    }),
    computed: {
      differentThanPrevVersion () {
        let isDifferent = false
        const maxVersion = this.configuration.maxVersion

        this.configuration.items.forEach(variable => {
          if (variable.type === 'boolean') {
            if (variable.value === true || variable.value === 'true') {
              if (variable.versions[maxVersion] !== 'true') {
                isDifferent = true
              }
            } else if (variable.value === false || variable.value === 'false') {
              if (variable.versions[maxVersion] !== 'false') {
                isDifferent = true
              }
            }
          } else {
            if (variable.value !== variable.versions[maxVersion]) {
              isDifferent = true
            }
          }
        })

        return isDifferent
      },
      versionLabel () {
        if (this.configuration.versions.length === 0) {
          return null
        } else {
          let date = Date.parse(this.configuration.versions[this.configuration.shownVersion].effectiveDate)
          date = new Date(date)
          date = date.toLocaleString(undefined, { timeZoneName: 'short' })

          const versionLabel =
            this.configuration.shownVersion !== this.configuration.maxVersion
              ? date
              : `latest (${date})`
          return `Version #${this.configuration.shownVersion} - ${versionLabel}`
        }
      },
      editModeItems () {
        if (!this.configuration.editMode) {
          return []
        } else {
          let filter = this.configuration.filter.filter
          if (filter === null || filter === undefined || filter === '') {
            filter = ''
            this.configuration.items.map(el => {
              el.visible = true
            })
            return this.configuration.items
          }

          if (this.configuration.filter.caseSensitive) {
            filter = filter.toUpperCase()
          }

          this.configuration.items.map(el => {
            const rightName = this.configuration.filter.caseSensitive
              ? el.name.toUpperCase()
              : el.name
            let rightValue = el.value
            if (rightValue !== null) {
              rightValue = this.configuration.filter.caseSensitive
                ? el.value.toUpperCase()
                : el.value
            } else {
              rightValue = ''
            }

            if (el.type === 'password') {
              rightValue = ''
            }

            el.visible =
              rightName.includes(filter) || rightValue.includes(filter)
          })

          return this.configuration.items
        }
      },
      configModelItems () {
        if (this.configuration.editMode) {
          return []
        } else {
          let filter = this.configuration.filter.filter
          if (filter === null || filter === undefined || filter === '') {
            filter = ''
            this.configuration.items.map(el => {
              el.visible = true
            })
            return this.configuration.items
          }

          if (this.configuration.filter.caseSensitive) {
            filter = filter.toUpperCase()
          }

          this.configuration.items.map(el => {
            const rightName = this.configuration.filter.caseSensitive
              ? el.name.toUpperCase()
              : el.name
            let rightValue = el.value
            if (rightValue !== null) {
              rightValue = this.configuration.filter.caseSensitive
                ? el.value.toUpperCase()
                : el.value
            } else {
              rightValue = ''
            }

            if (el.type === 'password') {
              rightValue = ''
            }

            el.visible =
              rightName.includes(filter) || rightValue.includes(filter)
          })

          return this.configuration.items
        }
      },
    },
    async created () {
      const response = await this.axios.get(
        `${this.$store.state.mainUrl}/baseConfigurations?filter={"order":"sequenceNumber ASC"}`
      )

      this.baseArray = response.data
      this.baseArray.forEach(el => {
        this.arrayOfArrays[el.sequenceNumber] = []
      })

      if (this.baseArray.length > 0) {
        await this.getArrayFromBase(this.baseArray[0].name, 0)
        this.$forceUpdate()
      }

      if (this.baseArray.length === 0) {
        this.configuration.configInfo = 'You need to setup your Base Models first'
      }
    },
    methods: {
      async getArrayFromBase (base, sequenceNumber) {
        const array = await this.axios.get(
          `${this.$store.state.mainUrl}/configurationModels?filter={"where":{"base": "${base}"}}`
        )

        let restrictions = []

        if (sequenceNumber > 0) {
          restrictions = this.values[sequenceNumber - 1].restrictions
        }

        array.data.sort((a, b) => {
          return a.name.toUpperCase().localeCompare(b.name.toUpperCase())
        })

        let newArray = array.data

        if (sequenceNumber > 0) {
          if (this.values[sequenceNumber - 1].options.hasRestrictions) {
            newArray = array.data.filter(item => {
              if (restrictions.includes(item.name)) {
                return true
              }
              return false
            })
          }
        }

        this.arrayOfArrays[sequenceNumber] = newArray
      },
      async fillNextArray (sequenceNumber) {
        this.configuration.items = []
        this.configuration.editMode = false
        this.configuration.configInfo = 'Find your configuration'

        if (this.baseArray.length > sequenceNumber + 1) {
          const base = this.baseArray[sequenceNumber + 1].name

          for (let i = sequenceNumber + 1; i < this.arrayOfArrays.length; i++) {
            this.arrayOfArrays[i] = []
            this.values[i] = null
          }

          await this.getArrayFromBase(base, sequenceNumber + 1)

          this.$forceUpdate()
        } else {
          let undef = false

          for (const el of this.values) {
            if (el === undefined) {
              undef = true
            }
          }

          if (!undef) {
            this.getConfiguration()
          }
        }
      },
      async getConfiguration (promotedConfiguration) {
        let filter = ''

        this.values.forEach(value => {
          filter += `"${value.base}":"${value.name}",`
        })

        filter = filter.substring(0, filter.length - 1)

        this.configuration.items = []
        this.configuration.versions = []
        this.promoted = []

        this.configuration.loading = true

        const configuration = await this.axios.get(
          `${this.$store.state.mainUrl}/configurations?filter={"where":{${filter}},"order":"version DESC"}`
        )

        if (promotedConfiguration === undefined) {
          this.getPromoteCandidates()
        } else {
          if (configuration.data.length === 0) {
            this.configuration.backupItems = []
            promotedConfiguration.data[0].variables.forEach(variable => {
              const item = {
                name: variable.name,
                value: variable.value,
                type: variable.type,
                versions: [],
              }

              this.configuration.backupItems.push(item)
            })

            this.slowlyAddItems(0)
          } else {
            configuration.data[0].variables =
              promotedConfiguration.data[0].variables
          }
        }

        if (configuration.data.length === 0) {
          if (promotedConfiguration === undefined) {
            this.configuration.editMode = true
          } else {
            this.configuration.editMode = false
          }
          this.configuration.new = true
          this.configuration.configInfo =
            'Configuration not found, you can create new one'
        } else {
          this.configuration.configInfo = `Create new configuration, version #${configuration
            .data[0].version + 1}`

          this.configuration.shownVersion = configuration.data[0].version
          this.configuration.versions = []

          const currentVariables = configuration.data[0].variables
          configuration.data.forEach(conf => {
            conf.variables.forEach(variable => {
              const index = currentVariables.findIndex(el => {
                return el.name === variable.name
              })

              if (index !== -1) {
                if (currentVariables[index].versions === undefined) {
                  currentVariables[index].versions = []
                }

                currentVariables[index].versions[conf.version] = variable.value
              }
            })

            this.configuration.versions[conf.version] = {
              version: conf.version,
              effectiveDate: conf.effectiveDate,
            }
          })

          this.defaultValues = {}
          this.baseRules = []

          this.values.forEach(base => {
            base.defaultValues.forEach(defaultValue => {
              this.defaultValues[defaultValue.name] = {
                value: defaultValue.value,
                cause: `Value forced by ${base.base}`,
              }
            })

            base.rules.forEach(baseRule => {
              this.baseRules.push(baseRule)
            })

            if (base.rules !== undefined) {
              base.rules.forEach(rule => {
                const index = currentVariables.findIndex(variable => {
                  return variable.name === rule.name
                })

                if (index !== -1) {
                  if (currentVariables[index].rules === undefined) {
                    currentVariables[index].rules = []
                  }
                  currentVariables[index].rules.push(rule)
                }
              })
            }
            if (base.defaultValues !== undefined) {
              base.defaultValues.forEach(value => {
                const index = currentVariables.findIndex(variable => {
                  return variable.name === value.name
                })

                if (index !== -1) {
                  if (currentVariables[index].forced_value === undefined) {
                    currentVariables[index].forced_value = true
                    currentVariables[index].value = value.value
                    currentVariables[
                      index
                    ].force_cause = `Value forced by ${base.base}`
                  }
                }
              })
            }
          })

          if (promotedConfiguration === undefined) {
            this.configuration.minVersion =
              configuration.data[configuration.data.length - 1].version
            this.configuration.maxVersion = configuration.data[0].version

            this.configuration.backupItems = currentVariables
            this.configuration.editModeDisabled = true
          }

          this.slowlyAddItems(0)
        }

        this.configuration.loading = false
      },
      async getPromoteCandidates () {
        const filter = {}
        this.values.forEach(value => {
          filter[value.base] = value.name
        })

        const candidates = await this.axios.post(
          `${this.$store.state.mainUrl}/configurations/promotionCandidates`,
          filter
        )

        this.promoted = candidates.data
      },
      async loadPromoted (configuration) {
        configuration.configuration.versions = this.configuration.versions
        const array = {
          data: [configuration.configuration],
        }
        this.promotionDialog.show = false

        await this.getConfiguration(array)
      },
      showPromotionDialog () {
        this.promotionDialog.configuration = []
        this.promoted.forEach(promoted => {
          let text = ''
          this.values.forEach(value => {
            if (promoted[value.base] !== value.name) {
              text = `${value.base}: ${promoted[value.base]}, version #${
                promoted.version
              }`
            }
          })

          this.promotionDialog.configuration.push({
            text: text,
            configuration: promoted,
          })
        })

        this.promotionDialog.show = true
      },
      addNewConfigurationRow (data) {
        const find = this.configuration.items.find(el => {
          return el.name === data.name
        })

        const versions = new Array(this.configuration.versions.length)

        if (find === undefined) {
          const item = {
            name: data.name,
            value: data.value,
            type: data.type,
            versions: versions,
          }

          const findDefault = this.defaultValues[data.name]
          if (findDefault !== undefined) {
            item.value = findDefault.value
            item.forced_value = true
            item.force_cause = findDefault.cause
          }

          item.rules = this.baseRules.filter(baseRule => {
            return baseRule.name === data.name
          })

          this.configuration.items.push(item)

          data._this.reset()
        } else {
          data._this.errorMessage = 'Value with this name already exists'
        }

        this.configuration.items.sort((a, b) => {
          return a.name.toUpperCase().localeCompare(b.name.toUpperCase())
        })

        this.$refs.newConigurationRow.focus()
      },
      removeNewConfigurationRow (data) {
        this.configuration.items = this.configuration.items.filter(el => {
          return el.name !== data.name
        })
      },
      changeEditMode () {
        this.configuration.editModeDisabled = true
        this.configuration.backupItems = this.configuration.items.slice(0)
        this.slowlyRemoveItems()
      },
      slowlyRemoveItems () {
        if (this.configuration.items.length !== 0) {
          this.configuration.items = []
          this.configuration.editMode = !this.configuration.editMode
          this.slowlyAddItems(0)
        }
      },
      slowlyAddItems (i) {
        setTimeout(() => {
          const modif = 3
          const slice = this.configuration.backupItems.slice(
            i * modif,
            (i + 1) * modif
          )
          if (slice.length === 0) {
            this.configuration.editModeDisabled = false
          } else {
            this.configuration.items = this.configuration.items.concat(slice)
            this.slowlyAddItems(++i)
          }
        }, 5)
      },
      changeConfigurationRow (data) {
        this.configuration.items.map(el => {
          if (el.name === data.name) {
            el.type = data.type
            el.value = data.value
          }
        })
      },
      async saveConfiguration () {
        for (const row of this.$refs.configRows) {
          if (!row.valid()) {
            this.$store.commit(
              'setError',
              'Your configuration is invalid. Please review and correct it.'
            )
            return
          }
        }

        this.configuration.items.map(el => {
          if (el.value === undefined || el.value === null) {
            el.value = ''
          }
        })

        const items = []
        this.configuration.items.forEach(item => {
          items.push({
            name: item.name,
            value: item.value,
            type: item.type,
          })
        })

        const conf = {
          variables: items,
          promoted: false,
          description: '',
        }

        this.values.forEach(value => {
          conf[value.base] = value.name
        })

        await this.axios.post(
          `${this.$store.state.mainUrl}/configurations`,
          conf
        )

        this.getConfiguration()
      },
      previousVersion () {
        if (this.configuration.shownVersion !== this.configuration.minVersion) {
          let minus = 1
          while (
            this.configuration.versions[
              this.configuration.shownVersion - minus
            ] === undefined
          ) {
            minus++
            if (
              this.configuration.shownVersion - minus ===
              this.configuration.minVersion
            ) {
              this.configuration.minVersion = this.configuration.shownVersion
              return
            }
          }

          this.configuration.shownVersion -= minus
        }
      },
      nextVersion () {
        if (this.configuration.shownVersion !== this.configuration.maxVersion) {
          let plus = 1
          while (
            this.configuration.versions[
              this.configuration.shownVersion + plus
            ] === undefined
          ) {
            plus++
            if (
              this.configuration.shownVersion + plus ===
              this.configuration.maxVersion
            ) {
              this.configuration.maxVersion = this.configuration.shownVersion
              return
            }
          }

          this.configuration.shownVersion += plus
        }
      },
    },
  }
</script>

<style scoped>
.configurationTitle {
  height: 50px;
  color: lightgray;
  margin-bottom: 20px;
  margin-top: 20px;
  cursor: default;
}

.thirdWidth {
  max-width: 32%;
  width: 32%;
}
</style>
