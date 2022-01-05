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
    var containerWidth = window.screen.width;
    var containerHeight = window.innerHeight;
    // console.log(containerWidth, containerHeight, window.screen.height);

    var cols = Math.floor((containerWidth - 30) / 9);
    cols = cols > 120 ? 120 : cols; // 最大120，有些编辑器不支持80以上的
    var rows = Math.floor((containerHeight - 60 - 20 - 20) / 17);
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
    terminalContainer.style.height = containerHeight - 60 - 20 - 20 + "px";
    this.term = new Terminal({
      rendererType: "canvas", //渲染类型
      rows, //行数
      cols, // 设置之后会输入多行之后覆盖现象
      convertEol: true, //启用时，光标将设置为下一行的开头
      scrollback: 10, //终端中的回滚量
      // fontSize: 14, //字体大小
      disableStdin: false, //是否应禁用输入。
      cursorStyle: "block", //光标样式
      cursorBlink: true, //光标闪烁
      tabStopWidth: 4,
      theme: {
        // foreground: "yellow", //字体
        // background: "#060101", //背景色
        // cursor: "help" //设置光标
      }
    });

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
    this.term.focus();

    console.log("mounted is going on");
  },
  beforeDestroy() {
    this.terminalSocket.close();
    // this.term.destroy();
    this.term.dispose();
  }
};
</script>
