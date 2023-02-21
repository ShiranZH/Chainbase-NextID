<template>
    <div>
        <table class="center" border="1" width="500px">
            <!-- <template v-for="(item, index) in header.split(',')" :key="index">
                <el-table-column :label="item"></el-table-column>
            </template> -->
            <thead>
                <tr>
                    <th v-for="(item, index) in header.split(',')" :key="index">{{ item }}</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in body" :key="index">
                    <td v-for="(val, index) in item" :key="index">{{ val }}</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>
  
<script>
export default {
    name: "one",
    props: {
        header: {
            type: String,
            default: "header"
        },
        body: {
            type: Array,
            default: "body"
        }
    }
};

</script>

<style scoped>
.center {
    margin-left: auto;
    margin-right: auto;
}

</style>