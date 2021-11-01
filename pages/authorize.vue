<template>
  <div>
    <b> connect </b>
    <div>
      <div>CurrentUser: {{ username }}</div>
      <div>source: {{ source }}</div>
      <div>
        <textarea v-model="message"></textarea>
      </div>
      <div>
        <button @click="approve">Approve</button>
      </div>

      <div>
        <button @click="reject">Reject</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import {
  UPAccount,
  UPMessage,
  UPAuthMessage,
  UPAuthResponse,
  UPResponse,
} from "./up-types";

export default Vue.extend({
  data() {
    return {
      source: "http://example.com",
      username: "johnz",
      message: "",
    };
  },
  mounted() {
    this.source = document.referrer;
    // send ready to opener
    window.opener.postMessage(new UPMessage("UP_READY"), this.source);
    window.addEventListener("message", this.messageHandler, false);
  },
  methods: {
    closeWindow() {
      window.removeEventListener("message", this.messageHandler);
    },
    postMessage(message: UPMessage) {
      window.opener.postMessage(message, this.source);
    },
    messageHandler(event: MessageEvent) {
      if (event.origin !== "http://localhost:3000") return;
      console.log("event", event.data);
      const { type, payload } = event.data as UPMessage;

      // check type
      if (type !== "UP_AUTH") {
        this.postMessage(
          new UPMessage(
            "UP_RESPONSE",
            JSON.stringify(
              new UPResponse("DECLINE", `wrong message type ${type}`)
            )
          )
        );
        this.closeWindow();
        return;
      }

      const {
        type: authType,
        username,
        payload: authPayload,
        hash,
      } = JSON.parse(payload as string) as UPAuthMessage;

      // check username
      if (username !== this.username) {
        this.postMessage(
          new UPMessage(
            "UP_RESPONSE",
            JSON.stringify(
              new UPResponse("DECLINE", "username not match, please relogin")
            )
          )
        );

        this.closeWindow();
        return;
      }

      this.message = authPayload;
    },
    approve() {
      // send user info to opener
      this.postMessage(
        new UPMessage(
          "UP_RESPONSE",
          JSON.stringify(
            new UPResponse(
              "APPROVE",
              new UPAuthResponse("RSA", "pubkeyhere", "sighere")
            )
          )
        )
      );

      this.closeWindow();
    },
    reject() {
      this.postMessage(
        new UPMessage(
          "UP_RESPONSE",
          JSON.stringify(new UPResponse("DECLINE", "user reject signature"))
        )
      );
      this.closeWindow();
    },
  },
});
</script>
