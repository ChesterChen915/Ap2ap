<script setup>
//Form POST 至 https://uat-fps.ctbcins.com:8443/ap2ap/remit
  import { ref } from 'vue';
  import Ap2apUat from "./components/Ap2apUatFps.vue";

  const editDialog = ref(false);

  const ap2apUrl = ref('https://uat-fps.ctbcins.com:8443/ap2ap/remit')
  const updateData = () => {
    editDialog.value = false
  }
</script>

<template>
  <v-app>
    <v-app-bar :elevation="2">
      <v-app-bar-title>Ap2ap模擬轉帳測試</v-app-bar-title>
      <v-spacer></v-spacer>
      <v-btn class="ma-2" color="primary" @click="editDialog = true">edit</v-btn>
    </v-app-bar>
    <v-main class="d-flex align-center justify-center" style="min-height: 300px;">
      <Ap2apUat></Ap2apUat>
      <v-dialog v-model="editDialog" width="800">
        <v-card>
          <v-card-text>
            <v-table>
              <thead>
                <tr>
                  <th>參數</th>
                  <th>Value</th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <th>轉帳地址url</th>
                  <td><v-text-field v-model="ap2apUrl"></v-text-field></td>
                </tr>
              </tbody>
            </v-table>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" block @click="updateData">Close Dialog</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-main>
  </v-app>
</template>
  
<style scoped>

</style>