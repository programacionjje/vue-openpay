<template>
    <v-container fluid fill-height>
        <v-layout align-center justify-center>
            <v-card>
                <v-form ref="form5" v-model="valid" lazy-validation class="pt-4">
                    <v-container grid-list-md>
                        <v-layout row wrap>
                            <v-flex xs12>
                                <v-text-field
                                    prepend-icon="fa-id-card"
                                    label="Nombre del titular"
                                    v-model="name_titular"
                                    type="text"
                                    required
                                    :rules="[v => !!v || 'El nombre del titular es requerido']"
                                    autocomplete="off"
                                ></v-text-field>
                            </v-flex>
                        </v-layout>
                        <v-layout row wrap>
                            <v-flex xs12>
                                <v-text-field
                                    prepend-icon="fa-id-card"
                                    label="Apellidos del titular"
                                    v-model="last_name_titular"
                                    type="text"
                                    required
                                    :rules="[v => !!v || 'Los apellidos del titular son requeridos']"
                                    autocomplete="off"
                                ></v-text-field>
                            </v-flex>
                        </v-layout>
                        <v-layout row wrap>
                            <v-flex xs12>
                                <v-text-field
                                    prepend-icon="fa-user"
                                    name="email"
                                    label="Correo electrónico"
                                    type="email"
                                    v-model="email"
                                    required
                                    autocomplete="off"
                                    :rules="emailRules"
                                ></v-text-field>
                            </v-flex>
                        </v-layout>
                        <v-layout row wrap>
                            <v-flex xs12>
                                <v-text-field
                                prepend-icon="fa-credit-card"
                                name="number_card"
                                label="Número de tarjeta"
                                type="text"
                                :mask="mask_credit_card"
                                v-model="number_card"
                                required
                                :rules="[v => !!v || 'El número de tarjeta es requerido']"
                                autocomplete="off"
                                ></v-text-field>
                            </v-flex>
                        </v-layout>
                        <v-layout row wrap>
                            <v-flex xs12>
                                <v-text-field
                                    prepend-icon="fa-credit-card-alt"
                                    name="cvc"
                                    label="CVC"
                                    type="number"
                                    v-model="cvv2"
                                    :rules="[rules_cvc.required, rules_cvc.min, rules_cvc.max]"
                                    required
                                    autocomplete="off"
                                ></v-text-field>
                            </v-flex>
                        </v-layout>
                        <v-layout row wrap>
                            <v-flex xs12 sm12 md6 lg6 xl6>
                                <v-select
                                    prepend-icon="fa-calendar-o"
                                    :items="items_months"
                                    v-model="month_expiration"
                                    label="Mes de expiración"
                                    required
                                    :rules="[v => !!v || 'El mes de expiración es requerido']"
                                ></v-select>
                            </v-flex>
                            <v-flex xs12 sm12 md6 lg6 xl6>
                                <v-text-field
                                    prepend-icon="fa-calendar-o"
                                    label="Año de expiración"
                                    v-model="year_expiration"
                                    :rules="rules_year"
                                    type="number"
                                    required
                                ></v-text-field>
                            </v-flex>
                        </v-layout>
                        <v-layout align-center justify-center column class="pt-2">
							<v-btn block :color="colorButton" class="white--text text-capitalize" @click="pay" :disabled="!valid">
								{{ mesageButton }}
                                <v-icon>{{ iconButton }}</v-icon>
							</v-btn>
						</v-layout>
                    </v-container>
                </v-form>
            </v-card>
        </v-layout>
    </v-container>
</template>

<script>
import axios from 'axios';

export default {
    data: () => ({
        valid: true,
        number_card: '',
        mask_credit_card: 'credit-card',
        rules_cvc: {
            required: v => !!v || 'El cvc es requerido',
            min: v => (v && v.length >= 3) || 'Mínimo 3 números',
            max: v => (v && v.length <= 4) || 'Máximo 4 números'
        },
        items_months: [
            { text: "01 - Enero", value: "01" },
            { text: "02 - Febrero", value: "02" },
            { text: "03 - Marzo", value: "03" },
            { text: "04 - Abril", value: "04" },
            { text: "05 - Mayo", value: "05" },
            { text: "06 - Junio", value: "06" },
            { text: "07 - Julio", value: "07" },
            { text: "08 - Agosto", value: "08" },
            { text: "09 - Septiembre", value: "09" },
            { text: "10 - Octubre", value: "10" },
            { text: "11 - Noviembre", value: "11" },
            { text: "12 - Diciembre", value: "12" },
        ],
        name_titular: '',
        email: '',
        last_name_titular: '',
        month_expiration: '',
        year_expiration: '',
        cvv2: '',
        rules_year: [
            v => !!v || 'El año de expiración es requerido',
            v => (v && v.length >= 2) || 'Mínimo 2 números',
            v => (v && v.length <= 2) || 'Máximo 2 números'
        ],
        emailRules: [
            v => !!v || 'El correo electrónico es requerido',
            v => /.+@.+/.test(v) || 'El correo electrónico debe de ser valido'
        ],
        token: '',
        endpoint_sandbox_openpay: 'https://sandbox-api.openpay.mx/v1/',
        endpoint_payament: 'http://laravel-openpay.test/api/charge',
        deviceSessionId: '',
        openpay_id: 'mfev2bl7srfn8rsqw20u',
        openpay_key: 'pk_f77dd2e0844045bcbaaadfdf2fce4b0e',
        openpay_sandbox_mode: true,
        mesageButton: 'Pagar',
        iconButton: 'attach_money',
        colorButton: 'primary'
    }),
    mounted() {
        // sistema de fraude
        OpenPay.setId(this.openpay_id);
        OpenPay.setApiKey(this.openpay_key);
        OpenPay.setSandboxMode(this.openpay_sandbox_mode);

        this.deviceSessionId = OpenPay.deviceData.setup();
    },
    methods: {
        pay () {
            // generate token para realizar el cargo
            OpenPay.token.create({
                "holder_name": this.name_titular + ' ' + this.last_name_titular,
                "card_number": this.number_card,
                "cvv2": this.cvv2,
                "expiration_month": this.month_expiration,
                "expiration_year": this.year_expiration
            }, (response) => {
                this.token = response.data.id;
            }, (error) => {
                console.log(error);
            });

            // payment in OpenPay
            axios.post(this.endpoint_payament, {
                name: this.name_titular,
                last_name: this.last_name_titular,
                email: this.email,
                token: this.token,
                deviceSessionId: this.deviceSessionId
            }).then((response) => {
                this.mesageButton = 'Pagado';
                this.loadingPay = false;
                this.iconButton = 'check_circle';
                this.colorButton = 'green';
                
                console.log(response)
            });
        }
    },
  }
</script>

<style>

</style>
