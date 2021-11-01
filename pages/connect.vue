<template>
  <div>
    <b> connect </b>
    <div>
      <div>CurrentUser: {{ username }}</div>
      <div>source: {{ source }}</div>
      <div>
        <button @click="loginAccount">Login with currentUser</button>
      </div>

      <div>
        <button @click="switchAccount">Select Other Account</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { UPAccount, UPMessage, UPResponse } from "./up-types";

function receiveMessage(event: MessageEvent) {
  if (event.origin !== "http://localhost:3000") return;
  console.log("event", event.data);
}

export default Vue.extend({
  data() {
    return {
      source: "http://example.com",
      username: "johnz",
    };
  },
  mounted() {
    this.source = document.referrer;

    // send ready to opener
    window.opener.postMessage(new UPMessage("UP_READY"), this.source);
    window.addEventListener("message", this.handleMessage, false);
  },
  methods: {
    handleMessage(event: MessageEvent) {
      if (event.origin !== this.source) return;
      console.log("event", event.data);
    },
    loginAccount() {
      // send user info to opener
      window.opener.postMessage(
        new UPMessage(
          "UP_RESPONSE",
          JSON.stringify(
            new UPResponse("APPROVE", new UPAccount(this.username))
          )
        ),
        this.source
      );
    },
    switchAccount() {},
  },
});
</script>
