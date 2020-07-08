<template>
    <div id="app">
        <div class="container" ref="slideContainer">
            <div class="slide-wrapper" ref="slideWrapper">
                <div class="slide login" ref="slideLogin">
                    <input type="text" v-model="username" class="login__input login__input--user t-input" placeholder="Username">
                    <input type="password" v-model="password" class="login__input login__input-password t-input" placeholder="Password">
                    <button @click="loginUser" class="button login__button" :class="{ 'login__button--active': username !== '' && password !== '' }">Login</button>
                    <span class="login__message t-error">{{ loginErrorMessage }}</span>
                </div>
                <div class="slide list">
                    <span v-if="loadingList">Loading Cameras...</span>
                    <div class="list__container" v-else>
                        <div class="list__item" v-for="(item, i) in cameraList" :key="i" @click="retrieveSingleCamera(item.cameraId)">
                            <span class="list__item-number">{{ i + 1 }}</span>
                            <span class="list__item-name">{{ item.name }}</span>
                            <span class="list__item-id t-id">ID: {{ item.cameraId }}</span>
                        </div>
                    </div>
                </div>
                <div class="slide details">
                    <button @click="slidePanel(1)" class="button details__button t-back">Go back</button>
                    <span v-if="loadingDetails">Loading Camera Details...</span>
                    <div class="details__container t-details">
                        <div class="details__list" v-if="cameraDetails">
                            <span>Account ID: {{ cameraDetails.accountId }}</span>
                            <span>Camera ID: {{ cameraDetails.cameraId }}</span>
                            <span>Device Type ID: {{ cameraDetails.deviceTypeId }}</span>
                            <span>Ethernet Address: {{ cameraDetails.ethMacAddress }}</span>
                            <span>Name: {{ cameraDetails.name }}</span>
                            <span>Zone ID: {{ cameraDetails.zoneId }}</span>
                        </div>
                    </div>
                </div>
            </div>
            <transition name="fade">
                <span class="logout-progress t-logout" v-if="loggingOut">Logging out...</span>
                <button class="logout-button t-logout" v-else-if="loggedIn" @click="logOutUser">Log Out</button>
            </transition>
            <transition name="fade">
                <div class="axios-error" v-if="axiosError">{{ axiosError }}</div>
            </transition>
        </div>
    </div>
</template>

<script>

import axios from 'vue-cli-plugin-axios'
import { gsap } from 'gsap'

export default {
    name: 'App',

    data: () => {
        return {
            clientId: process.env.VUE_APP_CLIENT_ID,
            clientSecret: process.env.VUE_APP_CLIENT_SECRET,
            username: '',
            password: '',
            loginErrorMessage: '',
            accessToken: null,
            loadingList: false,
            loadingDetails: false,
            cameraList: null,
            cameraDetails: null,
            axiosError: null,
            loggedIn: false,
            loggingOut: false
        }
    },

    mounted () {
        new gsap.timeline()
        .set(this.$refs.slideContainer, {
            scale: 1.2
        })
        .set(this.$refs.slideLogin, {
            opacity: 0,
            y: 15
        })
        .to(this.$refs.slideContainer, 0.2, {
            scale: 1
        }, 'a')
        .to(this.$refs.slideLogin, 0.4, {
            opacity: 1,
            y: 1
        }, 'a+=.1')
    },

    methods: {

        loginUser () { 
            this.loginErrorMessage = '';

            const url = `https://rest.cameramanager.com/oauth/token?grant_type=password&scope=read&username=${this.username}&password=${this.password}&client_id=${this.clientId}&client_secret=${this.clientSecret}`;

            this.makeAxiosRequest('post', url, response => {
                if (response && response.data) {
                    this.loggedIn = true;
                    this.accessToken = response.data.access_token
                    this.retrieveAccountCameras();
                }
            }, error => {
                this.loginErrorMessage = 'Error Logging In';
            });
        },

        slidePanel (showPanel) {
            const slideAmount = -(100 / 3) * showPanel;

            gsap.to(this.$refs.slideWrapper, 0.65, {
                x: `${slideAmount}%`,
                ease: 'Power3.easeInOut'
            });
        },

        retrieveAccountCameras () {
            this.loadingList = true;
            this.slidePanel(1);

            const url = 'https://rest.cameramanager.com/rest/v2.4/cameras';

            this.makeAxiosRequest('get', url, response => {
                if (response && response.data) this.cameraList = response.data;
                this.loadingList = false;
            });
        },

        retrieveSingleCamera (cameraId) {
            this.loadingDetails = true;
            this.slidePanel(2);

            const url = `https://rest.cameramanager.com/rest/v2.4/cameras/${cameraId}`;

            this.makeAxiosRequest('get', url, response => {
                if (response && response.data) this.cameraDetails = response.data;
                this.loadingDetails = false;
            });
        },

        logOutUser () {
            this.loggingOut = true;
            const url = 'https://rest.cameramanager.com/rest/v2.0/users/self/tokens/current';

            this.makeAxiosRequest('delete', url, response => {
                this.slidePanel(0);
                this.loggedIn = false;
                this.loggingOut = false;
            })
        },

        makeAxiosRequest (type, url, callback, errorCallback) {
            this.axiosError = null;

            this.axios[type](url, {
                crossDomain: true,
                headers: {
                    Authorization: `Bearer ${this.accessToken}`,
                    Accept: 'application/json'
                }
            })
            .then(response => {
                callback(response);
            })
            .catch(error => {
                if (errorCallback) errorCallback(error);
                else this.axiosError = 'Error making the request';
            })
        }
    }
}
</script>

<style lang="stylus">
    @import "assets/stylus/app.styl"
</style>
