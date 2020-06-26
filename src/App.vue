<template>
  <div id="app" class="container">
    <div class="header">
      <h1><img src="icon/unown-report.png">Cramculator<small class="small-sub">beta</small></h1>
      <p>
        Calculator for Cram-o-matic item combinations in Pokémon Sword/Shield Isle of Armor DLC.<br>
        This tool is still under development. For bug reports, suggestions or lending some help, check the github page: <a href="https://github.com/lmmfranco/cramculator">here</a>
      </p>
    </div>
    <Main/>
    <div class="footer">
      <span>
        Data obtained from:
        <a href="https://bulbapedia.bulbagarden.net/wiki/Cram-o-matic">Bulbapedia</a> |
        <a href="https://serebii.net/swordshield/cram-o-matic.shtml">Serebii</a> |
        <a href="https://www.youtube.com/watch?v=TieEbKKaf1o">Austin John Plays</a>
      </span> |
      <span>Sprites by: <a href="https://github.com/msikma/pokesprite">Pokesprite</a></span>
      <br>
      <span>Coded by: <a href="mailto:lmmfranco@gmail.com">lmmfranco@gmail.com</a></span> | 
      <span>Contribute here: <a href="https://github.com/lmmfranco/cramculator">GitHub</a></span>
      <br>
      <span>
        Tool made for free with no commercial use.
        Pokémon © 2002-2020 Pokémon. © 1995-2020 Nintendo/Creatures Inc./GAME FREAK inc. ™, ® and Pokémon character names are trademarks of Nintendo.
        No copyright or trademark infringement is intended in using Pokémon content on this page.
      </span>
    </div>
  </div>
</template>

<script>
import Main from './components/Main.vue'

export default {
  name: 'App',
  components: {
    Main
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.small-sub {
  font-size: 45%;
}
.footer {
  margin-top: 20px;
  font-size: 12px;
}
</style>
