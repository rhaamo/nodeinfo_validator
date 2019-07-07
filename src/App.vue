<template>
  <div id="app">
    <div id="header">
      <h1>Nodeinfo Validator</h1>
      <span id='nav'><a href='https://github.com/jhass/nodeinfo/' target='_blank'>Nodeinfo</a> | <a href='https://github.com/rhaamo/nodeinfo_validator' target='_blank'>source</a></span>
      <br><br>
    </div>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
