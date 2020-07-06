<template>
    <div id="app">
        <div class="container">
            <div class="slide-wrapper" ref="slideWrapper">
                <div class="slide login">
                    <input type="text" v-model="username" class="login__input login__input--user t-input" placeholder="Username">
                    <input type="password" v-model="password" class="login__input login__input-password t-input" placeholder="Password">
                    <button @click="loginUser" class="button login__button t-text" :class="{ 'login__button--active': username !== '' && password !== '' }">Login</button>
                    <span class="login__message">{{ loginMessage }}</span>
                </div>
                <div class="slide list">
                    <span class="t-text" v-if="loadingList">Loading Cameras...</span>
                    <div class="list__container" v-if="!loadingList">
                        Camera list
                    </div>
                </div>
                <div class="slide details">
                    <span class="t-text" v-if="loadingList">Loading Cameras...</span>
                    <div class="list__container" v-if="!loadingList">
                        Camera list
                    </div>
                </div>
            </div>
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
            username: '',
            password: '',
            loginMessage: '',
            accessToken: null,
            loadingList: false,
            loadingDetails: false,
            cameraList: null,
            cameraDetails: null,
            selectedCameraId: null
        }
    },

    methods: {
        loginUser () {
            // const url = 'https://rest.cameramanager.com/oauth/token';
            let clientId = 'dev_test';
            let clientSecret = '3H1Bf6mCctIgpCuzvrnyekf3VhAUEnKJ';

            const url = `https://rest.cameramanager.com/oauth/token?grant_type=password&scope=read&username=onlinedemo@cameramanager.com&password=demo1234&client_id=${clientId}&client_secret=${clientSecret}`;

            this.axios.post(url, {})
            .then(response => {
                if(response && response.data) {
                    this.accessToken = response.data.access_token

                    this.loginMessage = loggedIn;

                    this.slidePanel(1);
                    this.retrieveAccountCameras();
                }
            })
            .catch(error => {
                debugger;
                this.loginMessage = 'Error Logging In';
            })
        },

        slidePanel(showPanel) {

        },

        retrieveAccountCameras () {
            // Works
            this.loadingList = true;

            const url = "https://rest.cameramanager.com/rest/v2.4/cameras";

            this.axios.get(url, {
                headers: {
                    Authorization: `Bearer ${this.accessToken}`
                }
            })
            .then(response => {
                debugger;
                this.loadingList = false;
            })
            .catch(error => {
                debugger;
            })
        },

        makeAxiosRequest (url, callback) {

        },

        retrieveSingleCameraInfo () {

        },

        goBackToList () {

        }
    }
}
</script>

<style lang="stylus">
    @import "assets/stylus/app.styl"
</style>
