<template>
  <div class="d-flex justify-center align-center fill-height">
    <v-card 
      class="pa-6 mx-auto"
      elevation="6"
    >
      <v-form
        ref="form"
        v-model="valid"
        lazy-validation
      >
        <v-text-field
          v-model="name"
          :rules="nameRules"
          label="Name"
          md6
          sm6
          xs6
          required
        ></v-text-field>

        <v-text-field
          v-model="room"
          :rules="roomRules"
          label="Room"
          md6
          sm6
          xs6
          required
        ></v-text-field>


        <v-btn
          :disabled="!valid"
          color="primary"
          class="mr-4 mt-4"
          block
          @click="validate"
        >
          JOIN
        </v-btn>


      </v-form>

    </v-card>
  </div>
</template>

<script lang="ts">
import { Component , Vue } from 'vue-property-decorator';

@Component
export default class Login extends Vue {
  
  valid = true
  name = ''
  nameRules = [
    (v: any) => !!v || 'Name is required',
  ]
  room = ''
  roomRules = [
    (v: any) => !!v || 'Room is required',
  ]

  validate () {
    // eslint-disable-next-line
    this.$refs.form.validate()
    if(this.valid) {
      this.$router.push({ name: 'Draw', params: { username: this.name, room: this.room } })
    }
  }
  
}

</script>
