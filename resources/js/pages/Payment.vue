<script>
import SliderCart from '../components/partials/SliderCart.vue';
import {store} from '../data/store'
import {ApiService} from '../services/api.service';
import Loader from '../components/partials/Loader.vue';
export default {
    name: "Payment",
    token: "",
    components: {
        SliderCart,
        Loader
    },
    data() {
        return {
            ApiService,
            store,
            cart        : null,
            checkCart   : true,
            checkData   : false,
            checkPayment: false,
            name        : '',
            surname     : '',
            address     : '',
            email       : '',
            message     : '',
            isLoading   : false
        };
    },
    methods: {
        getPayment() {
        const form = document.getElementById("payment-form");

        braintree.dropin
            .create({
            authorization: this.token,
            container: "#dropin-container",

            })
            .then((dropinInstance) => {
            form.addEventListener("submit", (event) => {
                event.preventDefault();
                dropinInstance
                .requestPaymentMethod()
                .then((payload) => {
                    document.getElementById('nonce').value = payload.nonce;
                    form.submit();
                    this.isLoading = true;
                })
                .catch((error) => {
                    console.log(error);;
                });
            });

            })
            .catch((error) => {
                console.log(error);
            });

        },
        checkDataForm(){
            if(this.name === '' && this.surname === '' && this.email === '' && this.address === ''){
                this.message = 'COMPILA TUTTI I CAMPI PER POTER PASSARE AL PROSSIMO STEP';
            }else {
                this.checkData = false;
                this.checkPayment = true;
            }
        }
    },
    async mounted(){
        store.show_cart = false;
        store.is_modal  = false;
        let res = await this.ApiService.getApi('payment/generate', {});
        this.token = res.token;
        this.getPayment();
    }
};
</script>

<template>

    <div v-if="!isLoading" class="container mf-container">
        <h1 class="text-center font-bold mb-5">
            IMPOSTAZIONI DI PAGAMENTO
        </h1>
        <div class="row mb-5 mf-row row-cols-3">
            <div :class="{'active' : checkCart}" class="col">
                CARRELLO
            </div>
            <div :class="{'active' : checkData}" class="col">
                DATI
            </div>
            <div :class="{'active' : checkPayment}" class="col">
                PAGAMENTO
            </div>
        </div>
        <!-- CHECK CART -->
        <div v-if="checkCart">
            <SliderCart :cart="store.shopping_cart" />
            <div class="text-center mt-5">
                <button v-if="store.shopping_cart.foods.length !== 0" @click="checkCart = false, checkData = true">
                    VAI AL PROSSIMO STEP
                </button>
                <div v-else class="text-danger">
                    <h4 class="mb-4">
                        Impossibile procedere al pagameno, devi aggiungere almeno un prodotto al carrello per effettuare l'ordine.
                    </h4>

                    <a href="/" class="text-light text-decoration-none">Torna alla home</a>
                </div>
            </div>
        </div>
        <!-- //CHECK CART -->

        <!-- FORM PAYMENT -->
        <form

            id="payment-form"
            action="/api/payment/pay"
            method="post"
            @submit.prevent="submit"
        >
            <div :class="{'d-none' : !checkData}" class="container w-50" >
                <h3 class="py-3 text-center">Inserisci i dati per completare l'ordine</h3>
                <div class="mb-3">
                    <label for="name" class="form-label"> Nome *</label>
                    <input  v-model="name" required type="text" class="form-control" id="name" name="name" placeholder="UGO">
                </div>
                <div class="mb-3">
                    <label for="surname" class="form-label">Cognome *</label>
                    <input v-model="surname" required type="text" class="form-control" id="surname" name="surname" placeholder="DE UGHI">
                </div>
                <div class="mb-3">
                    <label for="address" class="form-label">Indirizzo *</label>
                    <input v-model="address" required type="text" class="form-control" id="address" name="address" placeholder="Via dei fiocchi, 12">
                </div>
                <div class="mb-3">
                    <label for="email" class="form-label">Email * </label>
                    <input v-model="email" required type="email" class="form-control" id="email" name="email" placeholder="name@example.com">
                </div>
                <input type="hidden" name="token" id="token" value="fake-valid-nonce">
                <input type="hidden" name="cart" id="token" :value="JSON.stringify(store.shopping_cart)">

                <div class="text-center mt-5">
                    <a class="text-white text-decoration-none" @click="checkDataForm()">
                        VAI AL PAGAMENTO
                    </a>
                    <p class="text-danger" v-if="message">
                        {{ message }}
                    </p>
                </div>
            </div>
            <div :class="{'d-none' : !checkPayment}" id="dropin-wrapper" class="d-flex justify-center align-items-center flex-column">
                <div id="checkout-message"></div>
                <div id="dropin-container"></div>

                <div class="text-center">
                    <button id="submit-button" type="submit" >
                        PAGA
                    </button>
                </div>

                <input type="hidden" id="nonce" name="payment_method_nonce"  />
            </div>
        </form>
        <!--// FORM PAYMENT -->


    </div>
    <div v-else class="text-center mf-container">
        <Loader />
    </div>

</template>

<style lang="scss" scoped>
.mf-container {
    padding-top: 100px;
}
.mf-row {
    .col {
        background-color: #828887;
        color: white;
        padding: 20px;
        text-align: center;
        &.active {
            background-color: #24645b;
            color: white;
            border-bottom: 4px solid #43efce;
        }
    }
}

button, a {
    border: none;
    color: white;
    background-color: #26635B;
    padding: 8px 15px;
    margin-bottom: 10px;
    border-radius: 5px;
    cursor: pointer;
    display: inline-block;
}
</style>
