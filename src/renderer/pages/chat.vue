<template>
  <v-app id="vuetify">
    <v-system-bar app>
      <v-spacer></v-spacer>

      <v-icon>mdi-square</v-icon>

      <v-icon>mdi-circle</v-icon>

      <v-icon>mdi-triangle</v-icon>

    </v-system-bar>

    <v-navigation-drawer
      app
      v-model="drawer"
    >
      <v-sheet
        class="pa-4"
        color="grey lighten-4"
      >
        <v-avatar
          class="mb-4"
          color="grey darken-1"
          size="64"
        ></v-avatar>

        <div>john@vuetifyjs.com</div>
      </v-sheet>

      <v-divider></v-divider>

      <v-list>
        <v-list-item
          :key="icon"
          link
          v-for="[icon, text] in links"
        >
          <v-list-item-icon>
            <v-icon>{{ icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ text }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main
    >
      <div
        class="py-8 px-6"
        style="display: flex; flex-flow: row wrap; min-height: 100%">
        <div style="align-self: flex-start;">
          <v-row>
            <v-col
              :key="card"
              cols="12"
              v-for="card in cards"
            >
              <v-card
                elevation="2"
                outlined
              >
                <v-card-title>{{ card }}</v-card-title>
              </v-card>
            </v-col>
          </v-row>
        </div>
        <div style="align-self: flex-end; min-width: 100%">
          <v-row>
            <v-col cols="12">
              <v-textarea
                @click:append="sendClick"
                append-icon="mdi-send"
                auto-grow
                clearable
                rounded
                rows="1"
                solo
                v-model="msg"
              >
              </v-textarea>
            </v-col>
          </v-row>
        </div>
      </div>
    </v-main>
  </v-app>
</template>

<script>
  const net = require('net');

  export default {
    name: "vuetify",
    data: () => ({
      cards: ['Hello, Welcome to rChat.', 'Enjoy yourself. XD'],
      drawer: null,
      msg: '',
      socket: null,
      cbor: null,
      links: [
        ['mdi-inbox-arrow-down', 'Inbox'],
        ['mdi-send', 'Send'],
        ['mdi-delete', 'Trash'],
        ['mdi-alert-octagon', 'Spam'],
      ],
    }),
    methods: {
      send(msg) {
        const cborMsg = this.cbor?.encodeOne(msg)
        const size = cborMsg.length
        const newMessage = Buffer.allocUnsafe(2);
        newMessage.writeInt16BE(size, 0);
        let finalMsg = Buffer.concat([newMessage, cborMsg]);
        this.socket?.write(finalMsg)
      },
      sendClick() {
        console.log(this.msg);
        this.send({"cmd": "Message", "data": this.msg});
        this.cards.push(this.msg);
        this.msg = '';
      },
      handle(msg) {
        let size = msg.readInt16BE(0);
        msg = msg.slice(2);
        let val = this.cbor.decodeAllSync(msg)[0];

        console.log("ctor", val);
        switch (val.cmd) {
          case "Ping": break;
          case "Message":
            this.cards.push(val.data);
            break;
        }
      }
    },
    created() {
      const port = 8080;
      const host = '127.0.0.1';
      this.socket = new net.Socket();
      console.log(this.socket);
      this.socket.connect(port, host);

      this.cbor = require('cbor');

      setInterval(() => {
        const hb = {"cmd": "Ping"};
        this.send(hb);
      }, 5000);

      this.socket.on('data', msg => {
        this.handle(msg);
      });

    },
  }
</script>

<style scoped>

</style>
