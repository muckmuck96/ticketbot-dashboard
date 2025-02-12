<script context="module">
    import axios from 'axios';
    import {OAUTH} from "../js/constants";

    const _tokenKey = 'token';

    export function getToken() {
        let token = window.localStorage.getItem(_tokenKey);
        if (token == null) {
            redirectLogin();
            return;
        }

        return token;
    }

    export function setToken(token) {
        window.localStorage.setItem(_tokenKey, token);
    }

    export function redirectLogin() {
        let state = btoa(new URL(window.location.href).pathname);
        window.location.href = `https://discord.com/oauth2/authorize?client_id=1332749420026007652&response_type=code&redirect_uri=https%3A%2F%2Fdashboard.ticket.emotionv.de%2Fcallback&scope=identify+guilds&state=${state}`;
    }

    export function clearLocalStorage() {
        window.localStorage.clear();
    }

    export function setDefaultHeaders() {
        axios.defaults.headers.common['Authorization'] = getToken();
        axios.defaults.headers.common['x-tickets'] = 'true'; // arbitrary header name and value
        axios.defaults.validateStatus = (s) => true;

        addRefreshInterceptor();
    }

    function addRefreshInterceptor() {
        axios.interceptors.response.use(async (res) => { // we set validateStatus to false
            if (res.status === 401) {
                redirectLogin();
            }
            return res;
        }, async (err) => {
            if (err.response.status === 401) {
                redirectLogin();
            }
            return err.response;
        });
    }
</script>
