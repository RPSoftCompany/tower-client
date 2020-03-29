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
    class="mt-4"
  >
    <div class="d-flex justify-center">
      <div class="halfWidth">
        <v-autocomplete
          ref="groupSelect"
          v-model="group.current"
          class="pa-2"
          :search-input.sync="group.text"
          label="Group"
          :items="group.items"
          item-text="name"
          clearable
          autocomplete="off"
          :append-outer-icon="icons.mdiPlus"
          return-object
          @click:append-outer="addGroup"
          @change="onGroupChange"
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
    <div
      v-if="group.current !== null && group.current !== undefined"
      class="pa-2"
    >
      <v-autocomplete
        v-model="role.current"
        :items="role.items"
        chips
        return-object
        multiple
        item-text="name"
        label="Group Roles"
        class="mx-3"
        @change="onRoleChange"
      />
    </div>
  </v-card>
</template>

<script>
  import { mdiPlus } from '@mdi/js'

  export default {
    name: 'GroupsSettings',
    data: () => ({
      icons: {
        mdiPlus,
      },
      group: {
        current: null,
        items: [],
        text: null,
      },
      role: {
        current: null,
        items: [],
      },
    }),
    computed: {
      noDataText () {
        if (this.group.text !== null) {
          return `No results matching <strong>${this.group.text}</strong>. Press <kbd>+</kbd> button on the right to create group with this name`
        } else {
          return 'Type name and press <kbd>+</kbd> button to create new group'
        }
      },
    },
    async mounted () {
      await this.resetData()
    },
    methods: {
      async resetData () {
        this.group.current = null
        const response = await this.axios.get(
          `${this.$store.state.mainUrl}/Groups?filter={"order":"name ASC"}`
        )

        this.group.items = response.data

        const roleResponse = await this.axios.get(
          `${this.$store.state.mainUrl}/Roles?filter={"order":"name ASC"}`
        )

        this.role.items = roleResponse.data
      },
      async addGroup () {
        if (this.group.text === null || this.group.text === '') {
          return
        }
        const response = await this.axios.post(
          `${this.$store.state.mainUrl}/Groups`,
          {
            name: this.group.text,
            roles: [],
          }
        )

        this.$refs.groupSelect.isMenuActive = false
        this.group.items.push(response.data)
        this.group.current = response.data
      },
      onGroupChange () {
        this.role.current = []
        this.role.items.forEach(role => {
          if (this.group.current !== undefined) {
            if (this.group.current.roles.includes(role.name)) {
              this.role.current.push(role)
            }
          }
        })
      },
      async onRoleChange () {
        this.group.current.roles = []
        this.role.current.forEach(role => {
          this.group.current.roles.push(role.name)
        })

        await this.axios.patch(
          `${this.$store.state.mainUrl}/Groups`,
          this.group.current
        )
      },
    },
  }
</script>

<style lang="scss" scoped>
.halfWidth {
	width: 50%;
}
</style>
