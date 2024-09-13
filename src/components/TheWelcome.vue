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
          },
          {
            product_code: 'CAD',
            product_name: 'Canadian Dollar',
            rate: '12000',
            amount: '5',
            total: '60000'
          },
          {
            product_code: 'DIR',
            product_name: 'Uni Emirate Arab Dirham',
            rate: '12000',
            amount: '5',
            total: '60000'
          },
          {
            product_code: 'EUR',
            product_name: 'European Euro',
            rate: '15000',
            amount: '15',
            total: '225000'
          }
        ]
      }
    }
  },
  mounted() {
    this.printArea = document.getElementById('print-area')
  },
  methods: {
    printDiv() {
      var myWindow = window.open('', '', 'width=800,height=800')
      var printContents = document.getElementById('print-area').innerHTML
      myWindow.document.write(printContents)

      myWindow.document.close()
      myWindow.focus()
      myWindow.print()
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
              <p style="margin: 0">{{ item.amount }}</p>
            </td>
            <td style="text-align: right">
              <p style="margin: 0">{{ item.rate }}</p>
            </td>
            <td style="text-align: right">
              <p style="margin: 0">{{ item.total }}</p>
            </td>
          </tr>
        </tbody>
      </table>
      <p class="print-footer" style="margin: 0">Total Amount: {{ transactionData.trx_total }}</p>
      <div style="padding-block: 10px; border-bottom: 2px dashed black">
        <p class="print-footer" style="margin: 0">
          Transaction Number: {{ transactionData.trx_code }}
        </p>
      </div>

      <div style="padding-block: 10px; border-bottom: 2px dashed black; text-align: center">
        <p class="print-footer" style="margin: 0">
          Thank you for using our services. Contact us if you have any questions or issues.
        </p>
        <p class="print-footer" style="margin: 0">..</p>
      </div>
    </div>
    <ButtonPrintBluetooth :printArea="printArea" />
    <button @click.prevent="printPage">print me</button>
  </div>
</template>
