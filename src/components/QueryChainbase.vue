<script setup>
import { ref } from 'vue'

defineProps({
    msg: String,
})

const count = ref(0)
</script>

<script getChainbaseData>
import axios from 'axios'

export default {
    name: "ChainbaseData",
    data() {
        return {
            response: '',
        }
    },
    methods: {
        getAccountTxs(event) {
            const options = {
                url: 'https://api.chainbase.online/v1/account/txs?chain_id=1&address=0xaeaecd497cd167ab0db77356f36eb9a68e888888',
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            };

            axios(options)
                .then(response => (this.response = response));


            // const response = axios(options);
            // this.msg = response.data;
            // alert(`${this.response}`);

        },
    }
}
</script>

<template>
    <h1>{{ msg }}</h1>

    <div id="ChainbaseData">
        {{ response }}
    </div>

    <div class="interaction">
        <!-- <button type="button" @click="count++">count is {{ count }}</button> -->
        <button @click="getAccountTxs">Get Account Transactions</button>
    </div>

    <div class="instruction">
        <p>
            Click to calculate credit score.
        </p>
    </div>


</template>

