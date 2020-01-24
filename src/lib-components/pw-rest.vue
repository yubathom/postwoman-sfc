<template>
  <section class='pw-rest'>
      <h3>
        <span>{{ method }}</span>
        <a :href="url + path">/{{ setPath }}</a>
      </h3>
      <div>
        <h4>URL</h4>
        <code>
          {{ url }}{{ setPath }}
        </code>
        <h4>Description</h4>
        <p>{{ description }}</p>
        <div>
            <h4>Code example</h4>
            <div>
                <select v-model="selectedRequestType">
                    <option>JavaScript XHR</option>
                    <option>Fetch</option>
                    <option>cURL</option>
                </select>
            </div>
            <div
              ref="req-example"
              :class="{ 
                'pw-rest__language-sh': selectedRequestType === 'cURL',
                'pw-rest__language-js': selectedRequestType === 'Fetch' || selectedRequestType === 'JavaScript XHR',
                }">
                  <pre><code>{{ ui.requestExample }}</code></pre>
              </div>
          </div>
          <button :class="{ 'pw-rest__fetching': api.fetching }" @click.prevent="send(`${url}${path}`, method)">
              <template v-if="api.fetching"><span>.</span><span>.</span><span>.</span></template>
              <template v-else>{{ ui.btnTest }} <span class="pw-rest__req-name">/{{ setPath }}</span></template>
          </button>
          <span :class="{ error: api.error, display: api.message }">{{ api.message }}</span>
      </div>
      <div class="pw-rest__response">
          <div v-if="api.message !== ''" class="pw-rest__api-message">
              <h4>Response</h4>
              <div class="pw-rest__language-json" :class="{ ready: !api.fetching }"><pre class="pw-rest__language-json"><code>{{ api.response }}</code></pre></div>
          </div>
      </div>
  </section>
</template>

<script>
export default {
    name: 'PwRest',
    data() {
        return {
            ui: {
                btnTest: 'Send request to ',
                requestType: 'cURL',
                requestExample: '',
            },
            api: {
                fetching: false,
                response: '',
                message: '',
                error: false
            }
        }
    },
    computed: {
        setPath () { 
            if (this.pathAlias === '') return this.path
            else return this.pathAlias
        },
        selectedRequestType: {
            get () {
                return this.ui.requestType
            },
            set (newReqStr) {
                this.ui.requestType = newReqStr
            }
        }
    },
    watch: {
        selectedRequestType (reqType) {
            this.ui.requestExample = this.getRequestExample(reqType, this.method, `${this.url}${this.path}`)
        }
    },
    created () {
        if (this.requestType !== 'cURL') this.ui.requestType = this.requestType
        this.ui.requestExample = this.getRequestExample(this.requestType, this.method, `${this.url}${this.path}`)
    },
    methods: {
        getRequestExample(reqType, method, reqUrl) {
            if (method === 'GET') {
                switch (reqType) {
                    case 'Fetch': return `fetch('${reqUrl}', {
    method: 'GET',
    credentials: 'same-origin'
    }).then(function(response) {
        response.status
        response.statusText
        response.headers
        response.url
        return response.text()
    }).catch(function(error) {
        error.message
})`
                    case 'JavaScript XHR': return `const xhr = new XMLHttpRequest()
xhr.open('GET', '${reqUrl}', true, null, null)
xhr.send()`
                    default: return `cURL -X GET ${reqUrl}`; break // default is cURL
                }
            }
            else throw new Error (`invalid params reqType:${reqType} or method: ${method} or reqUrl: ${reqUrl}`)
        },
        stringfyRes(jsonStr) {
            return JSON.parse(jsonStr, null, 2)
        },
        send (fullUrl, method) {
            this.api.fetching = true
            this.api.message = 'Sending...'
            const oReq = new XMLHttpRequest()
            oReq.addEventListener("load", evt => {
                this.api.response = this.stringfyRes(evt.target.response)
                this.api.fetching = false
                this.api.message = null
                this.ui.btnTest = 'Test it again'
            })
            oReq.addEventListener("error", evt => {
                this.api.response = evt.target.response
                this.api.fetching = false
                this.api.message = 'Error! The request failed'
            })
            oReq.addEventListener("abort", evt => {
                this.api.fetching = false
                this.api.message = 'Error! The request was cancelled'
            })
            oReq.open(method, fullUrl, true, null, null)
            oReq.send()
        }
    },
    props: {
        description: {
            type: String,
            default: ''
        },
        url: {
            type: String,
            required: true
        },
        path: {
            type: String,
            default: ''
        },
        pathAlias: {
            type: String,
            default: ''
        },
        method: {
            type: String,
            default: 'GET'
        },
        label: {
            type: String,
            default: ''
        },
        auth: {
            type: String,
            validator: function (value) {
                return ['None', 'Basic', 'Bearer Token'].indexOf(value) !== -1
            }
        },
        httpUser: {
            type: String,
            default: ''
        },
        httpPassword:{
            type: String,
            default: ''
        },
        bearerToken: {
            type: String,
            default: ''
        },
        headers: {
            type: Array
        },
        params: {
            type: Array
        },
        bodyParams: {
            type: Array
        },
        rawParams: {
            type: String,
            default: ''
        },
        rawInput: {
            type: String,
            default: ''
        },
        contentType: {
            type: String,
            validator: function (value) {
                return [
                    'application/json',
                    'application/hal+json',
                    'application/xml',
                    'application/x-www-form-urlencoded',
                    'text/html',
                    'text/plain'
                ].indexOf(value) !== -1
            }
        },
        requestType: {
            type: String,
            validator: function (value) {
                return [
                    'JavaScript XHR',
                    'Fetch',
                    'cURL'
                ].indexOf(value) !== -1
            },
            default: 'cURL'
        },
        passwordFieldType: {
            type: String,
            validator: function (value) {
                return [
                    'password',
                    'text'
                ].indexOf(value) !== -1
            }
        }
    }
}
</script>
