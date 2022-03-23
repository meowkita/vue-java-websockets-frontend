<template>
  <h1 class="header">Vue-Java WebSockets</h1>
  <div class="counter">
    <div class="label">Counter:</div>
    <div class="number">{{ connected ? counter : "Offline" }}</div>
  </div>
  <div class="buttons">
    <button @click="this.click" :disabled="!connected">Click</button><br />
    <button @click="this.connect" :disabled="connected">Connect</button>
    <button @click="this.disconnect" :disabled="!connected">Disconnect</button>
  </div>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";

export default {
  name: "App",
  data() {
    return {
      counter: 0,
      send_message: null,
      connected: false,
    };
  },
  methods: {
    connect() {
      this.socket = new SockJS("http://192.168.43.180:8080/connect");
      this.stompClient = Stomp.over(this.socket);
      this.stompClient.connect(
        {},
        (frame) => {
          this.connected = true;
          console.log(frame);
          this.stompClient.subscribe("/counter/current", (tick) => {
            console.log(tick);
            this.counter = JSON.parse(tick.body);
          });
          this.stompClient.send("/ws/counter/current");
        },
        (error) => {
          console.log(error);
          this.connected = false;
        }
      );
    },
    disconnect() {
      this.stompClient.disconnect();
      this.connected = false;
    },
    click() {
      this.stompClient.send("/ws/counter/increase");
    },
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800&display=swap");

#app {
  font-family: "Poppins", sans-serif;
  font-size: 1rem;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}

.header {
  margin-top: 20px;
  margin-bottom: 30px;
}

.counter .label {
  font-size: 3rem;
}

.counter .number {
  font-size: 5.5rem;
  font-weight: bold;
  color: #36a566;
}

.buttons {
  margin-top: 10px;
}

.buttons button {
  font-family: inherit;
  padding: 10px 20px;
  border: #36a566 solid 4px;
  border-radius: 1rem;
  background-color: #36a56600;
  color: #36a566;
  font-size: 2.3rem;
  margin: 10px;
}

@media (max-width: 768px) {
  .buttons button {
    width: 80%;
  }
}

.buttons button:hover {
  color: #ffffff;
  background-color: #36a566;
}

.buttons button:disabled {
  color: #ffffff;
  border: #a5d3b9 solid 4px;
  background-color: #a5d3b9;
}
</style>
