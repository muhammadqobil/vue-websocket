<template>
  <div class="flex justify-center column items-center" style="height: calc(100vh - 60px)">
    <div v-if="!connected">
      <div>
        <q-input v-model="from" type="text" outlined dense placeholder="Choose a nickname" />
      </div>
      <br />
      <div>
        <q-btn @click="connect" class="q-mx-sm" :disabled="connected">Connect</q-btn>
        <q-btn @click="disconnect" :disabled="!connected">Disconnect</q-btn>
      </div>
    </div>
    <br />
    <div v-if="connected">
      <q-form
        @submit.prevent="sendMessage"
        class="q-gutter-md"
      >
        <q-input
          v-model="phone"
          label="Manzil kiring"
          outlined
          dense
          class="q-pa-md col-xs-12 col-sm-12 col-md-12 col-lg-12"
          lazy-rules
          :rules="[ val => val && val.length > 0 || 'Please type something']"
        />

<!--        <q-input v-model="phone" placeholder="Telefon number"-->
<!--                 label="Telefon number"-->
<!--                 mask="(##) ### - ## - ##"-->
<!--                 fill-mask-->
<!--                 unmasked-value-->
<!--                 outlined-->
<!--                 class="q-pa-md col-xs-12 col-sm-12 col-md-12 col-lg-12" dense-->
<!--                 lazy-rules :rules="[val => !!val || this.$t('system.field_is_required')]">-->
<!--        </q-input>-->


        <div>
          <q-btn label="Submit" type="submit" color="primary"/>
        </div>
      </q-form>
<!--      <input v-model="text" type="text" placeholder="Write a message..." />-->
<!--      <button @click="sendMessage">Send</button>-->
      <p v-for="message in messages" :key="message.id">
        {{ message.from }}: {{ message.text }} ({{ message.time }})
      </p>
    </div>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue';
import SockJS from "sockjs-client/dist/sockjs"
import {Stomp} from '@stomp/stompjs';
import axios from "axios";

export default {
  name:'MyForm',
  setup() {
    const from = ref('');
    const text = ref('');
    const address = ref('');
    const phone = ref('');
    const connected = ref(false);
    const messages = ref([]);
    let stompClient = null;

    function connect() {
      const url = 'http://localhost:8081/chat'
      const socket = new SockJS(url);
      stompClient = Stomp.over(socket);
      stompClient.connect({}, (frame) => {
        connected.value = true;
        console.log('Connected: ' + frame);
        stompClient.subscribe('/topic/messages', (messageOutput) => {
          showMessageOutput(JSON.parse(messageOutput.body));
        });
      });
    };
    function disconnect() {
      if (stompClient !== null) {
          stompClient.disconnect();
      }
      connected.value = false;
      console.log('Disconnected');
    };
    function sendMessage() {
      console.log(JSON.stringify({ from: from.value, text: phone.value }))
      stompClient.send('/app/chat',{},JSON.stringify({ from: from.value, text: phone.value }));
      address.value = '';
      phone.value = '';
    };
    function showMessageOutput(messageOutput) {
      console.log('123=>',messageOutput)
      messages.value.push(messageOutput);
    };
    function setOrder() {
      axios.post('http://192.168.12.7:8080/orders', { location: address.value, phone: phone.value }, {
        headers: {
          "Content-Type": "application/json"
        }
      })
    };
    return {
      from,
      text,
      connected,
      messages,
      address,
      phone,

      //functions
      connect,
      disconnect,
      sendMessage,
      showMessageOutput,
      setOrder
    };
  },
};
</script>

<style scoped>
/* Add your styles here */
</style>
