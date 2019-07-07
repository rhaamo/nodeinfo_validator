<template>
  <div class="form">
    <form @submit='validate'>
      <label for="url">URL</label>&nbsp;
      <input id="url" v-model="url" type="text" name="url" placeholder="https://foo.example.com/" value="https://pleroma.otter.sh">&nbsp;
      <input type="submit" value="Validate">
      <p v-if="url_error">{{ url_error }}</p>
      <br>
      <p id="validation_output" v-html="validation_checks"></p>
    </form>
  </div>
</template>

<script>
import Ajv from 'ajv'
import endpointSchemaJson from '../schemas/endpoint.json'
import nodeinfo10SchemaJson from '../schemas/1.0.json'
import nodeinfo11SchemaJson from '../schemas/1.1.json'
import nodeinfo20SchemaJson from '../schemas/2.0.json'
import nodeinfo21SchemaJson from '../schemas/2.1.json'
import JSONSchemaDraft4Definition from 'ajv/lib/refs/json-schema-draft-04.json'

export default {
  data: () => ({
    url: '',
    url_error: '',
    validation_checks: ''
  }),
  methods: {
    log: function (msg) {
      this.validation_checks += msg + '<br>'
      console.log(msg)
    },
    fetchJson: function (url) {
      return window.fetch(url, { method: 'GET', headers: { 'Content-Type': 'application/json' }, dataType: 'jsonp' })
        .then((response) => {
          if (response.ok) {
            return response.json()
          } else {
            this.log('Error while fetching ' + url)
          }
        })
        .catch((error) => {
          this.log('Error while fetching ' + url + ': ')
          this.log(error.message)
        })
    },
    validate: async function (e) {
      e.preventDefault()

      this.validation_checks = '' // cleanup

      if (!this.url) {
        this.url_error = 'URL required'
        e.preventDefault()
        return
      }

      let url = this.url
      if (!url.endsWith('/')) {
        url += '/'
      }

      this.log('Validating Nodeinfo for: ' + url)
      this.validation_checks += '<br>'

      // Load nodeinfo endpoint

      let endpointJson = null
      const endpointUrl = url + '.well-known/nodeinfo'
      try {
        endpointJson = await this.fetchJson(endpointUrl)
      } catch (e) {
        this.log('Network error: ' + e)
      }
      console.log(endpointJson)

      if (!endpointJson) { e.preventDefault(); return }

      // Validate endpoint
      const ajv = new Ajv({
        schemaId: 'id', // draft-04 support requirment.
        allErrors: false
      })
      ajv.addMetaSchema(JSONSchemaDraft4Definition) // Nodeinfo uses draft-04

      ajv.addSchema(endpointSchemaJson, 'endpoint')
        .addSchema(nodeinfo10SchemaJson, '1.0')
        .addSchema(nodeinfo11SchemaJson, '1.1')
        .addSchema(nodeinfo20SchemaJson, '2.0')
        .addSchema(nodeinfo21SchemaJson, '2.1')

      console.log(endpointJson)
      const endpointValid = ajv.validate('endpoint', endpointJson)
      this.log('Schema validation result for the well-known endpoint:')
      if (!endpointValid) {
        this.log('Endpoint validation failed')
        // endpointValid.errors.forEach(err => this.log(err))
      } else {
        this.log('No error, endpoint valid')
      }
      this.validation_checks += '<br>'

      // Now loop over each nodeinfo schemas
      for (var i = 0; i < endpointJson.links.length; i++) {
        let item = endpointJson.links[i]

        let url = item.href
        let version = item.rel.split('/').pop()
        this.log('Validating nodeinfo ' + version + ': ' + url)

        let nodeinfoJson = null
        try {
          nodeinfoJson = await this.fetchJson(url)
        } catch (e) {
          this.log('Network error: ' + e)
        }

        if (!nodeinfoJson) { e.preventDefault(); return }

        const nodeinfoValid = ajv.validate(version, nodeinfoJson)
        this.log('Schema validation result for the Nodeinfo ' + version + ' schema:')
        if (!nodeinfoValid) {
          this.log('Nodeinfo ' + version + ' valiation failed')
          console.log(nodeinfoValid)
        } else {
          this.log('Nodeinfo ' + version + ' is valid.')
        }
        this.validation_checks += '<br>'
      }

      this.validation_checks += 'Validation finished'
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
p#validation_output {
  text-align: left;
  width: 50%;
  margin-left: auto;
  margin-right: auto;
}
</style>
