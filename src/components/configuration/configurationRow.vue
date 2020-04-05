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
    name="slowfade"
    mode="out-in"
  >
    <v-form
      v-show="visible"
      ref="newConfigForm"
      autocomplete="off"
      :class="getClass"
    >
      <div
        class="d-flex flex-row justify-space-around"
        :class="{crossed: deleted, 'font-weight-light': deleted, 'font-italic': deleted}"
        :style="local_type === 'boolean' ? 'height:70px' : ''"
      >
        <div
          class="px-2 text-center history"
          label="name"
          :style="
            versions.length > 0 || deleted ? 'min-width:33.33%' : 'min-width:50%'
          "
          v-text="local_name"
        />
        <div
          v-if="versions.length > 0 || deleted"
          :style="
            versions.length > 0 || deleted ? 'min-width:33.33%' : 'min-width:50%'
          "
          class="px-2 text-center history"
          :class="{ draft : draft, 'font-weight-light': draft, 'font-italic': draft}"
          v-text="
            local_type === 'password' && pass_locked
              ? '********'
              : deleted === true ? local_value : versions[current_version]
          "
        />
        <v-text-field
          v-if="local_type === 'string'"
          v-model="local_value"
          :style="
            versions.length > 0 || deleted ? 'min-width:33.33%' : 'min-width:50%'
          "
          class="px-2"
          autocomplete="off"
          label="value"
          :rules="local_rules"
          :disabled="forced_value"
          :hint="force_cause"
          persistent-hint
          @input="change"
        />
        <v-text-field
          v-if="local_type === 'Vault'"
          v-model="local_value"
          :style="
            versions.length > 0 || deleted ? 'min-width:33.33%' : 'min-width:50%'
          "
          class="px-2"
          autocomplete="off"
          label="value"
          :rules="local_rules"
          :disabled="forced_value"
          :hint="force_cause"
          persistent-hint
          @input="change"
        />
        <v-textarea
          v-if="local_type === 'text'"
          v-model="local_value"
          :style="
            versions.length > 0 || deleted ? 'min-width:33.33%' : 'min-width:50%'
          "
          class="px-2"
          autocomplete="off"
          rows="1"
          label="value"
          :rules="local_rules"
          :disabled="forced_value"
          :hint="force_cause"
          persistent-hint
          @input="change"
        />
        <v-text-field
          v-if="local_type === 'password'"
          v-model="local_value"
          :style="
            versions.length > 0 || deleted ? 'min-width:33.33%' : 'min-width:50%'
          "
          class="px-2"
          autocomplete="off"
          label="value"
          :type="pass_locked ? 'password' : 'text'"
          :append-icon="pass_locked ? icons.mdiLock : icons.mdiLockOpen"
          :rules="local_rules"
          :disabled="forced_value"
          :hint="force_cause"
          persistent-hint
          @input="change"
          @click:append="pass_locked = !pass_locked"
        />
        <div
          v-if="local_type === 'boolean'"
          class="px-2"
          :style="
            versions.length > 0 || deleted
              ? 'min-width:33.33%'
              : 'min-width:50%'
          "
        >
          <v-checkbox
            v-model="local_value"
            color="lightblack"
            class="align-center justify-center"
            style="margin-top: 20px; width: 100%"
            ripple
            :rules="local_rules"
            :disabled="forced_value"
            :hint="force_cause"
            persistent-hint
            @change="change"
          />
        </div>
        <v-text-field
          v-if="local_type === 'number'"
          v-model="local_value"
          :style="
            versions.length > 0 || deleted ? 'min-width:33.33%' : 'min-width:50%'
          "
          class="px-2"
          autocomplete="off"
          label="value"
          type="number"
          :rules="local_rules"
          :disabled="forced_value"
          :hint="force_cause"
          persistent-hint
          @input="change"
        />
      </div>
    </v-form>
  </transition>
</template>

<script>
  import { mdiLock, mdiLockOpen } from '@mdi/js'

  export default {
    name: 'CongiurationRow',
    props: {
      name: {
        type: String,
        required: true,
      },
      versions: {
        type: Array,
        required: false,
      },
      value: {
        type: [String, Boolean],
        default: '',
      },
      type: {
        type: String,
        required: true,
      },
      rules: {
        type: Array,
        required: false,
        default: () => {
          return new Array()
        },
      },
      deleted: {
        type: Boolean,
        default: false,
      },
      current_version: {
        type: Number,
        required: true,
      },
      forced_value: {
        type: Boolean,
        default: false,
      },
      force_cause: {
        type: String,
      },
      visible: {
        type: Boolean,
        default: true,
      },
      draft_versions: {
        type: Array,
        required: true,
      },
    },
    data: attrs => ({
      icons: {
        mdiLock,
        mdiLockOpen,
      },
      local_name: attrs.name,
      local_value:
        attrs.type === 'boolean'
          ? attrs.value === 'true'
          : attrs.value,
      local_type: attrs.type,
      local_rules: [],

      pass_locked: true,
    }),
    computed: {
      draft () {
        return this.draft_versions.includes(this.current_version)
      },
      getClass () {
        const base = this.different ? 'different' : 'noColor'

        return base
      },
      historicLabel () {
        return `Version #${this.current_version}`
      },
      different () {
        if (this.deleted) {
          if (this.local_type !== 'boolean') {
            return this.local_value
          } else {
            if (this.local_value === true) {
              return 'true'
            } else {
              return 'false'
            }
          }
        }

        if (this.local_type !== 'boolean') {
          return this.versions[this.current_version] !== this.local_value
        } else {
          if (this.local_value === true) {
            return this.versions[this.current_version] !== 'true'
          } else {
            return this.versions[this.current_version] !== 'false'
          }
        }
      },
    },
    watch: {},
    mounted () {
      this.createLocalRules()
    },
    methods: {
      change () {
        this.$emit('change', {
          name: this.local_name,
          value: this.local_value,
          type: this.local_type,
        })
      },
      createLocalRules () {
        this.rules.forEach(rule => {
          const regex = new RegExp(rule.value)
          this.local_rules.push(v => regex.test(v) || rule.error)
        })
      },
      valid () {
        return this.$refs.newConfigForm.validate()
      },
    },
  }
</script>

<style scoped>
.thirdWidth {
  max-width: 32%;
}

.halfWidth {
  max-width: 50%;
}

.different {
  background: #f2a52a69;
  transition: all 0.3s;
}

.draft {
  background: rgba(64, 67, 78, 0.15);
  transition: all 0.3s;
}

.noColor {
  background: rgba(255, 255, 255, 0);
  transition: all 0.3s;
}

.invisible {
  display: none;
}

.history {
  padding-top: 20px;
}

.crossed {
  text-decoration: line-through;
}
</style>
