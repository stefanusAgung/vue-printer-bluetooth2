<template>
  <button @click.prevent="printPage">Blueetoth Print</button>
</template>
<script>
export default {
  name: 'button-print-bluetooth',
  props: {
    printArea: { type: Object, default: null, required: true }
  },
  data() {
    return {
      printCharacteristic: null,
      indexData: 0,
      dataPrint: null
    }
  },
  methods: {
    async requestBluetoothAccess() {
      try {
        navigator.bluetooth
          .requestDevice({
            filters: [
              {
                services: ['000018f0-0000-1000-8000-00805f9b34fb']
              }
            ]
          })
          .then((device) => {
            console.log('> Found ' + device.name)
            console.log('Connecting to GATT Server...')
            return device.gatt.connect()
          })
          .then((server) => server.getPrimaryService('000018f0-0000-1000-8000-00805f9b34fb'))
          .then((service) => service.getCharacteristic('00002af1-0000-1000-8000-00805f9b34fb'))
          .then(async (characteristic) => {
            this.printCharacteristic = characteristic
            this.sendPrintData()
          })
          .catch(this.handleError)
      } catch (error) {
        console.log('Argh! ' + error)
      }
    },

    handleError(error) {
      console.log(error)
      this.printCharacteristic = null
    },

    async getHeaderText() {
      const section = this.printArea
      let headers = section.querySelectorAll('.print-header')
      let headerText = ''
      headers.forEach(async (dt) => {
        if (dt.innerText.toUpperCase() === 'ITEM') {
          headerText += '<tableSlot>'
          headers = null
        } else {
          headerText += dt.innerText + '\n'
        }
      })
      return headerText
    },

    async getFooterText() {
      const section = this.printArea
      let footers = section.querySelectorAll('.print-footer')
      let footerText = ''
      footers.forEach(async (dt) => {
        console.log(dt)
        if (dt.innerText.toUpperCase() === 'ITEM') {
          footerText += '<tableSlot>'
          footers = null
        } else {
          footerText += dt.innerText + '\n'
        }
      })
      return footerText + '\n\n'
    },

    async htmlTableToPlainText() {
      const section = this.printArea
      let line = '---------------------------------------------' + '\n'
      const tableRows = section.querySelectorAll('tr')
      let tableText = 'ITEM                                    TOTAL' + '\n\n'

      // Loop through table rows and extract text
      tableRows.forEach(async (row) => {
        const cells = row.querySelectorAll('th, td')
        const rowText = Array.from(cells).map((cell) => cell.textContent.trim())
        if (rowText[1].toUpperCase() !== 'AMOUNT') {
          let length1 = rowText[0].length
          let length2 = rowText[3].length
          let fillGap = ''
          let fixLength = 45 - length1 - length2
          for (let i = 0; i < fixLength; i++) {
            fillGap += ' '
          }
          tableText +=
            rowText[0] +
            fillGap +
            rowText[3] +
            '\n' +
            rowText[1] +
            ' * ' +
            rowText[2] +
            '\n' +
            line +
            '\n'
        }
      })
      return `\n${tableText}`
    },

    sendNextDataBatch() {
      // Can only write 512 bytes at a time to the characteristic
      // Need to send the image data in 512 byte batches
      if (this.indexData + 512 < this.dataPrint.length) {
        let dt = this.dataPrint.slice(this.indexData, this.indexData + 512)
        console.log(dt)
        this.printCharacteristic.writeValue(dt).then(() => {
          this.indexData += 512
          this.sendNextDataBatch()
        })
      } else {
        // Send the last bytes
        console.log(this.indexData)
        console.log(this.dataPrint.length)

        if (this.indexData < this.dataPrint.length) {
          let dt = this.dataPrint.slice(this.indexData, this.dataPrint.length)
          this.printCharacteristic.writeValue(dt)
        } else {
          //resolve()
        }
      }
    },

    async sendPrintData() {
      this.indexData = 0
      let encoder = new TextEncoder('utf-8')

      const headers = await this.getHeaderText()
      const tables = await this.htmlTableToPlainText()
      const footers = await this.getFooterText()
      const command = encoder.encode(headers + tables + footers + '\u000A\u000D')
      this.dataPrint = command
      this.sendNextDataBatch()
    },

    async printPage() {
      if (this.printCharacteristic == null) {
        await this.requestBluetoothAccess()
      } else {
        this.sendPrintData()
      }
    }
  }
}
</script>
