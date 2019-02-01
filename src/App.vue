<template>
    <v-app>
        <v-toolbar app>
            <v-toolbar-title class="headline text-uppercase">
                <span>Programación</span>
                <span class="ml-2 font-weight-light">JJE</span>
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn
                flat
                href="https://github.com/programacionjje"
                target="_blank">
                <span class="mr-2">View project in GitHub</span>
            </v-btn>
        </v-toolbar>

        <v-content>
            <FormPayment/>
        </v-content>
    </v-app>
</template>

<script>
import FormPayment from './components/FormPaymentComponent'

export default {
  name: 'App',
  components: {
    FormPayment
  },
  data () {
    return {
      //
    }
  }
}
</script>
