<template>
<div id="app">
<main>
  <div class="search-box">
    <input type="text" class="search-bar" placeholder="Pesquisar CNPJ" v-model="query" @keypress="fetchCNPJ"/>
    <button class="button" @click="fetchCNPJ2()">Pesquisar</button>
  </div>
  <div class="cnpjlog">
  <ul style="display: inline;" v-for="(cnpj2,n) in cnpj2s" :key="cnpj2.id">

    <span class="cnpj2">{{cnpj2}}</span> <button class="button2" @click="removeCnpj2(n)">Limpar</button>

  </ul>
  </div>
  <div class="json-wrap" v-if="typeof cnpj.message != 'undefined'">
    <div class="error-box">
      <div class="temp">{{cnpj.message}}</div>
    </div></div>
  <div class="json-wrap" v-if="typeof cnpj.cnpj != 'undefined'">
    <div class="raw-box">
      <div class="temp">{{cnpj}}</div>
    </div>
    <div class="data-box">
      <div class="cnpj">{{cnpj.cnpj}}</div>
      <div class="nome">Razão Social: {{cnpj.razao_social}}</div>
      <div class="nome">Nome Fantasia: {{cnpj.nome_fantasia}}</div>
      <div class="nome">Endereço: {{cnpj.descricao_tipo_logradouro}} {{cnpj.logradouro}}, {{cnpj.numero}} {{cnpj.complemento}}, {{cnpj.bairro}} - {{cnpj.municipio}}/{{cnpj.uf}}</div>
      <div class="nome">CEP: {{cnpj.cep}}</div>
      <div class="nome">Telefone: {{cnpj.ddd_telefone_1}}</div>
      <div class="nome">Situação: {{cnpj.descricao_situacao_cadastral}}</div>
      <div class="nome">Início de Atividade: {{cnpj.data_inicio_atividade}}</div>
      <div class="nome">CNAE: {{cnpj.cnae_fiscal}}</div>
      <div class="nome">Descrição Primária: {{cnpj.cnae_fiscal_descricao}}</div>
      <div class="nome">Descrição Secundária: {{cnpj.cnaes_secundarias[0].descricao}}</div>
      <div class="nome">Capital Inicial: R${{cnpj.capital_social}}</div>
      <div class="nome">Porte: {{cnpj.descricao_porte}}</div>
      <div class="nome">Sócio 1: {{cnpj.qsa[0].nome_socio}}</div>
      <div class="nome">Sócio 2: {{cnpj.qsa[1].nome_socio}}</div>
    </div>

    
    </div>
</main>
</div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
    query: "",
    cnpj: {},
    cnpj2s:[],
    newCnpj2:null
    }
  },
  mounted() {
    if(localStorage.getItem('cnpj2s')) {
      try {
        this.cnpj2s = JSON.parse(localStorage.getItem('cnpj2s'));
      } catch(e) {
        localStorage.removeItem('cnpj2s');
      }
    }
  },
  methods: {
    fetchCNPJ (e) {
      if (validarCNPJ(this.query)){
      if (e.key == "Enter") {
        // ensure they actually typed something
      if(!this.query) return;
      this.cnpj2s.push(this.query);
      this.newCnpj2 = '';
      this.saveCnpj2s();
        fetch(`https://minhareceita.org/${this.query}`)
          .then(res => {
            return res.json();
          }).then(this.setResults);
      }}
    },
    fetchCNPJ2 () {
      // ensure they actually typed something
      if(!this.query) return;
      this.cnpj2s.push(this.query);
      this.newCnpj2 = '';
      this.saveCnpj2s();
        fetch(`https://minhareceita.org/${this.query}`)
          .then(res => {
            return res.json();
          }).then(this.setResults);
      
    },
    setResults(results) {
      
      this.cnpj = results;
      console.log(this.cnpj)
    },
  addCnpj2() {
      // ensure they actually typed something
      if(!this.query) return;
      this.cnpj2s.push(this.query);
      this.query = '';
      this.saveCnpj2s();
    },
    removeCnpj2(x) {
      this.cnpj2s.splice(x,1);
      this.saveCnpj2s();
    },
    saveCnpj2s() {
      let parsed = JSON.stringify(this.cnpj2s);
      localStorage.setItem('cnpj2s', parsed);
    }
  }
  };
  function validarCNPJ(cnpj) {
    cnpj = cnpj.replace(/[^\d]+/g,'');
 
    if(cnpj == '') return false;
     
    if (cnpj.length != 14)
        return false;
 
    // Elimina CNPJs invalidos conhecidos
    if (cnpj == "00000000000000" || 
        cnpj == "11111111111111" || 
        cnpj == "22222222222222" || 
        cnpj == "33333333333333" || 
        cnpj == "44444444444444" || 
        cnpj == "55555555555555" || 
        cnpj == "66666666666666" || 
        cnpj == "77777777777777" || 
        cnpj == "88888888888888" || 
        cnpj == "99999999999999"){
      alert("CNPJ Inválido!")
          return false;}
         
    // Valida DVs
    let tamanho = cnpj.length - 2
    let numeros = cnpj.substring(0,tamanho);
    let digitos = cnpj.substring(tamanho);
    let soma = 0;
    let pos = tamanho - 7;
    for (let i = tamanho; i >= 1; i--) {
      soma += numeros.charAt(tamanho - i) * pos--;
      if (pos < 2)
            pos = 9;
    }
    let resultado = soma % 11 < 2 ? 0 : 11 - soma % 11;
    if (resultado != digitos.charAt(0)){
      alert("CNPJ Inválido!")
          return false;}
         
    tamanho = tamanho + 1;
    numeros = cnpj.substring(0,tamanho);
    soma = 0;
    pos = tamanho - 7;
    for (let i = tamanho; i >= 1; i--) {
      soma += numeros.charAt(tamanho - i) * pos--;
      if (pos < 2)
            pos = 9;
    }
    resultado = soma % 11 < 2 ? 0 : 11 - soma % 11;
    if (resultado != digitos.charAt(1)) {
      alert("CNPJ Inválido!")
          return false;}
           
    return true;
    
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'montserrat', sans-serif;
}
#app {
  background-image: url('./assets/2cold-bg.jpg');
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
#app.warm {
  background-image: url('./assets/warm-bg.jpg');
}
main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
}
.button {
    display: block;
    margin-top: -46px;
    min-height: 10px;
    padding: 5px;
    color: #313131;
    font-size: 20px;
    outline: none;
    background: none;
    box-shadow: 0px 0px 8px rgb(66 60 60 / 25%);
    background-color: rgba(40, 212, 255, 0.5);
    border-radius: 0px 16px 0px 16px;
    margin-left: 324px;
 }
 .button2 {
    padding: 5px;
    color: #ffffff;
    font-size: 10px;
    box-shadow: 0px 0px 8px rgb(66 60 60 / 25%);
    background-color: rgba(184, 3, 3, 0.5);
    border-radius: 16px 0px 16px 0px;
 }
.search-box {
  float: left;
  width: 36%;
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border:none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}
.data-box .cnpj {
  color: #FFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.cnpj2 {
  color: rgb(234, 241, 193);
  font-size: 15px;
  padding-left: 4px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.cnpjlog {
  color: rgb(234, 241, 193);
  display: block;
  font-size: 15px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  
}
.data-box .nome {
  color: #FFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}
.raw-box {
  text-align: center;
}
.raw-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #FFF;
  font-size: 10px;
  font-weight: 200;
  background-color:rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.error-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: rgb(245, 220, 220);
  font-size: 30px;
  font-weight: 200;
  background-color:rgba(238, 92, 92, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
</style>
