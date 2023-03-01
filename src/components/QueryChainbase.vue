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
            balance_data: [
                { 'Network': 'Ethereum', 'Balance': 0 },
                { 'Network': 'Polygon', 'Balance': 0 },
                { 'Network': 'Bnbchain', 'Balance': 0 },
                { 'Network': 'Avalanche', 'Balance': 0 },
                { 'Network': 'Fantom', 'Balance': 0 },
                { 'Network': 'Arbitrum', 'Balance': 0 },
                // { 'Network': 'Aptos', 'Balance': 0 },
            ],
            balance_header: 'Network, Balance (Unit Wei)',
            network_id: {
                'Ethereum': 1,
                'Polygon': 137,
                'Bnbchain': 56,
                'Avalanche': 43114,
                'Fantom': 250,
                'Arbitrum': 42161,
                // 'Aptos': 2,
            },
            nft_data: new Set(),
            tx_data: [],
            tx_header: 'Timestamp, To Address, Value, Gas Fee',
            contract_addr: '0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984',
            holder_header: 'Address, Total',
            holder_data: [],
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
                fetch('https://api.chainbase.online/v1/account/balance?chain_id=' + this.network_id[key] + '&address=' + wallet_addr, {
                    method: 'GET',
                    headers: {
                        'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                        'Content-Type': 'application/json'
                    }
                }).then(response => response.json())
                    .then(data => {
                        console.log(key, data.data);
                        this.balance_data.forEach((item, index) => {
                            if (item.Network == key) {
                                item.Balance = parseInt(data.data, 16);
                            }
                        });
                    }).catch(error => console.error(error));
            }
        },

        async getAccountNfts(wallet_addr) {
            alert('Query wallet: ' + wallet_addr);

            // Refer to https://docs.chainbase.com/reference/getaccountnfts.
            fetch('https://api.chainbase.online/v1/account/nfts?chain_id=1&address=' + wallet_addr, {
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            }).then(response => response.json())
                .then(data => {
                    console.log(data.data);
                    data.data.forEach((item, index) => {
                        if (item.image_uri != "") {
                            if (item.image_uri.startsWith('ipfs://')) {
                                this.nft_data.add(item.image_uri.replace('ipfs://', 'https://cloudflare-ipfs.com/ipfs/'));
                            } else {
                                this.nft_data.add(item.image_uri);
                            }
                        }
                    })
                }).catch(error => console.error(error));
        },

        async getAccountTxs(wallet_addr) {
            alert('Query wallet: ' + wallet_addr);

            // Refer to https://docs.chainbase.com/reference/getaccounttransactions.
            fetch('https://api.chainbase.online/v1/account/txs?chain_id=1&address=' + wallet_addr, {
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            }).then(response => response.json())
                .then(data => {
                    console.log(data.data);
                    data.data.every((item, index) => {
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
                        return true;
                    })
                }).catch(error => console.error(error));
        },

        async getTokenHolder(contract_addr) {
            alert('Query contract address: ' + contract_addr);
            const query_str =
                "WITH\
                    tokens AS (\
                        SELECT\
                            _from AS address,\
                            toInt256(- _value) AS v\
                        FROM\
                            ethereum.erc20_transfer\
                        WHERE\
                            contract_address = '" + `${contract_addr}` + "'\
                        UNION ALL\
                        SELECT\
                            _to AS address,\
                            toInt256(_value) AS v\
                        FROM\
                            ethereum.erc20_transfer\
                        WHERE\
                            contract_address = '" + `${contract_addr}` + "'\
                    )\
                SELECT\
                    address,\
                    sum(v) AS total\
                FROM\
                    tokens\
                GROUP By\
                    address\
                ORDER BY\
                    total DESC\
                LIMIT\
                    10";

            // Refer to https://docs.chainbase.online/r/data-cloud-studio/data-cloud-api.
            const url = 'https://api.chainbase.online/v1/dw/query';
            const proxy_url = 'https://thingproxy.freeboard.io/fetch/';
            fetch(proxy_url + url, {
                method: 'POST',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json; charset=utf-8',
                },
                data: {
                    query: query_str,
                },
            }).then(response => {
                console.log(response);
            })
                .then(data => {
                    console.log(data);
                    // this.holder_data = data.result;
                }).catch(error => console.error(error));

            // const options = {
            //     url: 'https://api.chainbase.online/v1/dw/query',
            //     method: 'POST',
            //     headers: {
            //         'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
            //         'Content-Type': 'application/json; charset=utf-8',
            //         'Access-Control-Allow-Origin': 'http://localhost:5173/', // replace with your Vue app URL
            //         'Access-Control-Allow-Credentials': true,
            //     },
            //     data: {
            //         query: query_str,
            //     },
            //     credentials: 'include',
            // };

            // console.log(query_str);

            // const response = await axios(options);
            // this.holder_data = response.data.result;
            // console.log(this.holder_data);
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

    <div>
        <p></p>
    </div>


    <!-- <div class="div">
        <h2>Get token holder top 10.</h2>
        <p>Please input the contract address of one ERC-20 token:</p>
        <p>
            <textarea class="input-text" v-model="contract_addr"
                placeholder="Please input the contract address..."></textarea>
        </p>
        <p>Note that only ERC-20 tokens are allowed due to the internal SQL settings.</p>
        <p>For example, Filecoin is not an ERC-20 token so it cannot be queried as for now.</p>
        <div class="interaction">
            <button @click="getTokenHolder(contract_addr)">Get token holder</button>
        </div>

        <div id="ChainbaseData">
            <p>
                <OneTable :header="holder_header" :body="holder_data"></OneTable>
            </p>
        </div>
    </div> -->
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

