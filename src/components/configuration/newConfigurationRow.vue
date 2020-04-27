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
  <transition
    name="fade"
    mode="out-in"
  >
    <v-form
      v-show="visible"
      ref="newConfigForm"
      lazy-validation
      autocomplete="off"
    >
      <div class="d-flex flex-row justify-space-around">
        <v-text-field
          ref="configName"
          v-model="local_name"
          class="pr-3 pl-4 thirdWidth"
          autocomplete="off"
          label="name"
          :rules="[rules.required]"
          :disabled="local_added"
          :error-messages="errorMessage"
          @input="removeErrorMessage"
        />
        <v-select
          v-model="local_type"
          label="Type"
          :items="local_types"
          class="thirdWidth"
          autocomplete="off"
          @input="changed"
        />
        <v-text-field
          v-if="local_type === 'string'"
          v-model="local_value"
          class="px-2 thirdWidth"
          autocomplete="off"
          label="value"
          @input="changed"
        />
        <v-text-field
          v-if="local_type === 'Vault'"
          v-model="local_value"
          class="px-2 thirdWidth"
          autocomplete="off"
          label="value"
          @input="changed"
        />
        <v-textarea
          v-if="local_type === 'text'"
          v-model="local_value"
          class="px-2 thirdWidth"
          autocomplete="off"
          rows="1"
          label="value"
          @input="changed"
        />
        <v-text-field
          v-if="local_type === 'password'"
          v-model="local_value"
          class="px-2 thirdWidth"
          autocomplete="off"
          label="value"
          :type="pass_locked ? 'password' : 'text'"
          :append-icon="pass_locked ? icons.mdiLock : icons.mdiLockOpen"
          @input="changed"
          @click:append="pass_locked = !pass_locked"
        />
        <div
          v-if="local_type === 'boolean'"
          class="thirdWidth"
          style="min-width:32%; max-height: 30px"
        >
          <v-checkbox
            v-model="local_value"
            color="lightblack"
            class="align-center justify-center"
            style="margin-top: 10px"
            ripple
            @change="changed"
          />
        </div>
        <v-text-field
          v-if="local_type === 'number'"
          v-model="local_value"
          class="px-2 thirdWidth"
          autocomplete="off"
          label="value"
          type="number"
          @input="changed"
        />
        <v-icon
          class="addRemoveIcon mr-3 mt-3 pb-0"

          @click="iconClicked"
          v-text="local_added ? icons.mdiMinus : icons.mdiPlus"
        />
      </div>
    </v-form>
  </transition>
</template>

<script>
  import { mdiLock, mdiLockOpen, mdiMinus, mdiPlus } from '@mdi/js'

  export default {
    name: 'NewConfigurationRow',
    props: {
      name: {
        type: String,
        default: null,
      },
      value: {
        type: [String, Boolean],
        default: null,
      },
      type: {
        type: String,
        default: 'string',
      },
      added: {
        type: Boolean,
        default: false,
      },
      filter: {
        type: String,
        default: '',
      },
      visible: {
        type: Boolean,
        default: true,
      },
    },
    data: attrs => ({
      icons: {
        mdiLock,
        mdiLockOpen,
        mdiMinus,
        mdiPlus,
      },
      local_name: attrs.name,
      local_value: attrs.value,
      local_type: attrs.type,
      local_added: attrs.added,
      local_types: ['string', 'number', 'password', 'boolean', 'text', 'Vault'],

      errorMessage: undefined,
      pass_locked: true,

      rules: {
        required: value => !!value || 'Required',
      },
    }),
    watch: {
      local_type (actual) {
        if (actual === 'boolean') {
          this.local_value = false
        }
      },
    },
    methods: {
      focus () {
        this.$refs.configName.focus()
      },
      iconClicked () {
        if (this.added) {
          this.$emit('remove_row', {
            name: this.local_name,
          })
        } else {
          if (this.$refs.newConfigForm.validate()) {
            this.$emit('add_row', {
              name: this.local_name,
              value: this.local_value,
              type: this.local_type,
              _this: this,
            })
          }
        }
      },
      changed () {
        this.$emit('change_row', {
          name: this.local_name,
          value: this.local_value,
          type: this.local_type,
          _this: this,
        })
      },
      reset () {
        this.$refs.newConfigForm.resetValidation()

        this.local_name = null
        this.local_value = null
        this.local_type = 'string'
        this.local_added = false
      },
      removeErrorMessage () {
        this.errorMessage = undefined
      },
    },
  }
</script>

<style scoped>
.addRemoveIcon {
padding-bottom: 10px;
width: 2%;
}

.thirdWidth {
max-width: 32%;
}

.invisible {
display: none;
}
</style>
