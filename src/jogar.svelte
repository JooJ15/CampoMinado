<script>
  import { onMount } from "svelte";
  import { createEventDispatcher } from "svelte";

  createEventDispatcher();

  let linhas;
  let colunas;
  let numeroBombas;

  let bombas = [];
  let dificuldade = "0";

	let mostrarDialogo = false;
  let dialogoMensagem = "";

  onMount(() => {
    configurarDificuldade();
  });

  function gerarCoordenadasAleatorias() {
    const linha = Math.floor(Math.random() * linhas);
    const coluna = Math.floor(Math.random() * colunas);
    return [linha, coluna];
  }

  function configurarDificuldade() {
    if (dificuldade === "0") {
      linhas = 6;
      colunas = 6;
      numeroBombas = 5;
    } else if (dificuldade === "1") {
      linhas = 8;
      colunas = 8;
      numeroBombas = 12;
    } else if (dificuldade === "2") {
      linhas = 12;
      colunas = 12;
      numeroBombas = 25;
    } else if (dificuldade === "3") {
      linhas = 12;
      colunas = 12;
      numeroBombas = 50;
    }

    bombas = [];
    for (let i = 0; i < linhas; i++) {
      let row = [];
      for (let j = 0; j < colunas; j++) {
        row.push({ linha: i, coluna: j, bomba: false, revelada: false, marcada: false });
      }
      bombas.push(row);
    }

    for (let i = 0; i < numeroBombas; i++) {
      let coordenadas = gerarCoordenadasAleatorias();
      while (bombas[coordenadas[0]][coordenadas[1]].bomba) {
        coordenadas = gerarCoordenadasAleatorias();
      }
      bombas[coordenadas[0]][coordenadas[1]].bomba = true;
    }
  }

  function exibirMensagem(mensagem) {
  dialogoMensagem = mensagem;
  mostrarDialogo = true;
  mostrarImage = true;
}


	  function recarregarPagina() {
    location.reload();
  }
	
  function revelarCelula(linha, coluna) {
    if (bombas[linha][coluna].revelada) return;

    if (bombas[linha][coluna].bomba) {
      revelarBombas();
      bombas[linha][coluna].revelada = true;
      setTimeout(() => {
        exibirMensagem("Fim de jogo!");
      }, 100);
      return;
    }

    bombas[linha][coluna].revelada = true;
    if (contarBombasProximas(linha, coluna) === 0) {
      revelarCelulasVaziasVizinhas(linha, coluna);
    }

    if (verificarVitoria()) {
      setTimeout(() => {
        exibirMensagem("Você venceu!");
      }, 100);
    }
  }

  function revelarBombas() {
    bombas = bombas.map((row) =>
      row.map((celula) => {
        if (celula.bomba) {
          celula.revelada = true;
        }
        return celula;
      })
    );
  }

  function contarBombasProximas(linha, coluna) {
    let contador = 0;
    for (let i = linha - 1; i <= linha + 1; i++) {
      for (let j = coluna - 1; j <= coluna + 1; j++) {
        if (i >= 0 && i < linhas && j >= 0 && j < colunas) {
          if (bombas[i][j].bomba) {
            contador++;
          }
        }
      }
    }
    return contador;
  }

  function revelarCelulasVaziasVizinhas(linha, coluna) {
    for (let i = linha - 1; i <= linha + 1; i++) {
      for (let j = coluna - 1; j <= coluna + 1; j++) {
        if (i >= 0 && i < linhas && j >= 0 && j < colunas) {
          if (!bombas[i][j].revelada) {
            revelarCelula(i, j);
          }
        }
      }
    }
  }

  function verificarVitoria() {
    for (let i = 0; i < linhas; i++) {
      for (let j = 0; j < colunas; j++) {
        if (!bombas[i][j].bomba && !bombas[i][j].revelada) {
          return false;
        }
      }
    }
    return true;
  }

  function marcarCelula(linha, coluna, event) {
    event.preventDefault();

    if (!bombas[linha][coluna].revelada) {
      bombas[linha][coluna].marcada = !bombas[linha][coluna].marcada;
    }
  }
</script>

<style>
  section{
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .dificuldade {
   text-align: center;
   color: white;
   font-size:15px;
   margin-bottom: 10px;
  }

  select {
    width: 60%;
    border-radius: 5px;
    height: 35px;
  }

  table {
    border-collapse: collapse;
    margin-left: auto;
    margin-right: auto;
  }

  .bomba {
    background-color: red;
    background-image: url("/../bomba.svg");
    background-size: contain; 
    background-repeat: no-repeat;
  }
	  .tela-escura {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.9);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
    color: white;
    font-size: 36px;
    text-align: center;
    cursor: pointer;
  }

  td {
    position: relative;
  }

	  .numero {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

	  .marcada {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-weight: bold;
  }

</style>
<section>
  
  <div class="dificuldade">
    Dificulade:
    <select bind:value="{dificuldade}" on:change="{configurarDificuldade}">
      <option value="0">Fácil</option>
      <option value="1">Médio</option>
      <option value="2">Difícil</option>
      <option value="3">Extremo</option>
    </select>
  </div>
  <table>
    <tbody>
      {#each bombas as row, linha}
        <tr>
          {#each row as celula, coluna}
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <td
              style="width: 42px; height: 42px; border: 2px solid white; background-color: {celula.revelada ? 'rgb(223 189 98)' : 'rgb(218, 72, 36)'}; {celula.bomba && celula.revelada ? 'background-color: red;' : ''}"
              class="{celula.bomba && celula.revelada ? 'bomba' : ''}"
              on:click="{() => revelarCelula(linha, coluna)}"
              on:contextmenu="{(event) => marcarCelula(linha, coluna, event)}"
            >
              {#if celula.revelada && !celula.bomba && contarBombasProximas(linha, coluna) !== 0}
                <span class="numero">{contarBombasProximas(linha, coluna)}</span>
              {/if}
              {#if celula.marcada && !celula.revelada}
                <span class="marcada"></span>
                <img src="../bandeira.svg" alt="bandeira">
              {/if}
            </td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>

</section>

{#if mostrarDialogo}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="tela-escura" on:click="{recarregarPagina}">
    <div class="dialogo">
      <div class="dialogo-container">
        <p>{dialogoMensagem}</p>
      </div>
    </div>
  </div>
{/if}