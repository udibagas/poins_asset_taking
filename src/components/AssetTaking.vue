<template lang="html">
    <v-ons-page>
        <v-ons-toolbar>
            <div class="left">
                <v-ons-back-button></v-ons-back-button>
            </div>
            <div class="center">ASSET TAKING #{{asset.reg_no}}</div>
        </v-ons-toolbar>
        <div class="background"></div>
        <div class="content">
            <p>
                <small>New Location</small><br>
                <select v-model="formData.new_asset_location_id" placeholder="New Location">
                    <option v-for="l in assetLocations" :value="l.id" :key="l.id">{{l.name}}</option>
                </select>
                <p class="error" v-if="validationErrors.new_asset_location_id">{{validationErrors.new_asset_location_id[0]}}</p>
            </p>
            <p>
                <small>New Status</small><br>
                <select v-model="formData.new_asset_status_id" placeholder="New Location">
                    <option v-for="s in assetStatuses" :value="s.id" :key="s.id">{{s.code}} - {{s.description}}</option>
                </select>
                <p class="error" v-if="validationErrors.new_asset_status_id">{{validationErrors.new_asset_status_id[0]}}</p>
            </p>
            <p>
                <small>Note</small><br>
                <textarea v-model="formData.note" rows="5" placeholder="Note"></textarea>
                <p class="error" v-if="validationErrors.note">{{validationErrors.note[0]}}</p>
            </p>
            <p>
                <v-ons-button class="full-width btn-round" @click.prevent="saveData">SIMPAN</v-ons-button>
            </p>
        </div>

        <v-ons-dialog :visible.sync="alert.show">
            <div style="text-align:center;padding:15px;">
                <p v-if="alert.title"><strong>{{alert.title}}</strong></p>
                <p>{{alert.message}}</p>
                <br>
                <p><v-ons-button class="full-width btn-round" @click.prevent="alert.show = false">OK</v-ons-button></p>
            </div>
        </v-ons-dialog>
    </v-ons-page>
</template>

<script>
import axios from 'axios'
import Main from './Main'

const API_URL = 'http://192.168.43.230:8000/api/'

export default {
    data: function() {
        return {
            user: {},
            formData: {},
            assetLocations: [],
            assetStatuses: [],
            validationErrors: {},
            alert: {
                show: false,
                title: '',
                message: ''
            }
        }
    },
    mounted: function() {
        this.getAssetLocation()
        this.getAssetStatus()
    },
    methods: {
        getAssetLocation: function() {
            let _this = this
            axios.get(API_URL + 'assetLocation').then(function(r) {
                _this.assetLocations = r.data
            }).catch(function(e) {
                alert(JSON.stringify(e));
            })
        },
        getAssetStatus: function() {
            let _this = this
            axios.get(API_URL + 'assetStatus').then(function(r) {
                _this.assetStatuses = r.data
            }).catch(function(e) {
                alert(JSON.stringify(e));
            })
        },
        saveData: function() {
            let _this = this
            axios.post(API_URL + 'assetTaking', _this.formData)
                .then(function(r) {
                    _this.$emit('replace-page', {
                        extends: Main,
                        data: function() {
                            return {
                                user: _this.user,
                                alert: {
                                    show: true,
                                    title: 'ASSET TAKING COMPLETED',
                                    message: 'Data telah disimpan!'
                                }
                            }
                        }
                    });
                })
                .catch(function(e) {
                    if (e.response) {
                        if (e.response.status === 500) {
                            _this.validationErrors = {};
                            _this.alert = {
                                show: true,
                                title: 'ERROR 500',
                                message: e.response.data.message
                            }
                        }

                        if (e.response.status === 422) {
                            _this.validationErrors = e.response.data.errors
                            _this.alert = {
                                show: true,
                                title: 'VALIDATION ERROR',
                                message: 'Please fill form correctly!'
                            }
                        }

                        if (e.response.status === 404) {
                            _this.validationErrors = {};
                            _this.alert = {
                                show: true,
                                title: 'ERROR 404',
                                message: 'Page not found!'
                            }
                        }
                    } else {
                        _this.alert = {
                            show: true,
                            title: 'ERROR',
                            message: 'Failed to connect to server!'
                        }
                    }
                })
        }
    }
}
</script>

<style lang="css" scoped>
.background {
    background-color: white;
}

.content {
    padding: 20px;
}

.btn-save {
    display: block;
    width: 100%;
    font-size: 20px;
    height: 30px;
    font-weight: normal;
    border-radius: 20px;
}

.full-width {
    display: block;
    width: 100%;
}

.btn-round {
    border-radius: 20px;
}

.error {
    color: red;
    font-size: 11px;
    margin: 2px 0;
}

.has-error {
    background-color:#f2dede;
}

textarea, select {
    width: 100%;
    background-color: #eee;
    border: 1px solid #eee;
    font-size: 16px;
}
</style>
