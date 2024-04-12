<script setup>
//Form POST 至 https://uat-fps.ctbcins.com/ap2ap/remit
  import {ref, computed} from 'vue';
  import debounce from 'lodash/debounce';

  const content = ref({
    accountNoReceive:"163540120484",
    accountType:"2",
    accountDate:"",  //日期
    memo1:"",  //虛擬繳款帳號扣掉開頭43824
    transactionAmount:"",  //費用
    transactionDate:"",  //日期
    transactionTime:"",  //時間
    crossBankNo:"8222876342",
    debitCredit:"C",
    amountType:"+",
    serviceManNo:"88888",
    transactionNo:"01045", //不用改
    transactionType:"1",
    accountNoSend:"8224477570016273316",
    memo2:"           ",  //11bytes
    companyCode:"",  //企業識別碼
    increase:"   00000000000000000 ", //21bytes
    serialNumber:"000000",
    filler:"   ", //3btyes
    remitter:"                                                                                ",  //80bytes
    remittanceMemo: "                                                                                "  //80bytes   
  });

  const now = new Date();
  const currentYear = now.getFullYear().toString();
  const rocYear = (parseInt(currentYear) - 1911).toString();
  const currentMonth = (now.getMonth() + 1).toString().padStart(2, '0');
  const currentDay = now.getDate().toString().padStart(2, '0');
  const currentHour = now.getHours().toString().padStart(2, '0');
  const currentMinute = now.getMinutes().toString().padStart(2, '0');
  const currentSecond = now.getSeconds().toString().padStart(2, '0');

  const date = ref(`${rocYear}${currentMonth}${currentDay}`);
  const time = ref(`${currentHour}${currentMinute}${currentSecond}`);
  const amount = ref("");
  const account = ref("");
  const transactionNo = ref("");

  const handlerDateInput = debounce(function (event){
    const dateInputValue = event.target.value;
    if (!/^\d*$/.test(dateInputValue)) {  //若輸入的不是數字就不處理
      event.target.value = "";
      return;
    } 
    if (dateInputValue.length > 7) {
      alert('日期格式錯誤，最多為七位數字，且必須符合民國(年月日)的格式');
      return;
    }
    if (dateInputValue.length === 7) {
      const year = parseInt(dateInputValue.substring(0, 3));
      const month = parseInt(dateInputValue.substring(3, 5));
      const day = parseInt(dateInputValue.substring(5, 7));
      const daysInMonth = new Date(year + 1911, month, 0).getDate();
      if (
        year < 1 || year > 999 ||
        month < 1 || month > 12 ||
        day < 1 || day > daysInMonth ||
        (month === 2 && day > 29 && (year % 4 === 0 && (year % 100 !== 0 || year % 400 ===0)))
      ){
        alert('日期格式錯誤，必須符合民國年月日');
        return;
      }
    }
  }, 1000);
  
  const handlerTimeInput = debounce(function (event){
    const timeInputValue = event.target.value;
    if (!/^\d*$/.test(timeInputValue)) {  //若輸入的不是數字就不處理
      event.target.value = "";
      return;
    }
    if (timeInputValue.length > 6) {
      alert('時間格式錯誤，最多為六位數字，且必須符合24小時制(時分秒)的格式');
      return;
    }
    if (timeInputValue.length === 6){
      const hour = parseInt(timeInputValue.substring(0, 2));
      const minute = parseInt(timeInputValue.substring(2, 4));
      const second = parseInt(timeInputValue.substring(4, 6));
      if (
        hour < 0 || hour > 23 ||
        minute < 0 || minute > 59 ||
        second < 0 || second > 59 
      ){
        alert('時間格式錯誤，必須符合(24小時制)時分秒');
        return;
      }
    }
  }, 1000);

  const handlerAmountInput = debounce(function (event){
    const amountInputValue = event.target.value;
    if (!/^\d*$/.test(amountInputValue)) {  //若輸入的不是數字就不處理
      event.target.value = "";
      return;
    }
    if (amountInputValue.length > 15){
      alert('超過轉帳金額上限');
      return;
    }
  }, 1000);

  const handlerAccountInput = debounce(function (event){
    const accountInputValue = event.target.value;
    if (!/^\d*$/.test(accountInputValue)) {  //若輸入的不是數字就不處理
      event.target.value = "";
      return;
    }
    if (accountInputValue.length > 14){
      alert('轉帳帳號格式錯誤，輸入之前五碼需為企業識別碼，後九碼需為虛擬繳款帳號')
      return;
    }
  }, 1000);

  const companyCodeMemo1 = computed(() => {
    const accountValue = account.value || "";
    return {
      companyCode: accountValue.substring(0, 5),
      memo1: accountValue.substring(5)
    };
  });

  const generateTransactionNo = function(){
    const dateValue = date.value;
    const timeValue = time.value;
    const serialNumber = Math.floor(Math.random() * 10000).toString().padStart(4, '0');

    return `000${dateValue}${timeValue}${serialNumber}`;
  }

  const concatenateData = function() {
    content.value.transactionDate = date.value;
    content.value.transactionTime = time.value;
    content.value.transactionAmount = amount.value.padStart(15, '0');
    content.value.accountDate = date.value;
    content.value.companyCode = companyCodeMemo1.value.companyCode;
    content.value.memo1 = companyCodeMemo1.value.memo1 + '     ';

    console.log(content.value);

    const keysInOrder = [
      'accountNoReceive',
      'accountType',
      'accountDate',
      'memo1',
      'transactionAmount',
      'transactionDate',
      'transactionTime',
      'crossBankNo',
      'debitCredit',
      'amountType',
      'serviceManNo',
      'transactionNo',
      'transactionType',
      'accountNoSend',
      'memo2',
      'companyCode',
      'increase',
      'serialNumber',
      'filler',
      'remitter',
      'remittanceMemo'
    ]
    let concatenatedData = "";
    keysInOrder.forEach(key => {
      concatenatedData += content.value[key];
    });
    console.log("串接完的字串:", concatenatedData);
    console.log(concatenatedData.length);

    const newTransactionNo = generateTransactionNo();
    transactionNo.value = newTransactionNo;

    sendData(concatenatedData);
  }

  const sendData = function(concatenatedData) {
    if (!date.value) {
      alert('日期欄位為必填');
      return;
    }
    const dateValue = parseInt(date.value);
    if (isNaN(dateValue) || date.value.length !== 7) {
      alert('日期格式錯誤，需為七位數字，且必須符合民國(年月日)的格式');
      return;
    }
    if (date.value.length === 7) {
      const year = parseInt(date.value.substring(0, 3));
      const month = parseInt(date.value.substring(3, 5));
      const day = parseInt(date.value.substring(5, 7));
      const daysInMonth = new Date(year + 1911, month, 0).getDate();
      if (
        year < 1 || year > 999 ||
        month < 1 || month > 12 ||
        day < 1 || day > daysInMonth ||
        (month === 2 && day > 29 && (year % 4 === 0 && (year % 100 !== 0 || year % 400 === 0)))
      ){
        alert('日期格式錯誤，必須符合(民國)年月日');
        return;
      }
    }

    if (!time.value) {
      alert('時間欄位為必填');
      return;
    }
    const timeValue = parseInt(time.value);
    if (isNaN(timeValue) || time.value.length !== 6) {
      alert('時間格式錯誤，需為六位數字，且必須符合24小時制(時分秒)的格式');
      return;
    }
    if (time.value.length === 6){
      const hour = parseInt(time.value.substring(0, 2));
      const minute = parseInt(time.value.substring(2, 4));
      const second = parseInt(time.value.substring(4, 6));
      if (
        hour < 0 || hour > 23 ||
        minute < 0 || minute > 59 ||
        second < 0 || second > 59 
      ){
        alert('時間格式錯誤，必須符合(24小時制)時分秒');
        return;
      }
    }

    if (!amount.value) {
      alert('轉帳金額欄位為必填');
      return;
    }
    if (content.value.transactionAmount.length !== 15) {
      alert('轉帳金額格式不正確');
      return;
    }

    if (!account.value) {
      alert('轉帳帳號欄位為必填');
      return;
    }
    if (account.value.length !== 14) {
      alert('轉帳帳號格式錯誤，輸入之前五碼需為企業識別碼，後九碼需為虛擬繳款帳號')
      return;
    }

    const form = document.createElement('form');
    form.setAttribute('method', 'POST');
    form.setAttribute('action', content.value.action);

    const msgIDField = document.createElement('input');
    msgIDField.setAttribute('type', 'hidden');
    msgIDField.setAttribute('name', 'MsgID');
    msgIDField.setAttribute('value', '2');
    form.appendChild(msgIDField);

    const transactionNoField = document.createElement('input');
    transactionNoField.setAttribute('type', 'hidden');
    transactionNoField.setAttribute('name', 'TransactionNo');
    transactionNoField.setAttribute('value', transactionNo.value);
    form.appendChild(transactionNoField);

    const bodyField = document.createElement('input');
    bodyField.setAttribute('type', 'hidden');
    bodyField.setAttribute('name', 'body');
    bodyField.setAttribute('value', concatenatedData);
    form.appendChild(bodyField);

    document.body.appendChild(form);
    form.submit();
    console.log(msgIDField.value);
    console.log(transactionNo.value);
    console.log(concatenatedData);
  }

  const editUrl = () => {
    const newUrl = prompt('請輸入新的URL:', 'https://uat-fps.ctbcins.com:8443/ap2ap/remit');
    if (newUrl !== null){
      content.value.action = newUrl;
      console.log(content.value.action);
    }
  };
</script>

<template>
  <div class="container">
    <div class="headline">Ap2Ap測試</div>
    <button class="edit-button" @click="editUrl">Edit</button>
    <div class="form">
      <div class="form-group">
        <label for="date">請輸入日期:</label>
        <input id="date" type="text" v-model="date" @input="handlerDateInput($event)" placeholder="ex:1120804">
      </div>
      <div class="form-group">
        <label for="time">請輸入時間:</label>
        <input id="time" type="text" v-model="time" @input="handlerTimeInput($event)" placeholder="ex:183524">
      </div>
      <div class="form-group">
        <label for="amount">請輸入轉帳金額:</label>
        <input id="amount" type="text" v-model="amount" @input="handlerAmountInput($event)" placeholder="ex:6500">
      </div>
      <div class="form-group">
        <label for="account">請輸入轉帳帳號:</label>
        <input id="account" type="text" v-model="account" @input="handlerAccountInput($event)" placeholder="ex:43824403202967" :placeholder="`${companyCodeMemo1.companyCode}${companyCodeMemo1.memo1}`">
      </div>
      <button @click="concatenateData">送出查詢</button>
    </div>
  </div>
</template>
  
<style scoped>
  .container {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
  }
  .headline {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 20px;
  }
  .form {
    display: flex;
    flex-direction: column;
  }
  .form-group {
    margin-bottom: 15px;
  }

  label {
    font-weight: bold;
  }

  input[type="text"] {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
  }

  button {
    padding: 10px 20px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
  }

  button:hover {
    background-color: #0056b3;
  }
</style>