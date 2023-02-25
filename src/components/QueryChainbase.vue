<script setup>
defineProps({
    msg: String,
})
</script>

<script getChainbaseData>
import axios from 'axios'
import OneTable from './OneTable.vue'

export default {
    name: "ChainbaseData",
    data() {
        return {
            wallet_addr: '0xaeaecd497cd167ab0db77356f36eb9a68e888888',
            ens_domain: '',
            // ens_data: [],
            // ens_header: 'Name, Registrant Time, Expiration Time',
            // balance_data: [
            //     { 'Network': 'Ethereum', 'Balance': 0 },
            //     { 'Network': 'Polygon', 'Balance': 0 },
            //     { 'Network': 'Bnbchain', 'Balance': 0 },
            //     { 'Network': 'Avalanche', 'Balance': 0 },
            //     { 'Network': 'Fantom', 'Balance': 0 },
            //     { 'Network': 'Arbitrum', 'Balance': 0 },
            //     { 'Network': 'Aptos', 'Balance': 0 },
            // ],
            balance_data: {},
            balance_header: 'Network, Balance (Unit Wei)',
            network_id: {
                'Ethereum': 1,
                'Polygon': 137,
                'Bnbchain': 56,
                'Avalanche': 43114,
                'Fantom': 250,
                'Arbitrum': 42161,
                'Aptos': 2,
            },
            nft_data: new Set(),
            tx_data: [],
            tx_header: 'Timestamp, To Address, Value, Gas Fee'
        }
    },
    methods: {

        async getEnsDomain(wallet_addr) {
            alert('Query wallet: ' + wallet_addr);

            // Refer to https://docs.chainbase.com/reference/getensreverse.
            const options = {
                url: 'https://api.chainbase.online/v1/ens/reverse?chain_id=1&address=' + wallet_addr,
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            };

            const response = await axios(options);
            this.ens_domain = response.data.data[0].name;
        },

        // async getEnsDomain(wallet_addr) {
        //     alert('Query wallet: ' + wallet_addr);

        //     // Refer to https://docs.chainbase.com/reference/getaccountens.
        //     const options = {
        //         url: 'https://api.chainbase.online/v1/account/ens?chain_id=1&address=' + wallet_addr,
        //         method: 'GET',
        //         headers: {
        //             'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
        //             'Content-Type': 'application/json'
        //         }
        //     };

        //     const response = await axios(options);
        //     response.data.data.forEach((item, index) => {
        //         this.ens_data.push(
        //             {
        //                 "name": item.name,
        //                 "reg_time": item.registrant_time,
        //                 "exp_time": item.expiration_time,
        //             });
        //     });

        // },

        async getAccountBalance(wallet_addr) {
            // const idx = 0;
            alert('Query wallet: ' + wallet_addr);
            for (let key in this.network_id) {

                // Refer to https://docs.chainbase.com/reference/getaccountbalance.
                const options = {
                    url: 'https://api.chainbase.online/v1/account/balance?chain_id=' + this.network_id[key] + '&address=' + wallet_addr,
                    method: 'GET',
                    headers: {
                        'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                        'Content-Type': 'application/json'
                    }
                };

                // TODO - need to verify whether response.data is null or not.
                // await axios(options)
                //     .then(response => (this.$set(
                //         this.balance_data, idx++,
                //         {
                //             'Network': key,
                //             'Balance': parseInt(response.data.data, 16)
                //         }
                //     )));

                const response = await axios(options);
                this.balance_data[key] =
                {
                    'Network': key,
                    'Balance': parseInt(response.data.data, 16)
                };

            }
        },

        async getAccountNfts(wallet_addr) {
            alert('Query wallet: ' + wallet_addr);

            // Refer to https://docs.chainbase.com/reference/getaccountnfts.
            const options = {
                url: 'https://api.chainbase.online/v1/account/nfts?chain_id=1&address=' + wallet_addr,
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            };

            // await axios(options)
            //     .then(response => (this.nft_data.append(response.data.data)));

            const response = await axios(options);
            response.data.data.forEach((item, index) => {
                if (item.image_uri != "") {
                    if (item.image_uri.startsWith('ipfs://')) {
                        this.nft_data.add(item.image_uri.replace('ipfs://', 'https://cloudflare-ipfs.com/ipfs/'));
                    } else {
                        this.nft_data.add(item.image_uri);
                    }
                }
                // alert(`${item.image_uri}`);
            });
            // for (var i = 0; i < response.data.data.length; i++) {
            //     this.nft_data.push(response.data.data[i]);
            // }

        },

        async getAccountTxs(wallet_addr) {
            alert('Query wallet: ' + wallet_addr);

            // Refer to https://docs.chainbase.com/reference/getaccounttransactions.
            const options = {
                url: 'https://api.chainbase.online/v1/account/txs?chain_id=1&address=' + wallet_addr,
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            };

            const response = await axios(options);
            response.data.data.every((item, index) => {
                if (index >= 20) {
                    return false;
                }
                this.tx_data.push(
                    {
                        "timestamp": item.block_timestamp,
                        "to_addr": item.to_address,
                        "value": item.value,
                        "gas_fee": item.gas_used,
                    });
                // alert(`${item.block_timestamp}`);
                return true;
            });

        },
    },
}
</script>

<template>
    <h1>{{ msg }}</h1>

    <div class="instruction">
        <p>Click to get account portfolio overview.</p>
        <p>Please input your wallet address:</p>
        <p>
            <textarea class="input-text" v-model="wallet_addr" placeholder="Please input your wallet address..."></textarea>
        </p>
    </div>

    <div class="div">
        <h2>Get ENS domain.</h2>
        <div class="interaction">
            <button @click="getEnsDomain(wallet_addr)">Get ENS domain</button>
        </div>

        <h4>{{ ens_domain }}.eth</h4>

        <!-- <div id="ChainbaseData">
            <p>
                <OneTable :header="ens_header" :body="ens_data"></OneTable>
            </p>
        </div> -->
    </div>

    <div>
        <p></p>
    </div>


    <div class="div">
        <h2>Get account balance.</h2>
        <div class="interaction">
            <!-- <button type="button" @click="count++">count is {{ count }}</button> -->
            <button @click="getAccountBalance(wallet_addr)">Get native token balance</button>
        </div>

        <div id="ChainbaseData">
            <p>
                <OneTable :header="balance_header" :body="balance_data"></OneTable>
            </p>
        </div>
    </div>

    <div>
        <p></p>
    </div>


    <div class="div">
        <h2>Get account NFTs.</h2>
        <div class="interaction">
            <button @click="getAccountNfts(wallet_addr)">Get NFTs</button>
        </div>

        <div id="ChainbaseData">
            <p>
                <li v-for="(item, index) in nft_data" :key="index">
                    <img :src="`${item}`" :height="200" :width="200" />
                    <!-- <p>{{ item }}</p> -->
                </li>
            </p>
        </div>
    </div>

    <div>
        <p></p>
    </div>


    <div class="div">
        <h2>Get 20 most recent account transactions.</h2>
        <div class="interaction">
            <button @click="getAccountTxs(wallet_addr)">Get account transactions</button>
        </div>

        <div id="ChainbaseData">
            <p>
                <OneTable :header="tx_header" :body="tx_data"></OneTable>
            </p>
        </div>
    </div>
</template>

<style>
.div {
    border: 2px #000 solid;
}

.input-text {
    width: 50%;
    text-align: center;
}
</style>

