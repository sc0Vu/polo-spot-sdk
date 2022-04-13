

# NodeJS Example

```javascript

'use strict'

const url = 'https://sand-spot-api-gateway.poloniex.com'
const apiKey = "api_key"
const secretKey = "secret_key"

const axios = require('axios')
const CryptoJS = require('crypto-js')
let timestamp = new Date().getTime()

const paramUtils = {
    values: [],
    put(k, v) {
        let value = encodeURIComponent(v)
        this.values.push(k + "=" + value)
    },
    sortedValues() {
        return this.values.sort()
    },
    addGetParams(params) {
        Object.keys(params).forEach(k => {
            this.put(k, params[k])
        })
        this.sortedValues()
    },
    getParams(requestMethod, param) {
        if (requestMethod === 'GET') {
            this.put("signTimestamp", timestamp)
            this.addGetParams(param)
            return this.values.join("&").toString()
        } else if (requestMethod === 'POST' || requestMethod === 'PUT' || requestMethod === 'DELETE') {
            return "requestBody=" + JSON.stringify(param) + "&signTimestamp=" + timestamp
        }
    }
}

class Sign {
    constructor(method, path, param, secretKey) {
        this.method = method
        this.path = path
        this.param = param
        this.secretKey = secretKey
    }

    sign() {
        let paramValue = paramUtils.getParams(this.method, this.param)
        let payload = this.method.toUpperCase() + "\n" + this.path + "\n" + paramValue
        console.log("payload:" + payload)

        let hmacData = CryptoJS.HmacSHA256(payload, this.secretKey);
        return CryptoJS.enc.Base64.stringify(hmacData);
    }
}


function getHeader(method, path, param) {
    const sign = new Sign(method, path, param, secretKey).sign()
    console.log(`signature:${sign}`)
    return {
        "Content-Type": "application/json",
        "key": apiKey,
        "signature": sign,
        "signTimestamp": timestamp
    }
}

function get(url, path, param = {}) {
    const headers = getHeader('GET', path, param)
    return axios.get(url + path, {params: param, headers: headers})
        .then(res => console.log(res.data))
        .catch(e => console.error(e))
}

function post(url, path, param = {}) {
    const headers = getHeader('POST', path, param)
    return axios.post(url + path, param, {headers: headers})
        .then(res => console.log(res.data))
        .catch(e => console.error(e))
}

const orderData2 = {
    "symbol": "link_usdt",
    "accountType": "spot",
    "type": "limit",
    "side": "buy",
    "timeInForce": "GTC",
    "price": "1.1",
    "amount": "1",
    "quantity": "1",
    "clientOrderId": "",
}

// Place Order
post(url, '/v2/orders', orderData2)

// getOrders
get(url, '/v2/orders', {limit: 10})

```