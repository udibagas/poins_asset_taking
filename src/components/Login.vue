<template>
    <ons-page>
        <div class="background"></div>
        <div class="content">
            <img src="../assets/logo.png"><br>
            <strong>ASSET TAKING</strong><br>
            <div class="form">
                <p> <v-ons-input v-model="email" placeholder="Email" class="full-width"></v-ons-input> </p>
                <p> <v-ons-input type="password" v-model="password" placeholder="Password" class="full-width"></v-ons-input> </p>
                <p><v-ons-button class="login-btn" @click.prevent="login">LOGIN</v-ons-button></p>
                <p class="error" v-if="error">{{error}}</p>

                <br><br>
                <small>&copy; {{year}} | KPP</small>
            </div>
        </div>
    </ons-page>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
import Main from './Main'

const API_URL = 'http://192.168.43.230:8000/api/'

export default {
    data: function() {
        return {
            email: '',
            password: '',
            error: false,
            year: moment().format('YYYY')
        }
    },
    methods: {
        login() {
            if (!this.email || !this.password) {
                return
            }

            // if (navigator.connection.type !== Connection.WIFI) {
            //     this.error = 'Check your WiFi connection!'
            //     return
            // }

            let _this = this
            _this.error = 'Logging in...'

            axios.post(API_URL + 'login', {
                email: _this.email,
                password: _this.password
            }).then(function(r) {
                if (r.data) {
                    _this.$emit('replace-page', {
                        extends: Main,
                        data: function() {
                            return {
                                user: r.data
                            }
                        }
                    })
                } else {
                    _this.error = 'User not found!'
                }
            }).catch(function(e) {
                if (e.response) {
                    if (e.response.status === 500) {
                        _this.error = e.response.data.message
                    }

                    if (e.response.status === 404) {
                        _this.error = 'Page not found!'
                    }
                } else {
                    _this.error = 'Failed to connect to server!'
                }
            })
        }
    }
}
</script>

<style scoped>
.background {
    background-color: #fff;
}

.content {
    text-align: center;
    margin: 50px auto 0;
}

img {
    width: 200px;
    margin: 10px auto;
}

.error {
    color: red;
}

.form {
    margin: 100px auto 10px;
    width: 250px;
}

.login-btn {
    font-size: 20px;
    height: 30px;
    font-weight: normal;
    border-radius: 20px;
    width: 100%;
}

.full-width {
    width: 100%;
}
</style>
