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
      printCharacteristic: null
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
            this.printValue()
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
      await this.sendToPrinter(headerText)
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
      await this.sendToPrinter(footerText + '\n' + '\u000A\u000D')
      return footerText
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
      await this.sendToPrinter(tableText)
      return `\n${tableText}`
    },

    async printValue() {
      const headers = await this.getHeaderText()
      const tables = await this.htmlTableToPlainText()
      const footers = await this.getFooterText()
      let textToPrint = headers + tables + footers
      console.log(textToPrint)
    },

    async sendToPrinter(data) {
      let encoder = new TextEncoder('utf-8')
      const command = encoder.encode(data + '\u000A\u000D')
      return this.printCharacteristic.writeValue(command).then(() => {
        console.log('Write done.')
      })
    },

    async printPage() {
      if (this.printCharacteristic === null) {
        await this.requestBluetoothAccess()
      } else {
        this.printValue()
      }
    }
  }
}
</script>
