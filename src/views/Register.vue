<template>
    <form class="card auth-card" @submit.prevent="submitHandler">
        <div class="card-content">
            <span class="card-title">Домашняя бухгалтерия</span>
            <div class="input-field">
                <input
                        id="email"
                        type="text"
                        v-model="email"
                        :class="{invalid: v$.email.$error}"
                >
                <label for="email">Email</label>
                <small class="helper-text invalid"
                       v-for="(error, index) of v$.email.$errors">
                    {{ printError(error.$validator, error.$params) }}
                </small>
            </div>
            <div class="input-field">
                <input
                        id="password"
                        type="password"
                        v-model="password"
                        :class="{invalid: v$.password.$error}"
                >
                <label for="password">Пароль</label>
                <small class="helper-text invalid"
                       v-for="(error, index) of v$.password.$errors"
                >
                    {{ printError(error.$validator, error.$params) }}
                </small>
            </div>
            <div class="input-field">
                <input
                        id="name"
                        type="text"
                        v-model="name"
                        :class="{invalid: v$.name.$error}"
                >
                <label for="name">Имя</label>
                <small class="helper-text invalid"
                       v-for="(error, index) of v$.name.$errors"
                >
                    {{ printError(error.$validator, error.$params) }}
                </small>
            </div>
            <p>
                <label>
                    <input type="checkbox" v-model="agree"/>
                    <span>С правилами согласен</span>
                </label>
            </p>
        </div>
        <div class="card-action">
            <div>
                <button
                        class="btn waves-effect waves-light auth-submit"
                        type="submit"
                >
                    Зарегистрироваться
                    <i class="material-icons right">send</i>
                </button>
            </div>

            <p class="center">
                Уже есть аккаунт?
                <router-link to="/login">Войти!</router-link>
            </p>
        </div>
    </form>
</template>

<script>
    import useVuelidate from "@vuelidate/core";
    import {email, minLength, required} from "@vuelidate/validators";

    export default {
        name: 'Register',
        setup() {
            return {v$: useVuelidate()}
        },
        data: () => ({
            email: '',
            password: '',
            name: '',
            agree: false,
        }),
        validations: () => ({
            email: {email, required},
            password: {required, minLength: minLength(6)},
            name: {required},
            agree: {checked: v => v}
        }),
        methods: {
            async submitHandler() {
                this.v$.$touch()
                if (this.v$.$error) return

                const formData = {
                    email: this.email,
                    password: this.password,
                    name: this.name
                }
               try {
                   await this.$store.dispatch('register', formData)
                   this.$router.push('/')
                   this.$toast.success("Вы успешно зарагестрировались!", {
                       position: "top-right",
                       duration: 3000,
                   });
               } catch(e){
                    console.log(e)
               }


            },

            printError($name, $param) {
                console.log($name)

                if ($name === 'required') {
                    return 'Поле не должно быть пустым'
                } else if ($name === 'minLength') {
                    console.log($param)
                    return `Минимальная длина должна быть ${$param.min} символов, сейчас он ${this.password.length}`
                } else if ($name === 'email') {
                    return 'Почта указана не полностью, проверьте'
                }
            }
        }

    }
</script>

<style scoped>

</style>