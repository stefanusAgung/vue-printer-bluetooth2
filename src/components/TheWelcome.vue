<script>
import ButtonPrintBluetooth from '@/components/ButtonPrintBluetooth.vue'

export default {
  components: {
    ButtonPrintBluetooth
  },
  data() {
    return {
      device: {
        id: null,
        name: null
      },
      printArea: null,
      deviceLists: [],
      printCharacteristic: null,
      myDescriptor: null,
      printAreaId: 'print-area',
      transactionData: {
        trx_code: 'JB0001',
        trx_total: '1000000',
        items: [
          {
            product_code: 'USD',
            product_name: 'United Stated Dollar',
            rate: '10000',
            amount: '10',
            total: '100000'
          },
          {
            product_code: 'SGD',
            product_name: 'Singaporean Dollar',
            rate: '9000',
            amount: '10',
            total: '90000'
          },
          {
            product_code: 'MYR',
            product_name: 'Malaysian Ringgit',
            rate: '3000',
            amount: '200',
            total: '600000'
          },
          {
            product_code: 'GBP',
            product_name: 'British Pound',
            rate: '15000',
            amount: '10',
            total: '150000'
          }
        ]
      }
    }
  },
  mounted() {
    this.printArea = document.getElementById('print-area')
  },
  methods: {
    async populateBluetoothDevices() {
      const devicesSelect = document.querySelector('#devicesSelect')
      try {
        console.log('Getting existing permitted Bluetooth devices...')
        const devices = await navigator.bluetooth.getDevices()

        console.log('> Got ' + devices.length + ' Bluetooth devices.')
        devicesSelect.textContent = ''
        for (const device of devices) {
          const option = document.createElement('option')
          option.value = device.id
          option.textContent = device.name
          devicesSelect.appendChild(option)
        }
      } catch (error) {
        console.log('Argh! ' + error)
      }
    },

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
            // Cache the characteristic
            this.printCharacteristic = characteristic
            return characteristic
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

    async sendTextData() {
      var zpl = document.getElementById(this.printAreaId).innerHTML
      await this.printValue(zpl)
    },

    htmlTableToPlainText() {
      const section = document.getElementById(this.printAreaId)
      const title = 'LALALALA'
      const subTitle = '123'
      const description = '123'

      const tableRows = section.querySelectorAll('tr')
      let tableText = ''
      let index = 1
      // Loop through table rows and extract text
      tableRows.forEach((row) => {
        const cells = row.querySelectorAll('th, td')
        const rowText = Array.from(cells).map((cell) => cell.textContent.trim())
        let length1 = rowText[0].length
        let length2 = rowText[1].length
        let fillGap = ''
        let line = '---------------------------------------------' + '\n'
        let fixLength = 45 - length1 - length2
        for (let i = 0; i < fixLength; i++) {
          fillGap += ' '
        }
        if (index % 2 == 0) {
          tableText += rowText[0] + fillGap + rowText[1] + '\n'
        } else {
          tableText += rowText[0] + fillGap + rowText[1] + '\n' + line
        }
        index++
      })
      // Combine everything
      return `${title}\n${subTitle}\n\n${description}\n\n${tableText}\n\n`
    },

    async printValue() {
      console.log('print')
      const textToPrint = this.htmlTableToPlainText()
      console.log(textToPrint)
      // const encoder = new TextEncoder('utf-8')
      // const command = encoder.encode(textToPrint + '\n' + '\u000A\u000D')
      // return this.printCharacteristic.writeValue(command).then(() => {
      //   console.log('Write done.')
      // })
    },

    getSupportedProperties(characteristic) {
      let supportedProperties = []
      for (const p in characteristic.properties) {
        if (characteristic.properties[p] === true) {
          supportedProperties.push(p.toUpperCase())
        }
      }
      return '[' + supportedProperties.join(', ') + ']'
    },

    printDiv() {
      var myWindow = window.open('', '', 'width=800,height=800')
      var printContents = document.getElementById('print-area').innerHTML
      myWindow.document.write(printContents)

      myWindow.document.close()
      myWindow.focus()
      myWindow.print()
    },

    async printPage() {
      alert('print page')
      if (this.printCharacteristic === null) {
        await this.requestBluetoothAccess()
        this.sendTextData()
      } else {
        this.sendTextData()
      }
    }
  }
}
</script>

<template>
  <div id="print-area" style="font-size: 10px; line-height: 14px">
    <div
      style="
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        border-top: 2px dashed black;
        border-bottom: 2px dashed black;
        padding: 10px;
        width: 100%;
      "
    >
      <p class="print-header" style="margin: 0">PT. Dewata Exchange</p>
      <p class="print-header" style="margin: 0">
        Address: Jl. Legian No. 123, Legian, Kuta, Badung
      </p>
      <p class="print-header" style="margin: 0">Phone: (0361) 987654</p>
      <p class="print-header" style="margin: 0">Website: www.dewataexchange.com</p>
    </div>
    <div style="padding: 18px; border-bottom: 2px dashed black">
      <div style="display: flex; flex: 1; justify-content: space-between; align-items: baseline">
        <p class="print-header" style="margin: 0">Date: 09-10-2024</p>
        <p class="print-header" style="margin: 0">Time: 10:00</p>
      </div>
    </div>
    <div>
      <table
        style="
          width: 100%;
          border-bottom: 2px dashed black;
          padding-block: 14px;
          text-align: left;
          margin-bottom: 5px;
        "
      >
        <thead>
          <tr>
            <th><p>Currency Exchanged</p></th>
            <th><p>Amount</p></th>
            <th style="text-align: right"><p>Rate</p></th>
            <th style="text-align: right"><p>Total</p></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in transactionData.items" :key="index">
            <td>
              <p style="margin: 0">{{ item.product_code }} - {{ item.product_name }}</p>
            </td>
            <td>
              <p style="margin: 0">{{ Number(item.amount).toLocaleString('en') }}</p>
            </td>
            <td style="text-align: right">
              <p style="margin: 0">{{ Number(item.rate).toLocaleString('en') }}</p>
            </td>
            <td style="text-align: right">
              <p style="margin: 0">{{ Number(item.total).toLocaleString('en') }}</p>
            </td>
          </tr>
        </tbody>
      </table>
      <p class="print-footer" style="margin: 0">
        Total Amount: {{ Number(transactionData.trx_total).toLocaleString('en') }}
      </p>
      <div style="padding-block: 10px; border-bottom: 2px dashed black">
        <p class="print-footer" style="margin: 0">
          Transaction Number: {{ transactionData.trx_code }}
        </p>
      </div>
      <div style="padding-block: 10px; border-bottom: 2px dashed black; text-align: center">
        <p class="print-footer" style="margin: 0">Thank you for using our services!</p>
        <p class="print-footer" style="margin: 0">
          Contact us if you have any questions or issues.
        </p>
      </div>
    </div>
    <ButtonPrintBluetooth :printArea="printArea" />
    <button @click.prevent="printPage">print me</button>
  </div>
</template>
