<template>
    <v-ons-page>
        <div class="background"></div>
        <div class="content">
            <img src="../assets/logo.png"><br>
            <strong>ASSET TAKING</strong><br>
            <div class="form">
                <p><input type="text" v-model="reg_no" class="po-number-input" placeholder="Reg. Number"></p>
                <p><v-ons-button :disabled="!reg_no" @click.prevent="reg_no = ''" class="submit-data-btn" modifier="material">CLEAR</v-ons-button></p>
                <p><v-ons-button @click.prevent="scan" class="scan-qr-code-btn">SCAN QR CODE</v-ons-button></p>
                <p><v-ons-button :disabled="!reg_no" @click.prevent="submitData" class="submit-data-btn" modifier="material">SUBMIT</v-ons-button></p>
                <p class="error" v-if="error">{{error}}</p>
            </div>
        </div>

        <v-ons-dialog :visible.sync="alert.show">
            <div style="text-align:center;padding:15px;">
                <p v-if="alert.title"><strong>{{alert.title}}</strong></p>
                <p>{{alert.message}}</p>
                <br>
                <p><v-ons-button class="full-width btn-round" @click.prevent="alert.show = false">OK</v-ons-button></p>
            </div>
        </v-ons-dialog>

        <v-ons-bottom-toolbar>
            <v-ons-row>
                <v-ons-col style="text-align:center">
                    <v-ons-button modifier="material--flat" @click.prevent="exit">EXIT APP</v-ons-button>
                </v-ons-col>
                <v-ons-col style="text-align:center">
                    <v-ons-button modifier="material--flat" @click.prevent="logout">LOGOUT</v-ons-button>
                </v-ons-col>
            </v-ons-row>
        </v-ons-bottom-toolbar>
    </v-ons-page>
</template>

<script>
import axios from 'axios'
import Login from './Login'
import AssetDetail from './AssetDetail'

const API_URL = 'http://192.168.43.230:8000/api/'

export default {
    data: function() {
        return {
            user: {},
            is_ready: false,
            reg_no: '',
            error: '',
            alert: {
                title: '',
                show: false,
                message: ''
            }
        }
    },
    methods: {
        logout: function() {
            let _this = this
            this.$ons.notification.confirm({
                title: '',
                message: 'Are you sure?'
            }).then(function(r) {
                if (r === 1) {
                    _this.$emit('replace-page', Login);
                }
            })
        },
        exit: function() {
            this.$ons.notification.confirm({
                title: '',
                message: 'Are you sure you want to quit?'
            }).then(function(r) {
                if (r === 1) {
                    navigator.app.exitApp()
                }
            })
        },
        scan: function() {
            let _this = this
            _this.$ons.ready(function() {
                cordova.plugins.barcodeScanner.scan(
                    function (result) {
                        _this.reg_no = result.text
                        if (_this.reg_no) {
                            _this.submitData()
                        }
                    },
                    function (error) {
                        alert("Scanning failed: " + error);
                    },
                    {
                        preferFrontCamera : false,
                        showFlipCameraButton : true,
                        showTorchButton : true,
                        torchOn: false,
                        saveHistory: true,
                        prompt : "Place a barcode inside the scan area",
                        resultDisplayDuration: 500,
                        formats : "QR_CODE,PDF_417",
                        orientation : "portrait",
                        disableSuccessBeep: false
                    }
                );
            })
        },
        submitData: function() {
            let _this = this
            axios.get(API_URL + 'asset', {params: {reg_no: _this.reg_no}})
                .then(function(r) {
                    if (r.data.status === 'success') {
                        if (r.data.asset) {
                            _this.$emit('push-page', {
                                extends: AssetDetail,
                                data: function() {
                                    return {
                                        asset: r.data.asset,
                                        user: _this.user
                                    }
                                }
                            })
                        } else {
                            _this.alert = {
                                show: true,
                                title: 'ASSET NOT FOUND',
                                message: 'Tidak ada asset dengan nomor registrasi ' + _this.reg_no
                            }
                        }
                    } else {
                        _this.alert = {
                            show: true,
                            title: 'ERROR',
                            message: r.data.message
                        }
                    }
                })
                .catch(function(e) {
                    if (e.response) {
                        if (e.response.status === 500) {
                            _this.alert = {
                                show: true,
                                title: 'ERROR 500',
                                message: e.response.data.message
                            }
                        }

                        if (e.response.status === 404) {
                            _this.alert = {
                                show: true,
                                title: 'ERROR 404',
                                message: 'Asset tidak ditemukan'
                            }
                        }
                    }
                })
        }
    }
}
</script>

<style scoped>
.background {
    background-color: white;
}

.content {
    text-align: center;
    margin: 50px auto 0;
}

.form {
    margin: 100px auto 10px;
    width: 270px;
}

img {
    width: 200px;
    margin: 10px auto;
}

.po-number-input {
    width: 100%;
    font-size: 32px;
    color: red;
    background-color: #eee;
    text-align: center;
    border: 1px solid #eee;
}

.full-width {
    display: block;
    width: 100%;
    margin-bottom: 5px;
}

.btn-round {
    border-radius: 20px;
}

.submit-data-btn, .scan-qr-code-btn {
    font-size: 20px;
    height: 30px;
    font-weight: normal;
    border-radius: 20px;
    width: 100%;
}

::-webkit-input-placeholder {
   text-align: center;
   font-weight: lighter;
}

:-moz-placeholder { /* Firefox 18- */
   text-align: center;
   font-weight: lighter;
}

::-moz-placeholder {  /* Firefox 19+ */
   text-align: center;
   font-weight: lighter;
}

:-ms-input-placeholder {
   text-align: center;
   font-weight: lighter;
}
</style>
