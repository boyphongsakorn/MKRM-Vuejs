<script setup>
//import HelloWorld from "./components/HelloWorld.vue";
//import TheWelcome from "./components/TheWelcome.vue";
</script>

<!--template>
  <header>
    <img
      alt="Vue logo"
      class="logo"
      src="./assets/logo.svg"
      width="125"
      height="125"
    />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>

  <main>
    <TheWelcome />
  </main>
</template-->

<template>
  <img class="sticky" id="qrcode" style="z-index: 1;" />
  <div class="listscreen" style="height: 100%; display: flex">
    <div style="width: 70%;height: 100vh;" id="videodisplay">
      <!--div id="ytplayer" width="100%" height="100%"></div-->
      <YouTube src="https://www.youtube.com/watch?v=jNQXAC9IVRw" @ready="onReady" @state-change="onSc" ref="youtube"
        width="100%" height="100%" />
    </div>
    <div style="width: 30%" id="playlist">
      <table class="table table-secondary">
        <thead>
          <tr>
            <th scope="col">#</th>
            <th scope="col">ชื่อเพลง/วิดีโอ</th>
          </tr>
        </thead>
        <tbody id="musiclist"></tbody>
      </table>

      <!--input class="form-control" id="textyoutube" type="text" placeholder="Default input" aria-label="default input example">
            <button type="button" onclick="addnewvideo()" class="btn btn-primary">Primary</button-->
    </div>
  </div>
</template>

<style>
@import "./assets/base.css";

#app {
  /*max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;*/

  font-weight: normal;
  width: 100%;
}

img.sticky {
  position: fixed;
  right: 0;
  bottom: 0;
  width: 200px;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

a,
.green {
  text-decoration: none;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}

@media (hover: hover) {
  a:hover {
    background-color: hsla(160, 100%, 37%, 0.2);
  }
}

@media (min-width: 1024px) {
  body {
    /*display: flex;
    place-items: center;*/
    width: 100%;
  }

  #app {
    display: grid;
    /*grid-template-columns: 1fr 1fr;
    padding: 0 2rem;*/
  }

  header {
    display: flex;
    /*place-items: center;*/
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .logo {
    margin: 0 2rem 0 0;
  }
}
</style>

<script>
import { defineComponent } from 'vue'
import YouTube from 'vue3-youtube'
import anime from 'animejs'
import { useToast } from "vue-toastification";
const toast = useToast()

let player

export default defineComponent({
  components: { YouTube },
  methods: {
    onReady() {
      player = this.$refs.youtube;
      // player.setVolume(50);
      //make player mute
      player.mute();
      PlayerReady();
      //player.playVideo();
    },
    onSc(event) {
      console.log(event)
      bigtest(event)
      //console.log(this.$refs.youtube.state)
    }
  },
})

//after 5 seconds pause video
/*setTimeout(() => {
  player.pauseVideo();
}, 5000);*/

var i = 0;

anime({
  targets: document.querySelector("#playlist"),
  translateX: 250,
  direction: "reverse",
  easing: "easeInOutSine",
});

fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/gsettings.php", { mode: "cors" })
  .then((r) => r.json())
  .then((json) => {
    if (json[3] == "on") {
      console.log(window.location.hostname)
      //document.getElementById("qrcode").src = "http://api.qrserver.com/v1/create-qr-code/?data=" + encodeURI("https://" + window.location.hostname + "/addmusic.html") + "&size=100x100";
      document.getElementById("qrcode").src = "http://api.qrserver.com/v1/create-qr-code/?data=" + encodeURI("https://mkrm.pwisetthon.com/addmusic.html") + "&size=100x100";
      anime({
        targets: ".sticky",
        translateY: 250,
        direction: "reverse",
        easing: "easeInOutSine",
      });
    } else {
      document.getElementById("qrcode").style.display = "none";
    }
  });

//var datalist
var stopnext
var playlist = [];

function PlayerReady() {
  //set 4 of div index to height 100%
  document.querySelectorAll("div")[3].style.height = "100%";
  fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/gpl.php")
    .then((r) => r.json())
    .then((json) => {
      playlist = json;
      //loop playlist
      for (i = 0; i < playlist.length; i++) {
        document.getElementById("musiclist").innerHTML += "<tr><th scope=\"row\">" + (i + 1) + "</th><td>" + playlist[i].pl_ytname + "</td></tr>";
      }
      if (!playlist.length) {
        stopnext = setInterval(getnext, 5000)
        toast.warning("กรุณาเพิ่มเพลงเข้าลิสต์");
      } else {
        setTimeout(function () {
          player.loadVideoById(playlist[0].pl_ytid)
          var urlencoded = new URLSearchParams();
          urlencoded.append("plid", playlist[0].pl_id);
          fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/ctp.php", { method: "POST", body: urlencoded })
            .then((r) => r.json())
            .then((json) => { });
          toast.success("เจอเพลงแล้ว | กำลังเริ่มเล่นเพลง");
          setTimeout(function () {
            document.getElementById("playlist").style.display = "none";
            document.getElementById("videodisplay").style.width = "100%";
          }, 10000);
          playlist.shift()
          clearInterval(stopnext);
          for (i = 0; i < playlist.length; i++) {
            document.getElementById("musiclist").innerHTML += "<tr><th scope=\"row\">" + (i + 1) + "</th><td>" + playlist[i].pl_ytname + "</td></tr>";
          }
          player.playVideo();
        }, 5000)
      }
    });
}

setInterval(showdiflist, 3000);

function showdiflist() {
  if (document.getElementById("playlist").style.display == "none") {
    fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/gpl.php")
      .then((r) => r.json())
      .then((json) => {
        if (json.length > playlist.length || json.length > playlist) {
          document.getElementById("playlist").style.display = "block";
          document.getElementById("videodisplay").style.width = "70%";
          document.getElementById("musiclist").innerHTML = "";
          //loop json
          for (i = 0; i < json.length; i++) {
            document.getElementById("musiclist").innerHTML += "<tr class=\"musiclist\"><th scope=\"row\">" + (i + 1) + "</th><td>" + json[i].pl_ytname + "</td></tr>";
          }
          anime({
            targets: document.querySelector("#playlist"),
            translateX: 250,
            direction: "reverse",
            easing: "easeInOutSine",
          });
          anime({
            targets: ".musiclist",
            translateX: 700,
            direction: "reverse",
            easing: "easeInOutSine",
            delay: function (el, i) {
              return 1000 * i;
            },
            endDelay: function (el, i) {
              return 1000 * i;
            }
          });
          setTimeout(function () {
            document.getElementById("playlist").style.display = "none";
            document.getElementById("videodisplay").style.width = "100%";
          }, 20000);
          console.log("เพลงใหม่มาแล้ว");
          playlist = json;
        }
      });
  }
  player.unMute();
}

function getnext() {
  toast.warning("กรุณาเพิ่มเพลงเข้าลิสต์");
  fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/gpl.php")
    .then((r) => r.json())
    .then((json) => {
      playlist = json;
      if (playlist.length) {
        document.getElementById("playlist").style.display = "block";
        document.getElementById("videodisplay").style.width = "70%";
        document.getElementById("musiclist").innerHTML = "";
        anime({
          targets: document.querySelector("#playlist"),
          translateX: 250,
          direction: "reverse",
          easing: "easeInOutSine",
        });
        //loop playlist
        for (i = 0; i < playlist.length; i++) {
          document.getElementById("musiclist").innerHTML += "<tr><th scope=\"row\">" + (i + 1) + "</th><td>" + playlist[i].pl_ytname + "</td></tr>";
        }
        var urlencoded = new URLSearchParams();
        urlencoded.append("plid", playlist[0].pl_id);
        fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/ctp.php", { method: "POST", body: urlencoded })
          .then((r) => r.json())
          .then((json) => { });
        player.loadVideoById(playlist[0].pl_ytid);
        toast.success("เจอเพลงแล้ว | กำลังเล่นเพลง");
        setTimeout(function () {
          document.getElementById('playlist').style.display = "none";
          document.getElementById('videodisplay').style.width = "100%";
        }, 20000);
        playlist.shift();
        clearInterval(stopnext);
      }
    });
}

function bigtest(event) {
  fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/gpl.php")
    .then((r) => r.json())
    .then((json) => {
      document.getElementById("musiclist").innerHTML = "";
      playlist = json;
      //loop playlist
      for (i = 0; i < playlist.length; i++) {
        document.getElementById("musiclist").innerHTML += "<tr><th scope=\"row\">" + (i + 1) + "</th><td>" + playlist[i].pl_ytname + "</td></tr>";
      }
    });

  if (event.data == 0 && playlist.length > 0) {
    document.getElementById("playlist").style.display = "block";
    document.getElementById("videodisplay").style.width = "70%";
    anime({
      targets: document.querySelector("#playlist"),
      translateX: 250,
      direction: "reverse",
      easing: "easeInOutSine",
    });
    player.loadVideoById(playlist[0].pl_ytid);
    toast.success("กำลังเล่นเพลง "+playlist[0].pl_ytname);
    setTimeout(function () {
      document.getElementById("playlist").style.display = "none";
      document.getElementById("videodisplay").style.width = "100%";
    }, 20000);
    var urlencoded = new URLSearchParams();
    urlencoded.append("plid", playlist[0].pl_id);
    fetch("https://anywhere.pwisetthon.com/https://mkrm.pwisetthon.com/ctp.php", { method: "POST", body: urlencoded })
      .then((r) => r.json())
      .then((json) => { });
    playlist.shift();
  } else if (event.data == 0 && !playlist.length) {
    stopnext = setInterval(getnext, 5000);
  }
}
</script>