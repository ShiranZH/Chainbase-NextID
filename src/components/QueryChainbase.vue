<script setup>
defineProps({
    msg: String,
})
</script>

<script getChainbaseData>
import axios from 'axios'

export default {
    name: "ChainbaseData",
    data() {
        return {
            wallet_addr: '',
            tx_data: '',
        }
    },
    methods: {
        getAccountTxs(wallet_addr) {
            const options = {
                url: 'https://api.chainbase.online/v1/account/txs?chain_id=1&address=' + wallet_addr,
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            };

            alert('Query wallet: ' + wallet_addr);

            // TODO - need to verify whether response.data is null or not.
            axios(options)
                .then(response => (this.tx_data = response.data.data));

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
        <ul>
            <li v-for="tx in tx_data" :key="tx.block_number">
                {{ tx.block_number }} - {{ tx.gas_used }} - {{ tx.to_address }}
            </li>
        </ul>
    </div>

    <div class="interaction">
        <!-- <button type="button" @click="count++">count is {{ count }}</button> -->
        <p>
            <textarea v-model="wallet_addr" placeholder="Please input your wallet address..."></textarea>
        </p>
        <button @click="getAccountTxs(wallet_addr)">Get Account Transactions</button>
    </div>

    <div class="instruction">
        <p>
            Click to get account portfolio overview.
        </p>
    </div>
</template>

