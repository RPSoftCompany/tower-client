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
  <v-form
    ref="variableValidationForm"
    class="d-flex"
  >
    <v-text-field
      v-model="in_name"
      label="Variable Name"
      :rules="[rules.required]"
      class="pl-5 pr-2"
      :disabled="!isNew"
    />
    <v-text-field
      v-model="in_value"
      label="Value"
      :rules="[rules.required]"
      :append-outer-icon="in_isNew ? icons.mdiPlus : icons.mdiMinus"
      class="pl-2 pr-5"
      @click:append-outer="addVariable"
      @change="modifyVariable"
    />
  </v-form>
</template>

<script>
  import { mdiPlus, mdiMinus } from '@mdi/js'

  export default {
    name: 'Variable',
    props: {
      name: {
        type: String,
        default: null,
        required: false,
      },
      value: {
        type: String,
        default: null,
        required: false,
      },
      isNew: {
        type: Boolean,
        default: true,
        required: false,
      },
      id: {
        type: String,
        default: null,
        required: false,
      },
    },
    data: attrs => {
      return {
        icons: {
          mdiPlus,
          mdiMinus,
        },
        in_name: attrs.name,
        in_value: attrs.value,
        in_isNew: attrs.isNew,
        in_id: attrs.id,

        rules: {
          required: value => !!value || 'Required',
        },
      }
    },
    methods: {
      addVariable () {
        if (this.in_isNew) {
          if (this.$refs.variableValidationForm.validate()) {
            this.$emit('add_variable', {
              name: this.in_name,
              value: this.in_value,
              _this: this,
            })
          }
        } else {
          this.$emit('remove_variable', {
            id: this.in_id,
          })
        }
      },
      modifyVariable () {
        if (!this.in_isNew) {
          if (this.$refs.variableValidationForm.validate()) {
            this.$emit('modify_variable', {
              id: this.in_id,
              name: this.in_name,
              value: this.in_value,
            })
          }
        }
      },
      reset () {
        this.$refs.variableValidationForm.reset()

        this.in_name = null
        this.in_value = null
        this.isNew = true
        this.in_id = null
      },
    },
  }
</script>
