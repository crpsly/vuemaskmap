<template>
  <div id="app">
    <div id="nav">
      <div class="logo">
        <img src="./assets/logo.png" alt="口罩即時查"><span>口罩即時查</span>
      </div>
      <div class="link">
        <router-link to="/">口罩供給現況</router-link> 
        <router-link to="/about">口罩怎麼買</router-link>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;900&display=swap');

$fullcolor: #11787a;
$fewcolor: #e67e22;
$nonecolor: #d4d6d7;
$maincolor: #34495e;

#app {
  font-family: 'Noto Sans TC', Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
  padding-bottom: 40px;
  box-sizing: border-box;
  background-color: #f8f9fa;
}

#nav {
  height: 10vh;
  background-color: #fff;
  color: $maincolor;
  display: flex;
  justify-content: space-between;
  padding: 0 40px;
  padding-top: 3vh;
  a {
    font-weight: bold;
    color: $maincolor;
    margin: 0 10px;

    &.router-link-exact-active {
      color: $maincolor;
      border-bottom: 2px solid $maincolor;
    }
  }
  img{
    width: 30px;
  }
}
@media(max-width: 500px){
#nav{
  padding: 30px 10px;
}
}
</style>
