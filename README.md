# 😋 qaFood — Testes Automatizados com Maestro

<img width="405" height="860" alt="Screenshot do qaFood" src="https://github.com/user-attachments/assets/15ad61e5-2117-42e6-a2d8-b7ddb1f90092" />



O projeto consiste de Suíte de testes **end-to-end (E2E)** para o aplicativo **qaFood**, uma versão do **iFood** utilizada como projeto de estudo, desenvolvida pela **Qazando** (professores Eduardo Finotti e Hebert Soares).

Todos os testes e a estrutura deste repositório foram criados por **Diogo Amancio**, com base nos conhecimentos adquiridos no curso **Automação Mobile com Maestro**, utilizando **Android Studio**, **WSL (Linux)** e **Maestro**.

---

## 📱 Sobre o app

O qaFood simula um aplicativo de delivery completo, cobrindo a jornada real de um usuário:

**Login → Lojas → Cardápio → Sacola → Pedido → Acompanhamento**

**Abaixo um vídeo demonstrativo do cenário de teste end-to-end (E2E) acima:**

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
| **1.Login** | Autenticação, campos, erros de validação e comportamento do botão de acesso | Ponto de entrada — “quero acessar o app” |
| **2.Lojas** | Listagem, busca, navegação e permissão de localização | “Onde quero pedir?” |
| **3.Cardápio** | Produtos, carrinho, contador e navegação dentro do restaurante | “O que vou comer?” |
| **4.Sacola** | Gerenciamento do carrinho, subtotal e persistência | “Revisar minha compra” |
| **5.Pedido** | Confirmação, pagamento, finalização e acompanhamento | “Confirmar, pagar e receber” |

---

## 🔍 1. Feature Login

Valida o processo de autenticação e o comportamento dos campos e do botão de acesso, organizado em **8 subtópicos de testes**.

**Abaixo um vídeo demonstrativo de um dos cenários de testes da tela de Login:**

https://github.com/user-attachments/assets/e8c3bc69-581b-4647-b176-4fc54099d5a4

## 📍 Subtópicos de testes - Feature Login

**A) 1. Fluxo básico:**

<img width="503" height="267" alt="image" src="https://github.com/user-attachments/assets/852c6fb0-860e-42e2-9fee-c8c8d6541a7f" />


**B) 2. Validação de formato e conteúdo:**

<img width="451" height="266" alt="image" src="https://github.com/user-attachments/assets/e3c000d8-16fe-490c-8aad-40d301beb1e1" />


**C) 3. Validação de senha:**

<img width="470" height="145" alt="image" src="https://github.com/user-attachments/assets/62fa7c72-5181-444e-8b21-303b5311c45b" />


**D) 4. Correção e recuperação de erro:**

<img width="398" height="57" alt="image" src="https://github.com/user-attachments/assets/91be85aa-47fd-4ec0-b386-ba0c102ad6f0" />


**E) 5. Interações com teclado e sistema operacional:**

<img width="480" height="60" alt="image" src="https://github.com/user-attachments/assets/cded17ec-009b-4a02-8cda-863faf4d5123" />


**F) 6. Cliques repetidos e comportamento de interface:**

<img width="427" height="122" alt="image" src="https://github.com/user-attachments/assets/b6a28f50-c6b5-4ad2-b515-cb21eba3ce63" />


**G) 7. Concorrência e condição de corrida:**

<img width="543" height="32" alt="image" src="https://github.com/user-attachments/assets/e77560ad-d207-4c6d-baf7-e8b12e7d4993" />


**H) 8. Bloqueio por tentativas de senha:**

<img width="561" height="86" alt="image" src="https://github.com/user-attachments/assets/66dbc146-fa83-4169-9f93-fa783b9a0a9c" />

---

## 🔍 2. Feature Lojas

Valida a exibição, navegação e pesquisa dos restaurantes, organizada em **8 subtópicos de testes**.

**Abaixo um vídeo demonstrativo de um dos cenários de testes da tela das Lojas:**

https://github.com/user-attachments/assets/25a9231e-af96-4e54-8c0a-f329d3bdd910

## 📍 Subtópicos de testes - Feature Lojas

**A) 1. Acesso básico à tela de Lojas:**


<img width="480" height="91" alt="image" src="https://github.com/user-attachments/assets/ce33c954-46e8-4d1e-8aae-04aa6f559411" />


**B) 2. Navegação e visualização da lista:**


<img width="458" height="178" alt="image" src="https://github.com/user-attachments/assets/05a42e26-f733-47f6-ad06-6a59f09fd73c" />


**C) 3. Permissão e seleção de endereço:**


<img width="553" height="120" alt="image" src="https://github.com/user-attachments/assets/192bb61f-3c25-4f7d-8792-df28e1500743" />


**D) 4. Busca básica:**


<img width="608" height="147" alt="image" src="https://github.com/user-attachments/assets/2b1c3255-9eb0-4a0c-b0eb-4f8de8da7856" />


**E) 5. Busca por restaurantes específicos:**


<img width="387" height="178" alt="image" src="https://github.com/user-attachments/assets/88299dfb-1ccb-4884-ae62-dff02502f31f" />


**F) 6. Busca com espaços e capitalização:**


<img width="642" height="175" alt="image" src="https://github.com/user-attachments/assets/d356284d-0f6b-4488-9e8e-4aeab9b95ae5" />


**G) 7. Casos de borda da busca:**


<img width="560" height="82" alt="image" src="https://github.com/user-attachments/assets/d26038cd-5bfe-40e6-81da-16cd622055fe" />


**H) 8. Persistência e ciclo de vida do aplicativo:**


<img width="492" height="56" alt="image" src="https://github.com/user-attachments/assets/21071b80-9a43-493c-a53d-d9d46bedae91" />


---

## 🔍 3. Feature Cardápio

Valida o acesso aos restaurantes e o comportamento dos produtos, organizada em **7 subtópicos de testes**.

**Abaixo um vídeo demonstrativo de um dos cenários de testes da tela de Cardápio:**

__INSERIR VÍDEO__

## 📍 Subtópicos de testes - Feature Cardápio


**A) 1. Acesso e carregamento básico do cardápio**

<img width="568" height="120" alt="image" src="https://github.com/user-attachments/assets/3365606a-167b-4bb3-be49-4a371295f129" />


**B) 2. Validação dos elementos do cardápio**



<img width="495" height="88" alt="image" src="https://github.com/user-attachments/assets/636d5765-14be-455f-b227-6c69117b01b6" />


**C) 3. Navegação dentro e fora do cardápio**


<img width="505" height="87" alt="image" src="https://github.com/user-attachments/assets/02fd2a7b-580b-41ba-89ed-74f101d7602e" />


**D) 4. Adição de um produto ao carrinho**

<img width="638" height="57" alt="image" src="https://github.com/user-attachments/assets/7ca5bfe1-acd3-40de-b049-d579615a8d26" />


**E) 5. Adição e persistência de múltiplos produtos**


<img width="692" height="92" alt="image" src="https://github.com/user-attachments/assets/b4aa6db4-a1d2-406e-a357-e57dd12b0d09" />


**F) 6. Persistência do estado em diferentes condições**

<img width="496" height="25" alt="image" src="https://github.com/user-attachments/assets/9ee55c16-81e4-4f47-9f32-f0511efa620e" />


**G) 7. Cenário de concorrência / múltiplas ações rápidas**


<img width="587" height="30" alt="image" src="https://github.com/user-attachments/assets/40c36103-245c-43fe-8f1f-a6ce549084c3" />

---

## 🔍 4. Feature Sacola (Carrinho)

Valida o funcionamento do carrinho, organizada em **7 subtópicos de testes**.

**Abaixo um vídeo demonstrativo de um dos cenários de testes da tela da Sacola (Carrinho):**

__INSERIR VÍDEO__

## 📍 Subtópicos de testes - Feature Sacola (Carrinho)

**A) 1. Operações básicas da Sacola**


<img width="662" height="122" alt="image" src="https://github.com/user-attachments/assets/b7f63ee8-240a-451e-849e-98a62885bdda" />


**B) 2. Cálculo/subtotal**

<img width="552" height="57" alt="image" src="https://github.com/user-attachments/assets/125bc761-00f3-468e-9806-cd19b8877c10" />


**C) 3. Navegação entre Sacola e Cardápio**


<img width="683" height="65" alt="image" src="https://github.com/user-attachments/assets/d5763925-dc3f-4937-875e-10139f452e72" />


**D) 4. Cancelamento**

<img width="637" height="57" alt="image" src="https://github.com/user-attachments/assets/0e3c83b5-03cf-4778-a695-6a2c53e7e449" />


**E) 5. Limpeza da Sacola**


<img width="593" height="60" alt="image" src="https://github.com/user-attachments/assets/b9e7bb64-1220-4581-a34a-6b5ca7712491" />


**F) 6. Múltiplos produtos e preservação de estado**


<img width="612" height="53" alt="image" src="https://github.com/user-attachments/assets/e6f55658-be0b-4c4c-8253-b3db5174ff87" />


**G) 7. Comportamento do aplicativo**


<img width="442" height="50" alt="image" src="https://github.com/user-attachments/assets/e691f621-42c0-42f4-baa2-1745dfe9889b" />


---

## 🔍 5. Feature Pedido

Valida a confirmação e a finalização do pedido, organizada em **7 subtópicos de testes**.

**Abaixo um vídeo demonstrativo de um dos cenários de testes da tela de Pedido:**

__INSERIR VÍDEO__

## 📍 Subtópicos de testes - Feature Pedido

**A) 1. Acesso e confirmação básica do pedido**

<img width="602" height="61" alt="image" src="https://github.com/user-attachments/assets/6c969ce5-f368-4a82-8cb9-035728643967" />


**B) 2. Validação de dados e condições obrigatórias**


<img width="672" height="90" alt="image" src="https://github.com/user-attachments/assets/809e6f43-189e-48e8-8f09-be35bae40b48" />


**C) 3. Cancelamento da finalização e preservação do carrinho**


<img width="513" height="31" alt="image" src="https://github.com/user-attachments/assets/3b449fe0-20ce-48be-aefc-b9db32b959d0" />


**D) 4. Realização do pedido por diferentes formas de pagamento**


<img width="603" height="93" alt="image" src="https://github.com/user-attachments/assets/3268d0f5-10b2-4eef-a169-0060ee6f284a" />


**E) 5. Validação completa do pedido realizado**

<img width="462" height="27" alt="image" src="https://github.com/user-attachments/assets/1b92901a-7bf2-40dd-a952-9ec07f1cfc63" />


**F) 6. Navegação após a conclusão do pedido**

<img width="488" height="36" alt="image" src="https://github.com/user-attachments/assets/24bb89a5-b36d-473e-b914-565f31f6ad98" />


**G) 7. Persistência do estado após alteração de orientação**


<img width="443" height="32" alt="image" src="https://github.com/user-attachments/assets/179e40f6-a924-4d58-b5d3-ad6d1ddc9d98" />

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
