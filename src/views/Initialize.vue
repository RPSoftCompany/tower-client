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
  <v-container
    class="fill-height"
    fluid
  >
    <v-row
      align="center"
      justify="center"
    >
      <v-col
        cols="12"
        sm="4"
        style="max-width:389px;"
      >
        <v-card>
          <div class="px-4 pt-4">
            <v-img
              src="/tower.png"
              class="towerImage"
            />
          </div>
          <v-card-text>
            <v-form
              ref="form"
              v-model="form.valid"
            >
              <v-text-field
                v-model="encryption.key"
                label="Encryption key"
                counter
                maxlength="32"
                :rules="[rules.exists, rules.fullLength]"
                @keyup.enter="submit"
              />
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-btn
              color="primary"
              block
              :disabled="!form.valid"
              @click="submit"
            >
              Set encryption key
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'Initialize',
    data: props => ({
      form: {
        valid: false,
      },
      encryption: {
        key: '',
      },
      rules: {
        exists: v => !!v || 'Required',
        fullLength: v => v.length === 32 || 'Encyrption key must be 32 characters long',
      },
    }),
    methods: {
      validate () {
        return this.$refs.form.validate()
      },
      async submit () {
        if (!this.validate()) {
          return
        }

        await this.axios.post(
          `${this.$store.state.mainUrl}/configurations/initialize?secret=${this.encryption.key}`,
        )

        this.$router.push('/login')
      },
    },
  }
</script>
