<template>
    <VDatepicker/>
</template>

<script>
    import VDatepicker from './components/VDatepicker.vue'

    export default {
        name: "App",
        components: {
            VDatepicker
        },
        data: function() {
            userName: 'John'
        }
    }
</script>

<style scoped>

</style>