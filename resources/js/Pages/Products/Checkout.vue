<template>
    <AuthenticatedLayout>
        <div class="container mx-auto max-w-[35rem] p-5 relative">
            <div v-if="step===1">
                <p class="font-bold text-3xl dark:text-gray-100">Confirmação do pedido</p>
                <p class="text-gray-500 md:text-sm mt-2 dark:text-gray-400">
                    Valide os dados do seu pedido, caso tudo esteja de acordo clique em continuar, caso queira voltar
                    para a tela do sorteio para fazer alguma alteração, clique em voltar.
                </p>
                <div class="my-4">
                    <div>
                        <img
                            src="https://quatrorodas.abril.com.br/wp-content/uploads/2018/04/corolla_gli_2019.jpg?quality=70&strip=info"
                            alt="" srcset="" class="rounded-2xl">
                    </div>
                    <div class="flex flex-col mt-3">
                        <p class="font-bold text-lg dark:text-gray-100">{{ product.name }}</p>
                        <p class="dark:text-gray-400">R$ {{ product.share_price }} / número</p>
                        <p class="dark:text-gray-400">{{ quantity }} números</p>
                    </div>
                </div>
                <hr>
                <div class="my-4">
                    <p class="md:text-sm dark:text-gray-400">Total</p>
                    <p class="font-bold text-lg dark:text-gray-100">R$ {{ product.share_price * quantity }}</p>
                    <div class="flex flex-row gap-3 mt-2">
                        <PrimaryButton @click="confirmCheckout()">Prosseguir</PrimaryButton>
                        <SecondaryButton>Voltar</SecondaryButton>
                    </div>
                </div>
            </div>
            <div v-if="step===2">
                <button @click="step--"
                        class="text-gray-600 md:text-sm flex flex-row items-center gap-2 rounded-lg mb-2 hover:bg-gray-100 w-fit px-2 py-1 cursor-pointer">
                    <i class="bi bi-chevron-left"></i> Voltar
                </button>
                <p class="font-bold text-3xl dark:text-gray-100">Pagamento</p>
                <p class="text-gray-500 md:text-sm mt-2 dark:text-gray-400">
                    Para finalizarmos o checkout e você receber os seus números da sorte, é preciso efetuar o pagamento
                    via PIX. Você pode utilizar tanto o QR code quanto a chave copia e cola.
                </p>

                <div class="border dark:border-slate-700 p-2 rounded-lg w-48 aspect-square mt-4">
                    <img :src="'data:image\png;base64,' + qrCode" alt=""
                         class="w-full aspect-square">
                </div>
                <p class="mt-3 mb-1.5 font-semibold dark:text-gray-100">Chave copia e cola:</p>
                <div class="flex flex-row w-full">
                    <div
                        class="rounded-l border-emerald-400 border-dashed border-2 bg-gray-50 dark:bg-transparent p-2 overflow-hidden w-full">
                        <div class="flex flex-row justify-between gap-1">
                            <p class="text-emerald-400 font-bold whitespace-nowrap text-sm">
                                {{ pix.pixCopiaECola }}
                            </p>
                        </div>
                    </div>

                    <button
                        class="bg-gray-200 grid place-items-center py-2 px-4 dark:bg-slate-800 rounded-r border border-gray-300 cursor-pointer rounded-l-none"
                        @click="copyPIX()">
                        <i class="bi bi-clipboard text-gray-600 dark:text-gray-400"></i>
                    </button>
                </div>
            </div>
            <div v-if="step===3">
                <div class="flex flex-col items-center text-center">
                    <img src="https://www.freeiconspng.com/uploads/success-icon-10.png" class="w-56 mb-2" alt="">
                    <div class="">
                        <h1 class="font-bold text-2xl mt-2 dark:text-gray-100">Pagamento confirmado</h1>
                        <p class="text-gray-600 dark:text-gray-400 mt-1 mb-3">
                            Seu pagamento foi confirmado e
                            seus números
                            <span v-if="isGenerating">estão sendo </span>
                            <span v-else>foram </span>
                            gerados!
                            <br>
                            Cruze os dedos e boa sorte!
                        </p>
                        <Link v-if="isGenerating" :href="route('index')">
                            <PrimaryButton>Voltar ao início</PrimaryButton>
                        </Link>
                        <Link v-else :href="route('numbers')">
                            <PrimaryButton>Ver meus sorteios</PrimaryButton>
                        </Link>
                    </div>
                </div>
            </div>
        </div>
    </AuthenticatedLayout>
</template>

<script setup>
import {reactive, ref} from "vue";
import PrimaryButton from "@/Components/PrimaryButton.vue";
import SecondaryButton from "@/Components/SecondaryButton.vue";
import {Link} from '@inertiajs/inertia-vue3'
import axios from "axios"
import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";
import $page from "../../../../public/build/assets/Edit.23e3abee";

const step = ref(1);
const qrCode = ref("");
const sandbox_token = ref("");
const pix = reactive({});
const timer = ref(10);
const isGenerating = ref(true);

const props = defineProps({
    product: Object,
    quantity: Number,
})

const sandbox_url = "https://devportal.itau.com.br/sandboxapi";

function generateToken() {
    return axios({
        method: "POST",
        url: `https://devportal.itau.com.br/api/jwt`,
        data: {
            client_id: "d4b0717d-da08-3baa-adee-77eb85f59566",
            client_secret: "f7ea714f-fdd7-4a4d-990e-f081fa7af269"
        }
    })
}

function generatePIX() {
    return axios({
        method: "POST",
        url: `${sandbox_url}/pix_recebimentos_ext_v2/v2/cob`,
        data: {
            calendario: {
                expiracao: 3600
            },
            valor: {
                original: "20.00",
                modalidadeAlteracao: 1
            },
            chave: "7d9f0335-8dcc-4054-9bf9-0dbd61d36906"
        },
        headers: {
            "Authorization": `Bearer ${sandbox_token.value}`,
            "x-itau-apikey": "d4b0717d-da08-3baa-adee-77eb85f59566", // client_id
            "x-sandbox-token": sandbox_token.value
        }
    })
}

function generateQRCode() {
    return axios({
        url: `${sandbox_url}/pix_recebimentos_ext_v2/v2/cob/${pix.txid}/qrcode`,
        method: 'GET',
        headers: {
            "Authorization": `Bearer ${sandbox_token.value}`,
            "x-itau-apikey": "d4b0717d-da08-3baa-adee-77eb85f59566", // client_id
            "x-sandbox-token": sandbox_token.value
        }
    })
}

function setPIX() {
    this.generateToken().then((r) => {
        this.sandbox_token = r.data.access_token

        this.generatePIX().then((r) => {
            this.pix = r.data

            this.generateQRCode().then((r) => {
                this.qrCode = r.data.imagem_base64
                // this.countDownTimer()
            })
        })
    })
}

function confirmCheckout() {
    try {
        this.setPIX()
        this.countDownTimer()
    } finally {
        this.step++
    }
}

function copyPIX() {
    navigator.clipboard.writeText(this.pix.pixCopiaECola)
    alert('PIX copiado com sucesso.')
}

function generateNumbers() {
    axios({
        method: 'POST',
        url: route('generate-numbers', {
            product_id: props.product.id,
            quantity: props.quantity,
        })
    }).then((r) => {
        console.log(r.data)
        this.isGenerating = false
    })
}

function endCheckout() {
    generateNumbers();
    this.step++
}

function showPIX() {
    return axios({
        method: 'GET',
        url: `${sandbox_url}/pix_recebimentos_ext_v2/v2/cob/${this.pix.txid}`,
        data: {},
        headers: {
            "Authorization": `Bearer ${sandbox_token.value}`,
            "x-itau-apikey": "d4b0717d-da08-3baa-adee-77eb85f59566", // client_id
            "x-sandbox-token": sandbox_token.value
        }
    })
}

function countDownTimer() {
    if (this.timer > 0) {
        setTimeout(() => {
            this.timer -= 1
            console.log(this.timer)
            this.countDownTimer()
        }, 1000)
    }

    if (this.timer === 0) {
        console.log('zerou')
        try {
            this.generateToken().then((r) => {
                this.sandbox_token = r.data.access_token

                this.showPIX().then((r) => {
                    if (r.data.status === 'CONCLUIDA') {
                        console.log('concluiu')
                        this.endCheckout()
                        this.timer = -1
                    } else {
                        this.timer = 20
                        console.log('não concluiu')
                        this.countDownTimer()
                    }
                })
            })
        } catch (e) {
            this.timer = 30
            console.log('deu erro')
            this.countDownTimer()
        }
    }
}
</script>
