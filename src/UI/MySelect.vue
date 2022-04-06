<template>
    <select class="wrapper" v-model="modelValue" @change="changeOption">
        <option disabled value="">Выберите из списка</option>
        <option 
        v-for="option in options" 
        :key="option.value" 
        :value="option.value"
        >
        {{ option.name }}
        </option>
    </select>
</template>

<script>
    export default {
        name: 'my-select',
        props: {
            modelValue: {
                type: String,
            },
            options: {
                type: Array,
                default: () => []
            }
        },
        methods: {
            changeOption(event) {
                this.$emit('update:modelValue', event.target.value)
            }
        }
    }
</script>

<style scoped>
.wrapper {
    background-color: rgb(197, 203, 211);
    border: 1px solid lightslategrey;
    margin-bottom: 1vh;
    height: 4vh;
}
</style>