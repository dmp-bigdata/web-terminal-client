<template>
  <div class="console" id="terminal"></div>
</template>
<script>
import { WebLinksAddon } from "xterm-addon-web-links";
import { SearchAddon } from "xterm-addon-search";
import { AttachAddon } from "xterm-addon-attach";
import { FitAddon } from "xterm-addon-fit";
import { Terminal } from "xterm";
export default {
  name: "Console",
  props: {
    msg: {
      type: String
    },
    username: {
      type: String
    },
    password: {
      type: String
    }
  },
  data() {
    return {
      term: null,
      terminalSocket: null
    };
  },
  methods: {
    runRealTerminal() {
      console.log("webSocket is finished");
    },
    errorRealTerminal() {
      console.log("error");
    },
    closeRealTerminal() {
      console.log("close");
    }
  },
  mounted() {
    var containerWidth = window.screen.height;
    var containerHeight = window.innerHeight; //window.screen.width;
    var cols = Math.floor((containerWidth - 30) / 9);
    var rows = Math.floor(containerHeight / 17) - 2;
    console.log("cols", cols, "rows", rows);
    var url = "";
    if (this.username === undefined) {
      url =
        (location.protocol === "http:" ? "ws" : "wss") +
        "://" +
        location.hostname +
        ":5001" +
        "/ws" +
        "?" +
        "msg=" +
        this.msg +
        "&rows=" +
        rows +
        "&cols=" +
        cols;
    } else {
      url =
        (location.protocol === "http:" ? "ws" : "wss") +
        "://" +
        location.hostname +
        ":5001" +
        "/ws" +
        "?" +
        "msg=" +
        this.msg +
        "&rows=" +
        rows +
        "&cols=" +
        cols +
        "&username=" +
        this.username +
        "&password=" +
        this.password;
    }
    // open websocket
    this.terminalSocket = new WebSocket(url);
    this.terminalSocket.onopen = this.runRealTerminal;
    this.terminalSocket.onclose = this.closeRealTerminal;
    this.terminalSocket.onerror = this.errorRealTerminal;
    // create terminal
    let terminalContainer = document.getElementById("terminal");
    this.term = new Terminal();

    // Load WebLinksAddon on terminal, this is all that's needed to get web links
    // working in the terminal.
    this.term.loadAddon(new WebLinksAddon());
    // SearchAddon
    const searchAddon = new SearchAddon();
    this.term.loadAddon(searchAddon);
    // FitAddon
    const fitAddon = new FitAddon();
    this.term.loadAddon(fitAddon);

    // AttachAddon
    const attachAddon = new AttachAddon(this.terminalSocket);
    this.term.loadAddon(attachAddon);

    this.term.open(terminalContainer);
    // FitAddon
    fitAddon.fit();

    // this.term.attach(this.terminalSocket);
    this.term._initialized = true;
    console.log("mounted is going on");
  },
  beforeDestroy() {
    this.terminalSocket.close();
    // this.term.destroy();
    this.term.dispose();
  }
};
</script>
