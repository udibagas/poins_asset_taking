<template lang="html">
    <v-ons-page>
        <v-ons-toolbar>
            <div class="left">
                <v-ons-back-button></v-ons-back-button>
            </div>
            <div class="center">ASSET DETAIL</div>
        </v-ons-toolbar>
        <div class="background"></div>
        <v-ons-list>
            <v-ons-list-item>
                <small class="label">Nomor Registrasi</small>
                <span class="list-item__title">{{asset.reg_no}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Name</small>
                <span class="list-item__title">{{asset.name}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Trademark</small>
                <span class="list-item__title">{{asset.trademark}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Version</small>
                <span class="list-item__title">{{asset.version}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">SN</small>
                <span class="list-item__title">{{asset.sn}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Lifetime</small>
                <span class="list-item__title">{{asset.lifetime}} Tahun</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Price</small>
                <span class="list-item__title">{{asset.price | formatNumber}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Year</small>
                <span class="list-item__title">{{asset.year}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Location</small>
                <span class="list-item__title">{{asset.location}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Status</small>
                <span class="list-item__title">{{asset.stts}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <small class="label">Last Update</small>
                <span class="list-item__title">{{asset.updated_at | readableDateTime}}</span>
            </v-ons-list-item>
            <v-ons-list-item>
                <v-ons-button class="full-width btn-round" @click.prevent="assetTaking">Asset Taking</v-ons-button>
            </v-ons-list-item>
        </v-ons-list>
    </v-ons-page>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
import AssetTaking from './AssetTaking'

const API_URL = 'http://192.168.43.230:8000/api/'

export default {
    data: function() {
        return {
            asset: {},
            user: {},
        }
    },
    filters: {
        readableDateTime: function(v) {
            return moment(v).format('DD-MMM-YYYY HH:mm')
        },
        formatNumber: function(v) {
            return parseFloat(v)
                .toFixed(0)
                .toString()
                .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
        }
    },
    methods: {
        assetTaking: function() {
            let _this = this
            _this.$emit('push-page', {
                extends: AssetTaking,
                data: function() {
                    return {
                        asset: _this.asset,
                        user: _this.user,
                        formData: {
                            date: moment().format('YYYY-MM-DD'),
                            asset_id: _this.asset.id,
                            user_id: _this.user.id,
                            old_asset_location_id: _this.asset.asset_location_id,
                            new_asset_location_id: _this.asset.asset_location_id,
                            old_asset_status_id: _this.asset.asset_status_id,
                            new_asset_status_id: _this.asset.asset_status_id
                        }
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

.success {
    color: green;
}

.danger {
    color: red;
}

.full-width {
    display: block;
    width: 100%;
    margin-bottom: 5px;
}

.btn-round {
    border-radius: 20px;
}

small.label {
    display: block;
}

.list-item__title {
    font-weight: bold;
}
</style>
