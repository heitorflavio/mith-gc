<script setup>
import { ref, onMounted } from "vue";

const lobbies = ref([]);

const extractInfo = async () => {
  let [tab] = await chrome.tabs.query({ active: true, currentWindow: true });

  chrome.scripting.executeScript({
    target: { tabId: tab.id },
    function: () => {
      document.querySelectorAll("article.LobbyRoom").forEach((lobby) => {
        const jogadores = [...lobby.querySelectorAll(".LobbyPlayerVertical")];

        // Verifica se a lobby possui exatamente 5 jogadores
        if (jogadores.length === 5) {
          // Nome da lobby
          const nomeLobby = lobby
            .querySelector(".LobbyRoom__title")
            ?.textContent.trim();

          // ID da sala (extraído do ID do contêiner de RoomCardWrapper)
          const idSala = lobby.closest(".RoomCardWrapper")?.id;

          // Dados dos jogadores
          const detalhesJogadores = jogadores.map((player) => ({
            id: player.getAttribute("href")?.match(/\/jogador\/(\d+)/)?.[1], // Extrai o ID do href
            nome: player.title.split("|")[0].trim(),
            kdr: player.querySelector("[kdr]")?.getAttribute("kdr"),
            nivel: player
              .querySelector(".LevelBadge__simple, .LevelBadge__subscriber")
              ?.textContent.trim(),
            adr: 0,
            wins: 0,
            loss: 0,
            matches: 0,
            hs: "0%",
          }));

          detalhesJogadores.forEach(async (player) => {
            try {
              console.log(player.id);

              const response = await this.$axios.get(
                `https://gamersclub.com.br/api/box/history/${player.id}`,
                {
                  headers: {
                    authorization:
                      "Basic ZnJvbnRlbmQ6NDdhMTZHMmtHTCFmNiRMRUQlJVpDI25X",
                    origin: "gamersclub.com.br",
                  },
                }
              );

              // The response data is directly available in response.data
              console.log(response.data);
            } catch (error) {
              console.error("Error fetching profile:", error);
            }
          });

          // pega a lobby e
          const component = document.querySelector(`#${idSala}`);
          if (component) {
            // pega a tag article
            const article = component.querySelector("article");
            // muda a cor do background para um amarelo transparente
            article.style.backgroundColor = "rgba(255, 255, 0, 0.5)";
            // adiciona os detalhes dos jogadores
            article.dataset.jogadores = JSON.stringify(detalhesJogadores);

            article.addEventListener("click", () => {
              // Verifica se já existe um modal
              if (document.querySelector(".custom-modal")) return;

              // Cria o modal
              const modal = document.createElement("div");
              modal.classList.add("custom-modal");
              modal.style.position = "fixed";
              modal.style.top = "0";
              modal.style.width = "100%";
              modal.style.height = "100%";
              modal.style.backgroundColor = "rgba(0, 0, 0, 0.5)";
              modal.style.zIndex = "999";
              modal.style.display = "flex";
              modal.style.justifyContent = "center";
              modal.style.alignItems = "center";

              const content = document.createElement("div");
              content.style.backgroundColor = "#1e293b"; // bg-slate-800
              content.style.padding = "20px";
              content.style.borderRadius = "10px";
              content.style.boxShadow = "0 4px 6px rgba(0, 0, 0, 0.1)";

              const table = document.createElement("table");
              table.style.width = "100%";
              table.style.borderCollapse = "collapse";
              table.style.borderSpacing = "0";
              table.style.color = "#e2e8f0"; // text-slate-300
              table.style.marginBottom = "20px";

              // Cabeçalho
              const thead = document.createElement("thead");
              thead.style.backgroundColor = "#334155"; // bg-slate-700
              thead.style.color = "#f1f5f9"; // text-slate-100

              const headerRow = document.createElement("tr");
              [
                "Nome",
                "KDR",
                "Nível",
                "ADR",
                "Wins",
                "Loss",
                "Matches",
                "HS",
              ].forEach((header) => {
                const th = document.createElement("th");
                th.textContent = header;
                th.style.padding = "10px";
                th.style.textAlign = "left";
                th.style.borderBottom = "2px solid #475569"; // Border slate-600
                headerRow.appendChild(th);
              });

              thead.appendChild(headerRow);
              table.appendChild(thead);

              // Corpo
              const tbody = document.createElement("tbody");
              detalhesJogadores.forEach((player, index) => {
                const row = document.createElement("tr");
                row.style.backgroundColor =
                  index % 2 === 0 ? "#1e293b" : "#0f172a"; // Alternar entre bg-slate-800 e bg-slate-900
                row.style.transition = "background-color 0.3s ease";
                row.addEventListener("mouseover", () => {
                  row.style.backgroundColor = "#475569"; // Hover: bg-slate-600
                });
                row.addEventListener("mouseout", () => {
                  row.style.backgroundColor =
                    index % 2 === 0 ? "#1e293b" : "#0f172a";
                });

                [
                  "nome",
                  "kdr",
                  "nivel",
                  "adr",
                  "wins",
                  "loss",
                  "matches",
                  "hs",
                ].forEach((key) => {
                  const td = document.createElement("td");
                  td.textContent = player[key];
                  td.style.padding = "10px";
                  td.style.borderBottom = "1px solid #334155"; // Border slate-700
                  td.style.textAlign = "center";
                  row.appendChild(td);
                });

                tbody.appendChild(row);
              });

              table.appendChild(tbody);
              content.appendChild(table);

              // Botão para fechar o modal
              const closeButton = document.createElement("button");
              closeButton.textContent = "Fechar";
              closeButton.style.padding = "10px 20px";
              closeButton.style.backgroundColor = "#ef4444"; // bg-red-500
              closeButton.style.color = "#fff"; // text-white
              closeButton.style.border = "none";
              closeButton.style.cursor = "pointer";
              closeButton.style.borderRadius = "5px";
              closeButton.style.transition = "background-color 0.3s ease";
              closeButton.style.alignSelf = "center";
              closeButton.addEventListener("mouseover", () => {
                closeButton.style.backgroundColor = "#dc2626"; // Hover: bg-red-600
              });
              closeButton.addEventListener("mouseout", () => {
                closeButton.style.backgroundColor = "#ef4444";
              });
              closeButton.addEventListener("click", () => {
                modal.remove();
              });

              content.appendChild(closeButton);
              modal.appendChild(content);
              document.body.appendChild(modal);
            });
          }

          // Exibe a lobby com o ID da sala e os jogadores
          console.log({
            idSala,
            nomeLobby,
            jogadores: detalhesJogadores,
          });

          // Retorna os dados da lobby
          return {
            idSala,
            nomeLobby,
            jogadores: detalhesJogadores,
          };
        }
      });
    },
  });
};

onMounted(() => {
  extractInfo();
  // teste();
});

const teste = async () => {
  const response = await axios.get(
    "https://gamersclub.com.br/api/box/history/1708945",
    {
      headers: {
        authorization: "Basic ZnJvbnRlbmQ6NDdhMTZHMmtHTCFmNiRMRUQlJVpDI25X",
        origin: "gamersclub.com.br",
      },
    }
  );

  console.log(response.data);
};
</script>

<template>
  <div class="card" v-if="lobbies.length">
    <button type="button" @click="extractInfo()">Info</button>
    <p>{{ lobbies }}</p>
  </div>
  <div class="card" v-else>
    <p>Nenhuma lobby encontrada</p>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
