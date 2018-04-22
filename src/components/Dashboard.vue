<template>
  <v-app id="app" dark>
    <v-dialog v-model="showTransferDialog" width="500px" class="dialog">
      <v-card>
        <v-card-text>
          <v-text-field label="Address" v-model="address"></v-text-field>
          <v-text-field label="Value" v-model="value"></v-text-field>
          <v-btn @click="transferTo(address, value)">Submit</v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
  
    <v-layout>
      <v-flex xs12 sm6 offset-sm3>
        <v-card>
          <v-container fill-height>
            <v-layout fill-height>
              <v-flex xs5>
                <v-select :items="accounts" v-model="activeAccount" label="Select Account" class="input-group--focused" item-text="name" item-value="address"></v-select>
              </v-flex>
              <v-flex xs12>
                <v-btn @click="showTransferDialog = true">Transfer</v-btn>
                <v-btn @click="createAccount()">New Account</v-btn>
              </v-flex>
              <v-flex xs12>
                Balance: {{ balance }}
              </v-flex>
            </v-layout>
          </v-container>
        </v-card>
      </v-flex>
    </v-layout>
  
  
  </v-app>
</template>

<script>
  var zippieprovider = require('vault-web3-provider')
  export default {
    name: 'dashboard',
    data() {
      return {
        address: undefined,
        value: undefined,
        BigNumber: undefined,
        showTransferDialog: false,
        activeAccount: undefined,
        accounts: [],
        newAccountName: undefined,
        balance: undefined
      }
    },
    created: function() {
      let self = this
      self.BigNumber = window.web3.utils.BN
      window.web3.eth.getAccounts().then(accounts => {
        self.accounts.push({
          name: "Account " + self.accounts.length,
          address: accounts[0]
        })
      })
      self.getBalance()
    },
    methods: {
      transferTo: function(address, value) {
        let valueInWei = window.web3.utils.toWei(value, "ether")
        window.web3.eth.getAccounts().then(accounts => {
          window.web3.eth.sendTransaction({
              from: accounts[0],
              gasPrice: "2000000000",
              gas: "21000",
              to: address,
              value: new self.BigNumber(valueInWei),
              data: ""
            }).once('transactionHash', function(hash) {
              console.log('transaction ' + hash)
            })
            .once('receipt', function(receipt) {
              console.log('receipt ' + receipt)
            })
            .on('confirmation', function(confirmationNumber, receipt) {
              console.log('confirmation ' + confirmationNumber + " " + receipt)
            })
            .on('error', console.error)
        })
  
      },
  
      createAccount: function() {
        let self = this
        zippieprovider.addAccount('m/' + Math.random() * 100000000).then((addy) => {
          self.accounts.push({
            name: "Account " + self.accounts.length,
            address: addy
          })
        })
      },
  
      getBalance: function() {
        let self = this
        if (self.activeAccount != undefined) {
          window.web3.eth.getBalance(self.activeAccount).then(balance => {
            self.balance = window.web3.utils.fromWei(balance, 'ether')
          })
        }
        window.setTimeout(function() {
          self.getBalance();
        }, 5000);
      }
    }
  }
</script>

<style>
  .dialog {
    margin: 20px;
  }
</style>


