# 😋 qaFood — Testes Automatizados com Maestro

<img width="405" height="860" alt="Screenshot do qaFood" src="https://github.com/user-attachments/assets/15ad61e5-2117-42e6-a2d8-b7ddb1f90092" />

Suíte de testes **end-to-end (E2E)** para o aplicativo **qaFood**, uma versão do **iFood** utilizada como projeto de estudo, desenvolvida pela **Qazando** (professores Eduardo Finotti e Hebert Soares).

Todos os testes e a estrutura deste repositório foram criados por **Diogo Amancio**, com base nos conhecimentos adquiridos no curso **Automação Mobile com Maestro**, utilizando **Android Studio**, **WSL (Linux)** e **Maestro**.

---

## 📱 Sobre o app

O qaFood simula um aplicativo de delivery completo, cobrindo a jornada real de um usuário:

**Login → Lojas → Cardápio → Sacola → Pedido → Acompanhamento**

https://github.com/user-attachments/assets/794fba4c-bc3d-45a3-a7b9-d5e267cb79f7

---

## 🛠️ Ambiente e rotina diária

O ambiente de testes combina:

- **Emulador Android no Windows**
- **WSL (Linux)**
- **ADB**
- **Maestro CLI**
- **Maestro Studio**

### 1. Abrir o emulador — PowerShell

```powershell
cd $env:LOCALAPPDATA\Android\Sdk\emulator
.\emulator.exe -avd Pixel_4 -gpu swiftshader_indirect
```

Aguarde o emulador carregar completamente antes de seguir.

> ⚠️ **Importante:** não use o botão ▶ do Android Studio para iniciar o emulador. Utilize o comando acima.

### 2. Instalação do aplicativo — `qafoodcompletao.apk`

O arquivo `.apk` é apenas o **instalador do aplicativo**. Ele **não faz parte da conexão entre Windows, WSL, ADB e Maestro**.

A comunicação dos testes depende apenas de:

**Emulador rodando → ADB conectado via rede → Maestro apontando para o host correto**

O APK só precisa ser instalado nos seguintes casos:

1. Emulador novo, sem o aplicativo instalado;
2. Reset/Wipe do emulador, que remove os aplicativos instalados;
3. Necessidade de reinstalar o aplicativo;
4. Necessidade de trocar a versão do aplicativo.

#### Instalar o APK

Confirme que o emulador está rodando e conectado:

```bash
adb devices
```

Instale o APK no device correto:

```bash
adb -s <IP>:25555 install caminho/para/qafoodcompletao.apk
```

Confirme que a instalação foi realizada:

```bash
adb shell pm list packages | grep qazandoqafood
```

Resultado esperado:

```text
package:com.qazandoqafood
```

> 📌 **Importante:** depois que o aplicativo estiver instalado, o arquivo `.apk` não precisa ser utilizado novamente para executar os testes. O Maestro interage diretamente com o aplicativo por meio do `appId: com.qazandoqafood`.

### 3. Conectar o WSL ao emulador

```bash
adb kill-server
adb connect <IP>:25555
adb devices
```

> ⚠️ O IP não é fixo e pode mudar a cada reinício do Windows/WSL. Descubra o valor atual com:
>
> ```bash
> ip route show default | awk '{print $3}'
> ```
>
> Resultado esperado do `adb devices`:
>
> ```text
> <IP>:25555   device
> ```

### 4. Executar os testes com Maestro

```bash
maestro --host <IP> test <caminho-do-arquivo>.yaml
```

### 5. Abrir o Maestro Studio — opcional

```bash
cd ~/Downloads
./MaestroStudio.AppImage
```

Selecione o device `<IP>:25555` na lista.

> ⚠️ O streaming de tela ao vivo dentro do Studio não funciona neste ambiente devido a um erro de gRPC pela rede. Os testes continuam funcionando normalmente. Para acompanhamento visual, utilize a janela do emulador no Windows.

### Ordem diária

**PowerShell (emulador) → WSL/ADB (conexão) → Maestro/Maestro Studio (execução)**

---

## 📁 Estrutura do repositório

```text
Maestro/
├── 1 - Feature_Login/
├── 2 - Feature_Lojas/
├── 3 - Feature_Cardápio/
├── 4 - Feature_Sacola (Carrinho)/
└── 5 - Feature_Pedido/
```

Cada teste que depende de login reutiliza o mesmo flow base por meio de `runFlow`, evitando duplicação:

```yaml
- runFlow:
    file: "../1 - Feature_Login/A) 1. Login com credenciais corretas.yaml"
```

> 📌 O caminho do `runFlow` é sempre **relativo ao arquivo que o chama**. Testes salvos dentro de uma subpasta de Feature utilizam `../1 - Feature_Login/...`. Testes salvos diretamente na raiz `Maestro/` utilizam o caminho sem `../`.

---

## 🧭 A Jornada do usuário

A suíte tem como objetivo automatizar e validar a jornada completa do usuário dentro do qaFood:

```text
Login → Lojas → Cardápio → Sacola → Pedido → Acompanhamento
```

Os testes não validam apenas funcionalidades isoladas, mas também simulam comportamentos e situações próximas da utilização real de um aplicativo de delivery.

As informações a seguir apresentam a estrutura completa da suíte de testes do qaFood, organizada em cinco Features que representam, em sequência, a jornada do usuário no aplicativo. Cada Feature possui um conjunto de subtópicos que agrupa os cenários por funcionalidade e nível crescente de complexidade, permitindo visualizar de forma clara o que é validado em cada etapa, desde o login até a finalização e o acompanhamento do pedido.

## 💻 Features 

| Feature | O que valida | Papel na jornada |
|---|---|---|
| **1. Login** | Autenticação, campos, erros de validação e comportamento do botão de acesso | Ponto de entrada — “quero acessar o app” |
| **2. Lojas** | Listagem, busca, navegação e permissão de localização | “Onde quero pedir?” |
| **3. Cardápio** | Produtos, carrinho, contador e navegação dentro do restaurante | “O que vou comer?” |
| **4. Sacola** | Gerenciamento do carrinho, subtotal e persistência | “Revisar minha compra” |
| **5. Pedido** | Confirmação, pagamento, finalização e acompanhamento | “Confirmar, pagar e receber” |

---

## 🔍 1. Feature Login

Valida o processo de autenticação e o comportamento dos campos e do botão de acesso, organizado em **8 subtópicos de testes**.

**Abaixo um vídeo demonstrativo de um dos cenários de testes de Login:**

https://github.com/user-attachments/assets/e8c3bc69-581b-4647-b176-4fc54099d5a4

## Subtópicos de testes - Feature Login

**A) 1. Fluxo básico**

Login com credenciais corretas, campos vazios (e-mail, senha ou ambos) e bloqueio de envio correspondente.

**B) 2. Validação de formato e conteúdo**

E-mail não cadastrado, espaços em branco isolados ou combinados, espaços nas pontas, maiúsculas e caracteres especiais (`+` e apóstrofo).

**C) 3. Validação de senha**

Senha incorreta, maiúsculas na senha e limites de tamanho (e-mail e senha muito longos).

**D) 4. Correção e recuperação de erro**

Correção do e-mail antes do envio e correção da senha após um erro até conseguir entrar.

**E) 5. Interações com teclado e sistema operacional**

Tecla **Enter/Done**, rotação de tela durante o preenchimento e retorno do aplicativo após ida para segundo plano.

**F) 6. Cliques repetidos e comportamento de interface**

Duplo clique sequencial, cliques fixos e cliques repetidos até erro, com e sem confirmação.

**G) 7. Concorrência e condição de corrida**

Duplo toque simultâneo no botão **Entrar**.

**H) 8. Bloqueio por tentativas de senha**

Mesma conta e contas diferentes com senha errada em sequência, além do reforço do teste de campo vazio.

---

## 🔍 2. Feature Lojas

Valida a exibição, navegação e pesquisa dos restaurantes, organizada em **8 subtópicos de testes**.**.

**A) 1. Acesso básico à tela de Lojas**

Login com acesso à lista, abertura de cardápio e scroll inicial.

**B) 2. Navegação e visualização da lista**

Scroll até cada restaurante individualmente e até o fim da lista.

**C) 3. Permissão e seleção de endereço**

Abertura do modal, permitir, cancelar e preenchimento automático do endereço.

**D) 4. Busca básica**

Busca por caractere único, termo parcial, restaurante inexistente e limpeza da busca para nova pesquisa.

**E) 5. Busca por restaurantes específicos**

Busca pelo nome exato de cada um dos 6 restaurantes cadastrados.

**F) 6. Busca com espaços e capitalização**

Espaços nas pontas e matriz completa de maiúsculas/minúsculas: total, parcial por palavra e mista.

**H) 7. Casos de borda da busca**

Apenas espaços em branco, caracteres especiais/números isolados ou misturados com nome válido.

**8. Persistência e ciclo de vida do aplicativo**

Sessão perdida ao fechar/reabrir o aplicativo e re-login funcional na sequência.

---

## 🔍 3. Feature Cardápio

Valida o acesso aos restaurantes e o comportamento dos produtos, organizada em **7 subtópicos de testes**.

**A) 1. Acesso e carregamento básico do cardápio**

Acesso a diferentes restaurantes, bloqueio sem endereço selecionado e permissão de localização não solicitada novamente.

**B) 2. Validação dos elementos do cardápio**

Cabeçalho do restaurante, nome/preço/descrição do item e carrinho vazio ao abrir.

**C) 3. Navegação dentro e fora do cardápio**

Scroll para baixo e para cima e retorno à tela anterior, tanto pelo botão da interface quanto pelo botão físico **Voltar**.

**D) 4. Adição de um produto ao carrinho**

Adicionar um item, confirmar o contador e validar sua persistência ao sair da página.

**E) 5. Adição e persistência de múltiplos produtos**

Adicionar vários itens, validar a persistência de todos ao sair e verificar ausência de duplicação/perda após idas e vindas.

**F) 6. Persistência do estado em diferentes condições**

Contador de produtos mantido após rotação de tela.

**G) 7. Cenário de concorrência / múltiplas ações rápidas**

Duplo toque rápido no botão de adicionar.

---

## 🔍 4. Feature Sacola (Carrinho)

Valida o funcionamento do carrinho, organizada em **7 subtópicos de testes**.

**A) 1. Operações básicas da Sacola**

Abrir vazia, abrir após adicionar, adicionar e remover e adicionar o mesmo item duas vezes.

**B) 2. Cálculo/subtotal**

Soma dos itens adicionados e soma quando o mesmo item é duplicado.

**C) 3. Navegação entre Sacola e Cardápio**

Retorno ao cardápio preservando o item e adição de um segundo produto diferente após o retorno.

**D) 4. Cancelamento**

Desistir da limpeza da sacola e desistir da remoção de um item.

**E) 5. Limpeza da Sacola**

Limpar com produtos diferentes, botão **Limpar** habilitado mesmo vazio e readicionar item pelo botão **“Adicionar itens”**.

**F) 6. Múltiplos produtos e preservação de estado**

Três itens diferentes com subtotal correto e botão físico **Voltar** preservando os itens.

**G) 7. Comportamento do aplicativo**

Rotação de tela, perda de sessão de login e perda do conteúdo da sacola ao fechar/reabrir o aplicativo.

---

## 🔍 5. Feature Pedido

Valida a confirmação e a finalização do pedido, organizada em **7 subtópicos de testes**.

**A) 1. Acesso e confirmação básica do pedido**

Subtotal, taxa de entrega e total com um item, além da soma correta com múltiplos itens.

**B) 2. Validação de dados e condições obrigatórias**

Alerta sem forma de pagamento selecionada, cupom vazio e cupom inválido.

**C) 3. Cancelamento da finalização e preservação do carrinho**

Voltar da tela de confirmação sem finalizar, mantendo o carrinho intacto.

**D) 4. Realização do pedido por diferentes formas de pagamento**

Pedido com **Dinheiro** e com **Cartão de crédito**, incluindo confirmação de sucesso.

**E) 5. Validação completa do pedido realizado**

Conferência de todos os dados da tela de acompanhamento: status, previsão, endereço, pagamento e total.

**F) 6. Navegação após a conclusão do pedido**

Retorno à página de **Lojas** após finalizar o pedido.

**G) 7. Persistência do estado após alteração de orientação**

Rotação de tela na tela de acompanhamento após a conclusão do pedido.

---

## 💡 Aprendizados técnicos

- **Sintaxe YAML:** `appId` fica no cabeçalho do arquivo, antes do `---`, nunca dentro da lista de comandos. Seletores como `id:` precisam de indentação correta e espaço após os dois-pontos (`id: "email"`, e não `id:"email"`).
- **`tapOn` não digita:** para preencher campos, utilize `inputText` após o comando de toque, quando necessário.
- **Ambiguidade de seletores:** `rightOf: "texto"` e `point: "x%,y%"` são frágeis após scroll e podem clicar no elemento errado sem gerar erro. Prefira o `id` real do elemento, descoberto via `maestro hierarchy` ou pelo Inspector do Maestro Studio.
- **`scrollUntilVisible` é just-in-time:** revelar um elemento não garante que os próximos também fiquem visíveis. Utilize um `scrollUntilVisible` para cada elemento que precise ser tocado ou validado.
- **Alertas e pop-ups de confirmação:** título e corpo do modal geralmente são utilizados com `assertVisible` para validação; o botão de ação final deve ser acionado com `tapOn`.
- **IDs confirmados no app:** `add-item-buttom` (sic — contém erro de digitação no próprio app), `open-cart-button` e `back-button`.
- **Mensagens reais confirmadas:** `"Erro ao realizar login"`, `"CUPOM inválido"` e `"Selecione uma forma de pagamento"`.
- **Ciclo de vida do aplicativo:** fechar/reabrir o app com `launchApp: clearState: false` **não preserva a sessão de login neste aplicativo**. É necessário refazer o `runFlow` de login mesmo sem limpar o estado.
