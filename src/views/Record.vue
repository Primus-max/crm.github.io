<template>
    <div>
        <div class="page-title">
            <h3>Новая запись</h3>
        </div>

        <Loader v-if="loading"/>

        <p class="center" v-else-if="!categories.length">Категории еще не созданы
            <router-link to="/categories">добавить категорию</router-link>
        </p>

        <form class="form" v-else @submit.prevent="submitHandler">
            <div class="input-field">
                <select ref="select" v-model="category">
                    <option

                            v-for="c of categories"
                            :key="c.id"
                            :value="c.id"
                    >
                        {{ c.title }}
                    </option>
                </select>
                <label>Выберите категорию</label>
            </div>

            <p>
                <label>
                    <input
                            class="with-gap"
                            name="type"
                            type="radio"
                            value="income"
                            v-model="type"
                    />
                    <span>Доход</span>
                </label>
            </p>

            <p>
                <label>
                    <input
                            class="with-gap"
                            name="type"
                            type="radio"
                            value="outcome"
                            v-model="type"
                    />
                    <span>Расход</span>
                </label>
            </p>

            <div class="input-field">
                <input
                        id="amount"
                        type="number"
                        v-model.number="amount"
                        :class="{invalid: v$.amount.$error}"
                >
                <label for="amount">Сумма</label>
                <span class="helper-text invalid" v-if="v$.amount.$error">Введите сумму {{v$.amount.minValue.$params.min}}</span>
            </div>

            <div class="input-field">
                <input
                        id="description"
                        type="text"
                        v-model="description"
                        :class="{invalid: v$.description.$error}"
                >
                <label for="description">Описание</label>
                <span
                        class="helper-text invalid" v-if="v$.description.$error">Введите описание
                </span>
            </div>

            <button class="btn waves-effect waves-light" type="submit">
                Создать
                <i class="material-icons right">send</i>
            </button>
        </form>
    </div>
</template>

<script>
    import {required, minValue} from '@vuelidate/validators'
    import Loader from "@/components/app/Loader"
    import useVuelidate from "@vuelidate/core"
    import {mapGetters} from 'vuex'

    export default {
        name: 'Record',
        components: {Loader},
        setup() {
            return {v$: useVuelidate()}
        },
        validations: () => ({
            amount: {minValue: minValue(1)},
            description: {required},
        }),

        data: () => ({
            loading: true,
            categories: [],
            select: null,
            type: 'outcome',
            amount: 1,
            description: '',
            category: null,
        }),
        async mounted() {
            this.categories = await this.$store.dispatch('fetchCategories')
            this.loading = false


            if (this.categories.length) {
                this.category = this.categories[0].id
            }

            setTimeout(() => {
                this.select = M.FormSelect.init(this.$refs.select);
                M.updateTextFields()
            }, 0)

        },
        computed: {
            ...mapGetters(['info']),
            canCreateRecord() {
                if (this.type === 'income') {
                    return true
                }
                return this.info.bill >= this.amount
            }
        },
        methods: {
            async submitHandler() {
                if (this.v$.$invalid) {
                    this.v$.$touch()
                    return
                }
                if (this.canCreateRecord) {

                    try {
                        await this.$store.dispatch('createRecord', {
                            categoryId: this.category,
                            amount: this.amount,
                            description: this.description,
                            type: this.type,
                            date: new Date().toJSON()
                        })


                        const bill = this.type === 'income'
                            ? this.info.bill + this.amount
                            : this.info.bill - this.amount

                        await this.$store.dispatch('updateInfo', {bill})
                        this.$toast.success(`Запись успешно создана`, {
                            position: "top",
                            duration: 4000,
                        });
                        this.v$.$reset()
                        this.amount = 1
                        this.description = ''

                    } catch (e) {
                    }
                } else {
                    this.$toast.error(`Недостаточно средств на счете - ${this.amount - this.info.bill}`, {
                        position: "top",
                        duration: 4000,
                    });
                }

            },
            unmounted() {
                if (this.select && this.select.unmount) {
                    this.select.unmount()
                }
            }
        }
    }
</script>

<style scoped>

</style>